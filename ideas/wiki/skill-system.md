# Skill System：フォルダ単位のClaudeへの命令セット

## 概要

Skill System は、フォルダ単位で Claude に対する命令セットを体系的に整理し、再利用可能にするフレームワークです。「一度教えて、何度でも使う」というコンセプトの下、SKILL.md を中心に scripts/、references/、assets/ などのアセットを組み合わせることで、Claude.ai、Claude Code、API といった全プラットフォームで統一的に動作する命令体系を実現します。

[Progressive Disclosure パターン](progressive-disclosure-pattern.md)に基づき、YAMLフロントマター → SKILL.md本文 → references/リンクという3段階の段階的開示により、Claude が必要に応じて適切な情報を段階的に参照できる構造を採用しています。

## Skill の本質：Markdown ファイルではなく、フォルダ単位の構成体

Skill の本質は **Markdown ファイルではなく、フォルダ（スクリプト・アセット・データ込み）として考える** ことです。[SKILL.md仕様](skill-md-specification.md)では SKILL.md を核に位置付けていますが、実運用では scripts/、references/、assets/ との組み合わせが重要です。また、[Skillのメモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md)における SQLite や JSON による状態管理を含めることで、単なる命令セットから自己参照型のワークフロー自動化へと進化させることができます。

実例として、[書籍メタデータ補完システム](book-metadata-completion-system.md)では `fill_book_meta.py` スクリプトが OpenBD → NDL Search API → Google Books という3段階フォールバック機構を実装し、170件中132件のメタデータを自動補完しています。このように、Skill は単なる指示ではなく、実行可能なツール・スクリプトを含んだ統合的なアセットセットとして機能します。

## Skill = 再利用可能なプロセス定義：$20の投資で業務チーム化

~~Skill は単なるドキュメント~~ → **Skill は再利用可能なプロセス定義**です。Mr. Buzzoniが示唆する「Top 67 Claude Skills」は、Claude の $20/月サブスクリプションを最大活用するための体系的なプロセス集を意味します。

これは単なる「一度書いたプロンプトを再利用する」というレベルではなく、**定期的なメンテナンス手順、トラブルシューティングプロセス、レポート生成フロー、ログ解析、バックアップ検証** といった業務の核となるルーティン作業をSkill化することで、以下を実現します：

- **業務ナレッジの標準化**：複数スタッフが同じプロセスで同じ品質を実現
- **新人育成の効率化**：Skill として体系化されたプロセスは、教育・引き継ぎコストを大幅削減
- **社内ドキュメント作成業務の削減**：年間数百時間のドキュメント業務を Claude に委譲
- **季節・プロジェクト条件下での一貫性**：同じSkillを異なる条件で繰り返し実行しても、人為的ミスリスクが低下

製造業のシステム管理者の場合、定期メンテナンス手順（月次チェック、四半期バックアップ検証、年次監査ログ分析）をSkill化すれば、$20/月で複数人の業務を自動化できます。これが「$20の投資で開発チーム全体の機能を実現する」という意味です。

## Skill 実行の原理：「正しいフォーム」の定義と繰り返し実行

Skill の実行原理は、ランニングや筋トレと同じです。**一度正しいフォーム（プロセス）を定義すれば、あとは繰り返し実行するだけ** で、以下のメリットが得られます：

- **フォームの定義段階**：最初はClaudeと対話しながら、試行錯誤で最適なプロセスを構築（投資期間）
- **実行段階**：一度定義されたSkillを何度でも再利用。学習曲線を排除し、常に高品質を維持
- **改善段階**：実運用での失敗や効率化機会を [Gotchas](#gotchasセクションの重要性) として記録し、次の実行に反映

これは**自分たちの業務フローを言語化してAIに定義させることが、継続的な業務改善の第一歩** となるという原則です。定期的にSkillを見直し、新しい条件下で再実行させることで、プロセスそのものが自動的に進化します。

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

実行可能なコード資産。Python、Bash、その他の言語で記述された、Skill の自動化を担当するモジュール。バージョン管理と依存関係の明記が必須です。[再利用可能プロセス定義によるSKILL.md](skill-reusable-process-definition-manufacturing-knowledge-management.md)では、この層がルーティン業務の自動化を担う中心的な役割を果たします。

### 参考資料層（references/）

外部ドキュメント、API仕様、テンプレート、チェックリスト。SKILL.md から段階的に参照される補助資料。

### アセット層（assets/）

データベース、設定ファイル、サンプルデータ。Skillの実行に必要な静的・動的データを格納。

## Skill の再利用戦略

### プロンプトでの利用

Claude.ai でのやり取りでは、SKILL.md ファイルの全文をプロンプトに含めることが基本です。複数の Skill を組み合わせる場合は、Gotchas セクションを最優先で読み込ませ、その後 scripts/ のコード例を参照させるという優先度制御が効果的です。

### Claude Code での利用

Claude Code 環境では、scripts/ ディレクトリ全体をアップロードしたうえで SKILL.md を参照させることで、Claude が環境に応じた適切な実装変更を自動的に判断できるようになります。[Claude Code設定駆動ワークフロー](claude-code-configuration-driven-workflow.md)では、CLAUDE.md による設定管理を組み合わせることで、再利用可能な自動化をさらに強化できます。

### API 統合利用

Claude API を用いた自動化の場合、SKILL.md の Gotchas セクションと scripts/ の参考実装をシステムプロンプトに組み込むことで、エラーハンドリングと例外対応の精度が向上します。[Claude API レート制限最適化](claude-api-rate-limit-optimization-workflow.md)では、Skill に基づいた段階的モデル選択による効率化戦略を実装できます。

## Skill 開発のライフサイクル

1. **計画フェーズ**：問題定義、実装範囲、成功指標の明確化
2. **実装フェーズ**：スクリプト開発、テスト、ドキュメント作成
3. **検証フェーズ**：実運用での試行、失敗パターンの記録
4. **最適化フェーズ**：Gotchas の充実、スクリプトの改善、参考資料の整備
5. **保守フェーズ**：定期的な見直し、依存関係の更新、バージョン管理

[Skillテスト戦略](skill-testing-strategy.md)では、トリガー・機能・パフォーマンス検証の3段階フレームワークを用いることで、各フェーズでの品質保証を体系化できます。

## Skill 組織展開と標準化

Skill の真の価値は、組織全体での再利用と標準化にあります。[Claude 67コアスキル](claude-skill-67-core-competencies-standardization.md)や [再利用可能プロセス定義によるSKILL.md](skill-reusable-process-definition-manufacturing-knowledge-management.md)で示されるように、業務ドメイン別に Skill 库を構築することで、以下を実現できます：

- **標準プロセスの可視化**：製造業の保守手順、トラブルシューティング、監査ログ分析など、業務の核となるプロセスを明文化
- **品質の均一性**：誰が、どの条件で実行しても、同じクォリティで結果が得られる
- **新人教育の加速**：Skill を教材として、段階的に業務ノウハウを習得可能
- **継続的改善**：実運用での発見や効率化機会を Gotchas として蓄積し、Skill 自体を進化させる

## 関連ページ

- [Progressive Disclosure パターン](progressive-disclosure-pattern.md): 段階的情報開示による効率化
- [SKILL.md仕様](skill-md-specification.md): ファイル命名ルールとフォルダ構造
- [Skillのメモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md): SQLite/JSONによる状態管理と自己参照型ワークフロー自動化
- [書籍メタデータ補完システム](book-metadata-completion-system.md): OpenBD→NDL→Google Booksの3段階フォールバック
- [Claude Code Skill設計の落とし穴](claude-code-skill-design-gotchas.md): Gotchasセクションの重要性と失敗から学ぶ設計知見
- [MCP（Model Context Protocol）と Skill の関係](mcp-skill-architecture.md): ツール連携とワークフロー設計
- [Skillテスト戦略](skill-testing-strategy.md): トリガー・機能・パフォーマンス検証の3段階
- [Claude Code設定駆動ワークフロー](claude-code-configuration-driven-workflow.md): CLAUDE.mdの設計規約自動遵守と保守業務の並列化
- [Claude API レート制限最適化](claude-api-rate-limit-optimization-workflow.md): 段階的モデル選択によるトークン効率化ワークフロー
- [Claude 67コアスキル](claude-skill-67-core-competencies-standardization.md): $20サブスクリプション活用と業務プロセス標準化
- [再利用可能プロセス定義によるSKILL.md](skill-reusable-process-definition-manufacturing-knowledge-management.md): 製造業の保守手順・トラブルシューティングの標準化と新人育成効率化
- [skill-creator スキル](skill-creator-automation.md): Skill設計・レビュー・改善の自動化

## 更新履歴
- 2026-03-08: 自分の業務へのアイデア
- 2026-03-08: descriptionフィールドが最重要
- 2026-03-08: 3段階の段階的開示（Progressive Disclosure）
- 2026-03-08: SKILL.md作成の厳守ルール
- 2026-03-08: Skillとは何か（基礎）
- 2026-03-08: MCPとSkillの役割分担
- 2026-03-08: テスト戦略
- 2026-03-18: CLAUDE.mdをHookで自動育成する（AppBrew）
- 2026-03-18: Claude Code スキルの設計知見（Thariq氏 / Anthropic）
- 2026-03-23: 書籍メタデータ補完の完了
- 2026-05-04: [XユーザーのMr. Buzzoniさん: 「Top 67 Claude Skills That Turn a $20 Subscription Into a Full Dev Team」](https://x.com/polydao/status/2044317956893471081)に基づく Skill = 再利用可能プロセス定義の再定義、$20投資による業務チーム化、ランニング的フォーム定義と繰り返し実行の原理、製造業の保守・トラブルシューティング標準化ユースケースの追加