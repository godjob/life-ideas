# Gitワークツリーによる並列タスク処理：3-5並列作業の生産性最大化

## 概要

Git worktreeは単一リポジトリから複数の独立したワーキングディレクトリを同時に作成できる機能で、3-5程度の並列タスク処理において劇的な生産性向上をもたらす。従来のブランチ切り替えによるコンテキスト喪失を排除し、複数の独立したタスク（UIファイル修正とスクリプト修正など）をシームレスに並行処理することで、開発フロー全体の効率化が実現する。

## 主要な知見

- **3-5並列作業が最適な粒度**：Claude Codeの50 Tips記事で報告されている知見として、worktreeによる3-5程度の並列タスク処理が最大の生産性向上をもたらす。6並列以上ではコンテキスト管理コストが増加し、逆効果になる傾向がある。

- **独立したタスクの並列化機会**：同一リポジトリ内でも、index.html改修とスクリプト修正、ドキュメント更新など、ファイルレベルでの競合が少ないタスクは容易に並列化できる。worktreeはこうした機会を構造的に活用する手段を提供する。

- **ブランチ切り替えオーバーヘッドの排除**：従来のgit checkoutによるブランチ切り替えでは、IDEの再読み込み、依存関係の再解決、ビルド再実行などのコンテキスト喪失が発生。worktreeはこれをゼロにする。

- **エージェント・チーム運用への応用**：[Claude Code Agent Teams](claude-code-agent-teams.md)における複数エージェントの並列実行と、worktreeの思想は通底している。各エージェントが独立したワーキングディレクトリで作業し、マージ時に統合する設計パターンが有効。

- **[Agentic Engineeringの監督者モデル](agentic-engineering-supervisor-model.md)との親和性**：複数の自動化エージェントが並列に実行される環境では、マージ・検証・調整の監督機能が重要になる。worktreeはこうしたマルチエージェント協調を技術的に支える基盤となる。

- **速度優先型プロトタイピングとの組み合わせ**：[速さが命題](speed-first-prototyping.md)の姿勢のもと、複数の案件提案やUI試案を同時並行で進める場合、worktreeにより物理的な並列性が確保され、検討時間が短縮される。

- **環境エンジニアリングの一部**：[エンバイロメント・エンジニアリング](environment-engineering.md)における「AI活動の土台づくり」として、開発環境の最適化はClaude Codeとの協働効率を高める重要要素である。worktreeの導入は開発フロー改善の実例。

- **[日常開発文脈とAIパートナーシップ](daily-context-ai-partnership.md)の強化**：複数タスクを物理的に分離することで、各worktreeに異なるCLAUDE.mdやSkillセットを配置し、AIに対するコンテキスト指示をより明確化できる。

- **コンテキスト管理の最適化**：各worktreeインスタンスで異なる依存パッケージバージョンやツール設定を持つことで、タスク間の環境干渉を最小化。特にレガシープロジェクトと最新プロジェクトの並列処理時に有効。

## 実装上の推奨事項

### worktreeの基本的な使い方

```bash
# 新しいworktreeを作成
git worktree add ../project-ui feature/ui-redesign
git worktree add ../project-api feature/api-enhancement
git worktree add ../project-docs feature/documentation

# worktreeの一覧確認
git worktree list

# worktreeの削除
git worktree remove ../project-ui
```

### 3-5並列運用のベストプラクティス

1. **タスクの独立性を最優先**：ファイルレベルで競合しないタスクを選別し、マージ時のコンフリクト処理を最小化
2. **各worktreeに専用ターミナルを割り当て**：コンテキストスイッチを物理的に削減
3. **定期的なマージイテレーション**：各worktreeの成果を2-3時間ごと、あるいは小機能ごとにmainにマージし、ドリフトを防止
4. **共有資源（データベース等）のハンドリング**：複数worktree間で共有リソースにアクセスする場合は、ロック機構やポート分離を用意
5. **CI/CDとの統合**：各worktreeからのプッシュが自動的にCI/CDパイプラインをトリガーするよう整備

## 関連ページ

- [Claude Code Agent Teams](claude-code-agent-teams.md)
- [Agentic Engineeringの監督者モデル](agentic-engineering-supervisor-model.md)
- [速さが命題](speed-first-prototyping.md)
- [エンバイロメント・エンジニアリング](environment-engineering.md)
- [日常開発文脈とAIパートナーシップ](daily-context-ai-partnership.md)

## 更新履歴
- 2026-03-20: git scrap からの気づき（26/03/20）
