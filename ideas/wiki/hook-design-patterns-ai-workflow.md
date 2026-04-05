# Hook設計パターン：SessionEnd・PreCompact・PreToolUseの活用法

## 概要

AIエージェントの自動実行・成長を実現するには、手書き指示では限界がある。Hookという機械的な強制タイミングを活用することで、[CLAUDE.md統治](claude-md-governance.md)の実行を確実化し、[Skill](skill-system.md)の継続的改善を仕組み化できる。SessionEnd・PreCompact・PreToolUseの3つのHookタイミングを組み合わせることで、セッション終了時の分析、圧縮前の重要情報抽出、スキル使用計測を漏れなく実現する。

## 主要な知見

- **AIも人間も「自動実行してください」は忘れる** - CLAUDE.mdに指示を書くだけでは実装されない。Hookで機械的に強制することが正解
- **SessionEnd + PreCompactの二重タイミング戦略** - SessionEndで自動実行を、PreCompactで圧縮前の重要情報を漏れなく拾える
- **PreToolUseによるスキル使用計測** - 自分のスキルの使用頻度・パターンを追跡でき、[量的自己記録とAI分析の相乗効果](quantified-self-ai-feedback-loop.md)を実現
- **無限ループ対策の環境変数フラグパターン** - `HOOK_EXECUTED=true`等のフラグで重複実行を防止。汎用的で他のHook設計にも転用可能
- **Git Hooksは権限スキップでも有効** - Claude Codeで`--dangerously-skip-permissions`を使用する場合でも、Git自体の仕組みのためHook実行は継続される。[Git Hooksセキュリティ防御](git-hooks-security-defense.md)として多層防御設計が重要

## Hook設計パターンの3つのタイミング

### 1. SessionEnd：セッション終了時の自動分析と記録

セッションが終了するタイミングで自動的に実行されるHook。[CLAUDE.md自動育成](claude-md-auto-cultivation-hooks.md)の主要な起動トリガーとなる。

**活用例：**
- セッション内での主要な判断・失敗を自動分析
- [AIエージェント失敗ログ](ai-failure-log.md)への自動記録
- [Skill](skill-system.md)の改善提案の自動生成
- セッション統計の記録（実行時間、エラー数、成功率等）

**実装のポイント：**
```
SessionEndで自動実行する内容：
- 本セッションで何を学んだか？
- 失敗は何か、原因は何か？
- CLAUDE.mdのどの部分を改善すべきか？
- 次のセッションに引き継ぐべき情報は？
```

### 2. PreCompact：圧縮前の重要情報抽出

コンテキストウィンドウの圧縮が発生する直前に実行。圧縮によって失われる可能性のある重要情報を事前に抽出・保存する。

**活用例：**
- 長期会話履歴から重要な決定・学習事項を自動抽出
- [メモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md)への永続化（SQLite/JSON）
- 次セッションへの知識引き継ぎ
- コンテキスト圧縮後の参照索引の生成

**実装のポイント：**
```
PreCompactで自動実行する内容：
- 会話履歴から固有名詞・重要決定の抽出
- セッション内で確立されたルール・パターンの保存
- スキル実行結果の要約と教訓の記録
- 未解決の課題・継続タスクの列挙
```

**圧縮前のメモリフラッシュパターン：**
```bash
#!/bin/bash
# pre-compact hook
if [ "$HOOK_EXECUTED" = "true" ]; then
  exit 0
fi

export HOOK_EXECUTED=true

# 会話履歴をJSONに構造化して永続化
sqlite3 memory.db "INSERT INTO sessions (timestamp, summary, key_decisions) VALUES ..."

# CLAUDE.mdの参照インデックスを更新
python scripts/extract_context.py > .context_index.json
```

### 3. PreToolUse：スキル使用前の計測と追跡

スキル・ツール実行の直前に実行。使用パターンの記録とリアルタイム分析を可能にする。

**活用例：**
- スキル使用頻度・成功率の自動計測
- [量的自己記録](quantified-self-ai-feedback-loop.md)への自動ログ記録
- 使用パターンの異常検知（予期しない組み合わせ等）
- スキルの有効性を定期的に評価

**実装のポイント：**
```
PreToolUseで自動実行する内容：
- ツール実行前の引数・パラメータを記録
- 使用コンテキスト（何を解決しようとしているか）の記録
- 類似の過去使用例との比較分析
- 使用予定スキルが最適か？代替手段はないか？の検証
```

**スキル利用統計の集約パターン：**
```json
{
  "tool_usage_log": {
    "timestamp": "2024-01-15T10:30:00Z",
    "tool_name": "code_analysis",
    "success": true,
    "execution_time_ms": 245,
    "context_tags": ["debugging", "performance"],
    "result_quality": "high"
  }
}
```

## 実装例：統合的なHook運用

### CLAUDE.mdの改善ループ設定

```yaml
# .git/hooks/post-commit (SessionEnd相当)
- 実行: セッション終了分析スクリプト
- 出力先: logs/session-analysis.md
- 保存内容: 学習内容・失敗分析・提案

# .git/hooks/pre-push (PreCompact相当)
- 実行: 圧縮前情報抽出スクリプト
- 出力先: .memory/persistent-context.json
- 保存内容: 重要決定・確立ルール・統計

# 各スキル実行時 (PreToolUse相当)
- 自動ログ記録: tools-usage.jsonl
- 分析対象: 月次スキル有効性レビュー
```

## Hook実行時の無限ループ対策

環境変数フラグパターンは、同じHookが複数回呼び出されるのを防ぐ汎用的な手法：

```bash
#!/bin/bash
if [ "$HOOK_SESSION_END_EXECUTED" = "true" ]; then
  exit 0  # 既に実行済み
fi

export HOOK_SESSION_END_EXECUTED=true

# 実際の処理
python analysis/session_end_analyzer.py
```

このパターンは、Hookが自らコミット・プッシュを生成する場合に特に重要。スクリプト実行 → Gitイベント発火 → Hook再実行という連鎖を防止する。

## 関連ページ

- [CLAUDE.md統治](claude-md-governance.md)
- [Skill](skill-system.md)
- [CLAUDE.md自動育成](claude-md-auto-cultivation-hooks.md)
- [AIエージェント失敗ログ](ai-failure-log.md)
- [メモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md)
- [量的自己記録とAI分析の相乗効果](quantified-self-ai-feedback-loop.md)
- [Git Hooksセキュリティ防御](git-hooks-security-defense.md)

## 更新履歴
- 2026-03-18: CLAUDE.mdをHookで自動育成する（AppBrew）
- 2026-03-21: --dangerously-skip-permissions のリスク対策
