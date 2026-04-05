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
- コンテキ