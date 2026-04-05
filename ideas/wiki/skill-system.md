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