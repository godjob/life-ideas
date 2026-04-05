# Skill System：フォルダ単位のClaudeへの命令セット

## 概要

Skill System は、フォルダ単位で Claude に対する命令セットを体系的に整理し、再利用可能にするフレームワークです。「一度教えて、何度でも使う」というコンセプトの下、SKILL.md を中心に scripts/、references/、assets/ などのアセットを組み合わせることで、Claude.ai、Claude Code、API といった全プラットフォームで統一的に動作する命令体系を実現します。

[Progressive Disclosure パターン](progressive-disclosure-pattern.md)に基づき、YAMLフロントマター → SKILL.md本文 → references/リンクという3段階の段階的開示により、Claude が必要に応じて適切な情報を段階的に参照できる構造を採用しています。

## Skill の本質：Markdown ファイルではなく、フォルダ単位の構成体

Skill の本質は **Markdown ファイルではなく、フォルダ（スクリプト・アセット・データ込み）として考える** ことです。[SKILL.md仕様](skill-md-specification.md)では SKILL.md を核に位置付けていますが、実運用では scripts/、references/、assets/ との組み合わせが重要です。また、[Skillのメモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md)における SQLite や JSON による状態管理を含めることで、単なる命令セットから自己参照型のワークフロー自動化へと進化させることができます。

実例として、[書籍メタデータ補完システム](book-metadata-completion-system.md)では `fill_book_meta.py` スクリプトが OpenBD → NDL Search API → Google Books という3段階フォールバック機構を実装し、170件中132件のメタデータを自動補完しています。このように、Skill は単なる指示ではなく、実行可能なツール・スクリプトを含んだ統合的なアセットセットとして機能します。

## Gotchas セクションの重要性

[Claude Code Skill設計の落とし穴](claude-code-skill-design-gotchas.md)で詳述されるように、**Gotchasセクションが最重要** です。Claudeが実際に失敗した場面から作られた内容が最も信号密度が高く、単なるベストプラクティスよりも実装時の落とし穴を効果的に回避できます。Skill設計時には、想定される失敗パターンや実装上の注意点を明示的に記録することで、再利用時の成功率を大幅に向上させることができます。

例えば、[書籍メタデータ補完システム](book-metadata-completion-system.md)では `import_kindle.py` の再実行時にメタデータが消失するバグを修正する際に、`load_existing_meta` 関数を追加することで既存データ保持の仕組みを整備しました。こうした実装上の失敗から学んだ知見を Gotchas として記録することで、同じ過ちを繰り返さないようにできます。

## Skill と MCP の役割分担

[MCP（Model Context Protocol）と Skill の関係](mcp-skill-architecture.md)を理解することは、AI エージェントの効果的な設計に不可欠です。

- **MCP = プロのキッチン**：ツール、データソース、外部サービスなどへの接続インターフェース。何ができるか（能力）
- **Skill = レシピ集**：MCP が提供する機能をどう組み合わせるか、どのような流れで実行するか。なぜそうするのか（戦略・ノウハウ）

MCP が提供する基本的な能力に対し、Skill はそれらを効果的に活用するための業務ノウハウを積層させます。言い換えると、MCP なしに Skill は動作できませんが、Skill があることで MCP の活用価値が大幅に向上するという依存関係にあります。

## Skill の構成要素

### SKILL.md 中核仕様

[SKILL.md仕様](skill-md-specification.md)では以下の標準セクション構成を定義しています：

1. **YAMLフロントマター**：メタデータ、タグ、バージョン、依存関係
2. **概要**：Skill が解く問題の定義
3. **前提条件**：実行環境、必要な権限、外部ツール
4. **手順**：段階的な実行フロー
5. **Gotchas**：実装時の落とし穴と対策
6. **参考資料**：references/ へのリンク

### スクリプト層（scripts/）

実行可能なコード資産。Python、Bash、その他の言語で記述された、Skill の自動化を担当するモジュール。バージョン管理と依存関係の明記が重須。

### 参考資料層（references/）

外部ドキュメント、API仕様、テンプレート、チェックリスト。SKILL.md から段階的に参照される補助資料。

### アセット層（assets/）

データベース、設定ファイル、サンプルデータ。Skillの実行に必要な静的・動的データを格納。

## Skill の再利用戦略

### プロンプトでの利用

Claude.ai でのやり取りでは、SKILL.md ファイルの全文をプロンプトに含めることが基本です。複数の Skill を組み合わせる場合は、Gotchas セクションを最優先で読み込ませ、その後 scripts/ のコード例を参照させるという優先度制御が効果的です。

### Claude Code での利用

Claude Code 環境では、scripts/ ディレクトリ全体をアップロードしたうえで SKILL.md を参照させることで、Claude が環境に応じた適切な実装変更を自動的に判断できるようになります。

### API 統用

Claude API を用いた自動化の場合、SKILL.md の Gotchas セクションと scripts/ の参考実装をシステムプロンプトに組み込むことで、エラーハンドリングと例外対応の精度が向上します。

## Skill 開発のライフサイクル

1. **計画フェーズ**：問題定義、実装範囲、成功指標の明確化
2. **実装フェーズ**：スクリプト開発、テスト、ドキュメント作成
3. **検証フェーズ**：実運用での試行、失敗パターンの記録
4. **最適化フェーズ**：Gotchas の充実、スクリプトの改善、参考資料の整備
5. **保守フェーズ**：定期的な見直し、依存関係の更新、バージョン管理

## 関連ページ

- [Progressive Disclosure パターン](progressive-disclosure-pattern.md)
- [SKILL.md仕様](skill-md-specification.md)
- [Skillのメモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md)
- [書籍メタデータ補完システム](book-metadata-completion-system.md)
- [Claude Code Skill設計の落とし穴](claude-code-skill-design-gotchas.md)
- [MCP（Model Context Protocol）と Skill の関係](mcp-skill-architecture.md)

## 更新履歴
- 2026-03-08: 自分の業務へのアイデア
- 2026-03-08: descriptionフィールドが最重要
- 2026-03-08: 3段階の段階的開示（Progressive Disclosure）
- 2026-03-08: SKILL.md作成の厳守ルール
- 2026-03-08: Skillとは何か（基礎）
- 2026-03-08: MCPとSkillの役割分担
- 2026-03-08: テスト戦略
- 2026-03-18: CLAUDE.mdをHookで自動育成する（AppBrew）
- 2026-03-18: Claude Code スキルの設計知見（Thariq氏 / Anthropic
- 2026-03-23: 書籍メタデータ補完の完了
