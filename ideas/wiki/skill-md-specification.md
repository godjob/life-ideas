# SKILL.md仕様：ファイル命名ルールとフォルダ構造

## 概要

SKILL.md は [Skill System](skill-system.md) の中核をなすドキュメントであり、フォルダ単位での Claudeへの命令セットを定義するファイルです。ファイル命名規則とフォルダ構造を厳密に守ることで、[Skill System](skill-system.md) の一貫性と検索性を確保し、[AI オーケストレーター](ai-orchestrator-role.md)による効率的なナレッジ活用を実現します。

本質的には、Skill は「Markdownファイル」ではなく **フォルダ（スクリプト・アセット・データを含む統合パッケージ）** として考えることが重要です。

## 主要な知見

- **ファイル名の厳守**：ファイル名は必ず `SKILL.md`（大文字小文字厳守）。変数名やバリエーションは許されない
- **Skill は本質的にはフォルダ**：Markdownドキュメントだけでなく、実行スクリプト・リソース・データファイルを含む統合的なユニット。[Claude Code スキル設計の落とし穴](claude-code-skill-design-gotchas.md)を参照
- **フォルダ命名規則**：フォルダ名は **ケバブケース**（`notion-project-setup`）で統一。スペース・アンダースコア・大文字は不可
- **README.md の禁止**：フォルダ内には README.md を置かない。すべての説明は SKILL.md または references/ サブフォルダに集約
- **構造の統一性**：一貫した命名規則により、[MCP と Skill の役割分担](mcp-skill-architecture.md)が明確になり、自動化ツールによる処理が容易になる
- **段階的情報開示**：[Progressive Disclosure パターン](progressive-disclosure-pattern.md)を適用し、SKILL.md では要点を、詳細は references/ に記載
- **description フィールドが最重要**：自動トリガーの成否を左右する critical な要素。「何をするか」と「いつ使うか（トリガー条件）」の両方を必ず含める。単なる要約ではなく、[いつ呼び出すか](description-field-best-practices.md)を明示的に記述することが機械的な自動実行の前提条件
- **Gotchas セクションの最高優先度**：Claudeが実際に失敗した場面から導き出された内容が最も信号密度が高い。[Claude Code スキル設計の落とし穴](claude-code-skill-design-gotchas.md)に記載されたパターンは、スキル設計時に必ず参照し、同じ轍を踏まないこと
- **テスト駆動設計**：難しいタスクを Claudeが成功するまで反復 → それを Skill に抽出するアプローチが最も効果的。[Skill テスト戦略](skill-testing-strategy.md)に基づき、トリガー・機能・パフォーマンスの3段階検証を実施
- **状態管理の統合**：スキルに SQLite/JSON でメモリ層を持たせると、自己参照型のワークフロー自動化が可能。[Skillのメモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md)参照