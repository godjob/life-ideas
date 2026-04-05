# Skillのメモリアーキテクチャ：SQLite/JSONによる状態管理と自己参照型ワークフロー自動化

## 概要

Skillが単なるマークダウンファイルではなく、スクリプト・アセット・データを含むフォルダ全体として機能する際、SQLiteやJSONをメモリ層として組み込むことで、ステートフルな自動化ワークフローが実現可能になる。これにより、スキルは外部依存なしに状態を永続化し、複数回の実行を通じた自己参照型の意思決定が可能になり、[Skillシステム](skill-system.md)の成熟度が大きく向上する。

## 主要な知見

- **Skillはフォルダ構造が本質**: [SKILL.md仕様](skill-md-specification.md)に従うフォルダは、スクリプト・アセット・データストレージを統合した完全な実行環境であり、マークダウンファイルのみでは不足している

- **メモリ層の役割**: SQLiteやJSONをSkill内に組み込むことで、Claudeからの複数回の呼び出しを通じて状態を保持でき、エージェント的な自動化が可能になる

- **自己参照型ワークフローの実装**: 前の実行結果を次の実行に継承する仕組みにより、タスクの進捗管理、条件分岐、リトライロジックなどが単一のSkillで完結する

- **descriptionの戦略的記述**: [descriptionフィールドの最適化](description-field-best-practices.md)において、スキルが「いつ呼び出されるべきか」を明確に書くことで、Claude側のトリガー判定精度が上がり、ステートフルな判断が活きてくる

- **Gotchasセクションの信号密度**: 実装時に直面する落とし穴（並行実行による競合、タイムスタンプ管理、ロック機構の必要性など）は、[Claude Code Skill設計の落とし穴](claude-code-skill-design-gotchas.md)として言語化することで、次のイテレーションの品質が劇的に向上する

- **スケーラビリティへの道**: 個別プロジェクト内での状態管理 → リポジトリチェックイン → プラグインマーケットプレイス共有という段階的な拡張が、自然な進化パターンである

## 実装パターン

### SQLiteを使った状態永続化

```
skills/
├── task-orchestrator/
│   ├── SKILL.md
│   ├── index.js          # メインスクリプト
│   ├── lib/
│   │   ├── db.js         # SQLite初期化・クエリ関数
│   │   └── state-manager.js
│   └── data/
│       └── workflow.db    # SQLiteデータベース
```

SQLiteを選ぶ利点：
- ファイルベースで環境依存なし（外部DBサーバー不要）
- トランザクション対応で並行実行の安全性確保
- JSONカラムで半構造化データも保持可能
- クエリの複雑な集計も可能

### JSONベースの軽量状態管理

シンプルなタスク管理なら、JSONファイルで十分：

```json
{
  "workflow_id": "wf-20260318-001",
  "status": "in_progress",
  "steps": [
    {
      "step_id": 1,
      "name": "data_validation",
      "status": "completed",
      "timestamp": "2026-03-18T10:30:00Z",
      "result": {
        "valid_records": 150,
        "invalid_records": 5
      }
    },
    {
      "step_id": 2,
      "name": "processing",
      "status": "pending",
      "timestamp": null,
      "result": null
    }
  ],
  "metadata": {
    "created_at": "2026-03-18T10:00:00Z",
    "last_updated": "2026-03-18T10:30:00Z",
    "retry_count": 0
  }
}
```

JSONの利点：
- 人間が読み書きしやすい
- バージョン管理システムとの親和性が高い
- 小～中規模なワークフローに最適
- スキーマの柔軟性がある

### 自己参照型ワークフローの実装例

```javascript
// index.js - 自己参照型実行ロジック
const fs = require('fs');
const path = require('path');

const STATE_FILE = path.join(__dirname, 'data', 'workflow-state.json');

function loadState() {
  if (fs.existsSync(STATE_FILE)) {
    return JSON.parse(fs.readFileSync(STATE_FILE, 'utf8'));
  }
  return { steps: [], status: 'pending' };
}

function saveState(state) {
  fs.mkdirSync(path.dirname(STATE_FILE), { recursive: true });
  fs.writeFileSync(STATE_FILE, JSON.stringify(state, null, 2));
}

function executeNextStep(state) {
  const nextStepIndex = state.steps.filter(s => s.status === 'completed').length;
  
  if (nextStepIndex >= state.steps.length) {
    state.status = 'completed';
    return state;
  }

  const step = state.steps[nextStepIndex];
  step.status = 'in_progress';
  step.started_at = new Date().toISOString();

  // ステップ固有のロジック実行
  try {
    const result = executeStep(step.name);
    step.result = result;
    step.status = 'completed';
  } catch (error) {
    step.error = error.message;
    step.status = 'failed';
    state.status = 'failed';
  }

  step.completed_at = new Date().toISOString();
  return state;
}

// メイン実行
const state = loadState();
const updatedState = executeNextStep(state);
saveState(updatedState);
console.log(JSON.stringify(updatedState, null, 2));
```

## 並行実行と競合の管理

### ファイルロック機構

複数のClaudeプロセスが同じSkillを呼び出す場合、状態ファイルへの競合が発生する：

```javascript
// lib/file-lock.js
const fs = require('fs');
const path = require('path');

class FileLock {
  constructor(baseDir) {
    this.lockFile = path.join(baseDir, '.lock');
    this.lockTimeout = 30000; // 30秒
  }

  acquire() {
    const startTime = Date.now();
    while (fs.existsSync(this.lockFile)) {
      if (Date.now() - startTime > this.lockTimeout) {
        throw new Error('Lock acquisition timeout');
      }
      // 100ms待機して再試行
      require('child_process').execSync('sleep 0.1');
    }
    fs.writeFileSync(this.lockFile, Date.now().toString());
  }

  release() {
    if (fs.existsSync(this.lockFile)) {
      fs.unlinkSync(this.lockFile);
    }
  }
}

module.exports = FileLock;
```

## 状態管理のベストプラクティス

### 1. バージョニング
状態ファイルに世代を持たせ、ロールバック可能に：

```json
{
  "version": 2,
  "timestamp": "2026-03-18T11:15:00Z",
  "history": [
    {
      "version": 1,
      "timestamp": "2026-03-18T11:00:00Z",
      "archived": true
    }
  ],
  "current_state": { ... }
}
```

### 2. タイムスタンプの厳密な管理
UTC ISO 8601形式を統一し、スキュー検出を可能に：

```javascript
function isStateStale(state, maxAgeSec = 3600) {
  const lastUpdate = new Date(state.last_updated);
  const age = (Date.now() - lastUpdate.getTime()) / 1000;
  return age > maxAgeSec;
}
```

### 3. メタデータの記録
実行環境、実行者、呼び出し元を記録：

```json
{
  "execution_context": {
    "invoked_by": "claude-3.5-sonnet",
    "invocation_time": "2026-03-18T11:20:00Z",
    "user_id": "user-abc123",
    "session_id": "session-xyz789"
  }
}
```

## SQLiteとJSONの使い分け

| 観点 | JSON | SQLite |
|------|------|--------|
| **単純性** | ◎ シンプル | △ セットアップが必要 |
| **スケール** | △ 数千レコード程度 | ◎ 大規模データセット対応 |
| **クエリ複雑性** | △ プログラマティック処理 | ◎ SQL で表現可能 |
| **並行実行** | △ ロック機構が必須 | ◎ トランザクション標準装備 |
| **バージョン管理** | ◎ Gitで自然に追跡 | △ バイナリのため効率が落ちる |
| **UI連携** | ◎ Webアプリから直接読み取り | ◎ ドライバ経由でアクセス |

## Skillの段階的な成熟度

### レベル1: ステートレス（現在の標準）
- マークダウン + スクリプトのみ
- 毎回独立実行、状態持たず

### レベル2: JSONベースの状態管理
- 軽量な進捗管理
- 小～中規模タスク向け
- バージョン管理との相性が良い

### レベル3: SQLiteで構造化状態管理
- 複雑なワークフロー
- 大規模データセット処理
- 分析・レポーティング機能

### レベル4: 自己学習型エージェントSkill
- 実行結果から自動的にパラメータ調整
- 異常検知・自動リカバリ
- エコシステム全体との連携

## 関連ページ

- [Skillシステム](skill-system.md)
- [SKILL.md仕様](skill-md-specification.md)
- [descriptionフィールドの最適化](description-field-best-practices.md)
- [Claude Code Skill設計の落とし穴](claude-code-skill-design-gotchas.md)

## 更新履歴

- **2026-03-18**: 初版作成。SQLiteとJSONのメモリアーキテクチャ、自己参照型ワークフロー、並行実行管理について記述
- **2026-03-18**: ベストプラクティス、段階的成熟度モデル追加