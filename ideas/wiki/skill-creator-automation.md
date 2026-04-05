# skill-creator スキル：Skill設計・レビュー・改善の自動化

## 概要

skill-creatorはClaude.aiに組み込まれた専門的なスキルで、Skillの設計・レビュー・改善提案を自動化します。一度成功したタスクをSkillに抽出する際、このスキルを活用することで、トリガー条件の最適化からパフォーマンス検証まで、プロンプトエンジニアリングの品質を大幅に向上させます。

## 主要な知見

- **反復成功からの抽出が最効率**：一つの難しいタスクをClaudeが成功するまで反復してから、それをSkillに抽出するのが最も効果的なアプローチ
- **自動レビュープロセス**：skill-creatorを使用することで、Skillの設計品質、説明の明確性、パフォーマンス最適化が自動的に検証される
- **3段階テストの実装支援**：[Skillテスト戦略](skill-testing-strategy.md)で定義されるトリガーテスト・機能テスト・パフォーマンス比較を自動で実施可能
- **トークン効率の可視化**：Skill有無でのトークン数・やり取り回数を比較し、実装の価値を定量的に証明
- **継続的改善サイクル**：一度作成したSkillを定期的に再評価し、改善提案を受け取る仕組み

## 活用シーン

### Skill設計の自動化

[skill-md-specification.md](skill-md-specification.md)で定義されたSKILL.md仕様に従って、手動で作成したスキル定義をskill-creatorに提示すると、以下を自動検証します：

- トリガー条件（descriptionフィールド）の最適化
- 機能説明の曖昧性排除
- パラメータスキーマの完全性チェック
- 実行手順の冗長性除去

### パフォーマンス改善提案

[skill-testing-strategy.md](skill-testing-strategy.md)の3段階テストを通じて収集したデータをskill-creatorに入力すると：

- トークン消費量削減の最適化案
- 不要な中間ステップの除去
- プロンプトの再構成提案
- 類似Skillの統合可能性検出

### 品質管理プロセス

[環境エンジニアリング](environment-engineering.md)として、Skillレビュー会議にskill-creatorを統合することで：

- 新規Skillの事前検証で議論時間を短縮
- [失敗ログ](ai-failure-log.md)からの改善ニーズを自動検出
- チーム内の[description書き方](description-field-best-practices.md)ルールの一貫性確保

## 実装アーキテクチャ

skill-creatorは[Skillシステム](skill-system.md)の一種として、他の[MCP](mcp-skill-architecture.md)ツール・Skillと連携します：

```
入力（新規or修正Skill）
    ↓
skill-creatorによる自動検証
    ├─ 構文・仕様チェック（SKILL.md準拠）
    ├─ テスト用データセット生成
    └─ パフォーマンス予測
    ↓
改善提案の出力
    ↓
[speed-first-prototyping.md](speed