# MCPとSkillの役割分担：ツール連携とワークフロー設計

## 概要

MCPとSkillは、AIエージェントが外部ツールやデータを活用する際の「接続」と「使用方法」を担当する相補的な概念です。MCPはプロのキッチンに置かれた調理器具やアクセスとして機能し、Skillはそれらをどう組み合わせるかのレシピとして機能します。MCPだけでは「接続したけど何をしていいかわからない」という状態に陥りやすく、Skillがその間隙を埋める役割を果たします。

特に実務では、[ワークフローのSkill化](workflow-skill-definition.md)を通じて、業務手順を標準化し、チーム内で共有可能な形に整理することが重要です。また、[Claude ChatとClaude Codeの役割分担](claude-chat-claude-code-workflow.md)を理解することで、MCPとSkillの実装がより効果的になります。

さらに、MCPの具体的な活用例として、[ドキュメント形式変換とMarkdown標準化](document-format-conversion-markdown-standardization.md)のようなツール連携により、複数形式の資料をAIが処理可能な統一形式に変換することで、ナレッジベース構築やLLMによる自動分析が加速します。

## 主要な知見

- **MCP = プロのキッチン**：外部ツール、API、データベース、リソースへのアクセスレイヤーを提供する基盤
- **Skill = レシピ**：MCPで利用可能なツールやデータをどのように組み合わせ、どのようなワークフローで活用するかを定義する命令セット
- **接続と活用の分離**：MCPが「何が使えるか」の問題を解決するのに対し、Skillは「どう使うか」の問題を解決する
- **実装の課題**：MCPの導入後、実際のワークフロー設計がなければ、ツール連携は形骸化しやすい
- **Skill Systemの位置づけ**：[Skill System](skill-system.md) はフォルダ単位でClaudeへの命令セットを定義し、[SKILL.md仕様](skill-md-specification.md)に従って実装される具体的な方法論
- **命名規則の厳守**：ファイル名は`SKILL.md`（大文字小文字厳守）、フォルダ名はケバブケース（例：`notion-project-setup`）で統一
- **descriptionフィールドの重要性**：Skillの自動トリガーには、「何をするか」と「いつ使うか（トリガー条件）」の両方が必須。ユーザーが実際に言いそうなフレーズを含め、1024文字以内で曖昧さを排除することが極めて重要。詳細は[descriptionフィールドの最適化](description-field-best-practices.md)を参照
- **ドキュメント形式統一の効果**：MCPサーバーを通じてWord、PDF、Excel等の複数形式をMarkdownに変換することで、[LLM統合による技術資料自動化](llm-integration-technical-documentation-automation.md)が可能になり、設備仕様書やマニュアルの問題診断が自動化できる
- **実務への適用と共有**：個人の業務手順をSkillとして定義することで、他のメンバーにも共有可能な標準化されたワークフローになる
- **Claude ChatとClaude Codeの役割分担**：Claude（チャット）は「相談・設計・プロンプト生成」、Claude Codeは「実行・ファイル操作」という役割分担が明確。MCPとSkillの設計段階ではClaude（チャット）を、実装・検証段階ではClaude Codeを活用することで、最も効率的なワークフローが実現できる

## MCPの役割

MCPは以下の機能を提供します：

- **ツール連携**：外部サービスとの接続インターフェース
- **データアクセス**：ファイルシステム、データベース、APIエンドポイントへのアクセス管理
- **リソース管理**：認証情報の安全な保管と、プロトコル経由のリソース公開
- **スケーラビリティ**：複数のツールを統一的に管理し、新規ツール追加時の拡張性を確保
- **ドキュメント形式統一**：複数の文書形式（Word、PDF、Excel等）をMarkdownに変換し、LLMが一律に処理可能な形式に標準化することで、ナレッジベース構築やテキスト検索の効率を大幅に向上

MCPが提供するのは「何が使えるか」という基盤のみであり、それをどの場面でどう活用するかはSkillによって定義される必要があります。

## Skillの役割

Skillは以下の機能を実現します：

- **ワークフロー定義**：複数のステップを組み合わせた業務手順の明文化
- **トリガー設定**：特定の条件下で自動実行される命令セットの設定
- **コンテキスト管理**：Skillの実行過程で必要な情報を整理し、Claudeに提供
- **標準化と再利用**：業務手順を形式化することで、チーム内での共有と再利用を可能に
- **ドキュメント活用**：MCPで統一化されたMarkdown形式の技術資料やマニュアルを活用し、問題診断やメンテナンス計画の自動立案を実行

Skillは単なるプロンプトではなく、MCPで利用可能なツールを活用して「実際に業務を遂行するための指示セット」です。

## MCPとSkillの相互作用

MCPとSkillが最大の効果を発揮するには、両者の役割を明確に分けることが重要です：

| 側面 | MCP | Skill |
|------|-----|-------|
| **提供物** | ツールへのアクセス権 | 業務手順の定義 |
| **設定場所** | システムレベル | ワークスペース/フォルダレベル |
| **変更頻度** | 低（ツール追加時） | 高（業務改善に応じて） |
| **責任者** | インフラ・IT部門 | 各チーム・個人 |
| **共有範囲** | システム全体 | 関連メンバーのみ |

## 実装のベストプラクティス

1. **MCPの準備段階**：必要なツール・APIの洗い出しとMCPの構築。特に複数形式のドキュメント処理が必要な場合は、markitdownのようなドキュメント変換ツールの導入を検討
2. **Skillの設計段階**：[Claude Chat](claude-chat-claude-code-workflow.md)を使用して業務フローを整理し、descriptionフィールドを適切に記述
3. **Skillの実装段階**：[Claude Code](claude-chat-claude-code-workflow.md)を活用してSKILL.mdファイルを作成・検証
4. **テストと反復**：実際の業務で検証し、トリガー条件やワークフローを改善
5. **チーム共有**：Skillが確定したら、関連メンバーに共有・導入

## ワークフロー設計の重要性

MCPとSkillが機能するには、ワークフロー設計が不可欠です。[ワークフローのSkill化](workflow-skill-definition.md)を通じて、以下を実現できます：

- 暗黙知を形式知に変換
- チーム全体での手順統一
- 新人教育の効率化
- 業務改善の可視化

特に製造業では、[データ統合可視化技術の製造業応用](data-integration-visualization-manufacturing-application.md)や[LLM統合による技術資料自動化](llm-integration-technical-documentation-automation.md)により、バラバラなデータソースを統一的にMCP経由でSkillに提供することで、設備仕様書・マニュアルからの自動問題診断やメンテナンス計画立案が実現可能になります。

単にツールを接続するだけでなく、それらをどう活用するかを明確に定義することで、初めてMCPとSkillの価値が実現されます。

## 関連ページ

- [Skill System](skill-system.md): フォルダ単位のClaudeへの命令セット
- [SKILL.md仕様](skill-md-specification.md): ファイル命名ルールとフォルダ構造
- [ワークフローのSkill化](workflow-skill-definition.md): 業務手順の標準化と共有
- [Claude ChatとClaude Codeの役割分担](claude-chat-claude-code-workflow.md): 効率的なワークフロー設計
- [descriptionフィールドの最適化](description-field-best-practices.md): トリガー条件と機能説明の書き方
- [ドキュメント形式変換とMarkdown標準化](document-format-conversion-markdown-standardization.md): 複数形式資料の統一化によるAI分析効率向上
- [LLM統合による技術資料自動化](llm-integration-technical-documentation-automation.md): 設備仕様書・マニュアルの問題診断とメンテナンス計画自動化
- [データ統合可視化技術の製造業応用](data-integration-visualization-manufacturing-application.md): バラバラなデータを統一的マップへ変換

## 更新履歴

- 2026-04-09: [markitdown/README.md at main · microsoft](https://github.com/microsoft/markitdown/blob/main/README.md) から、ドキュメント形式変換とMarkdown標準化によるMCP活用の実例を追加。複数形式資料の統一化、LLMによる自動分析加速、技術資料の問題診断自動化に関する知見を統合
- 2026-03-08: 初版作成