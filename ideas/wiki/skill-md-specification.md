# SKILL.md仕様：ファイル命名ルールとフォルダ構造

## 概要

SKILL.md は [Skill System](skill-system.md) の中核をなすドキュメントであり、フォルダ単位での Claudeへの命令セットを定義するファイルです。ファイル命名規則とフォルダ構造を厳密に守ることで、[Skill System](skill-system.md) の一貫性と検索性を確保し、[AI オーケストレーター](ai-orchestrator-role.md)による効率的なナレッジ活用を実現します。

本質的には、Skill は「Markdownファイル」ではなく **フォルダ（スクリプト・アセット・データを含む統合パッケージ）** として考えることが重要です。SKILL.md により定義された再利用可能なプロセスは、$20/月のClaudeサブスクリプションを開発チーム全体の機能へ転換し、製造業の保守手順やトラブルシューティングといったドメイン知識を標準化・継承可能な資産に変える仕組みです。

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
- **スキルの67コア競争力化**：製造業のシステム管理者や新人スタッフにとって、定期的なメンテナンス手順やトラブルシューティングプロセスをClaudeスキル化することで、業務ナレッジの標準化と新人育成の効率化が実現でき、月々の投資で社内ドキュメント作成業務を大幅削減できる。[Claude 67コアスキル：$20サブスクリプション活用と業務プロセス標準化](claude-skill-67-core-competencies-standardization.md)を参照

## ファイル構造

```
skills/
└── skill-folder-name/          # ケバブケース必須
    ├── SKILL.md               # 必須：仕様書とメタデータ
    ├── script.py              # 実行スクリプト（言語は問わず）
    ├── requirements.txt       # 依存関係（Python 例）
    ├── data/                  # スキル専用データフォルダ
    │   └── config.json
    └── references/            # 詳細ドキュメント
        ├── advanced-usage.md
        └── troubleshooting.md
```

## SKILL.md の最小構成

```markdown
---
title: "スキルタイトル"
description: "何をするか、いつ呼び出すか両方を含む詳細な説明"
version: "1.0.0"
triggers:
  - "自動実行の条件（トリガー文字列）"
  - "複数指定可"
author: "作成者"
last_updated: "YYYY-MM-DD"
---

# [スキル名]

## 説明
機能概要と使用シーン

## 使い方
実装例とコード例

## Gotchas
- Claudeが陥りやすいエラーパターン
- 回避方法

## 関連スキル
- [Related Skill](../related-skill/SKILL.md)
```

## 命名規則の細則

### フォルダ名

| ✅ OK | ❌ NG |
|------|------|
| `notion-project-setup` | `notion_project_setup` |
| `slack-message-formatter` | `SlackMessageFormatter` |
| `git-commit-validator` | `git commit validator` |
| `pdf-splitter` | `PDF-Splitter` |

### ファイル名

- `SKILL.md`（変数展開や小文字化は不可）
- スクリプト：言語による慣例に従う（`script.py`、`run.sh` など）
- リソース：ケバブケース（`config-template.json`）

## Description フィールドのベストプラクティス

Description は単なる説明ではなく、**自動化トリガーの仕様書**として機能します：

```markdown
description: "Notionのデータベースから完了済みタスク（status='Done'）を抽出し、
週単位のサマリーレポートを生成して Slack に投稿する。
トリガー条件：毎週月曜 09:00 UTC、または手動で '/report-tasks' コマンド実行時"
```

- **what**：機能内容を明確に
- **when**：トリガー条件を必ず含める
- **input/output**：期待される入出力形式

詳細な記述方法については [descriptionフィールドの最適化](description-field-best-practices.md) を参照してください。

## Progressive Disclosure パターンの適用

SKILL.md は **概要と実装** に絞り、詳細は別ファイルに分散：

- **SKILL.md**：30秒で読める概要、基本的な使い方、主要な Gotchas
- **references/advanced-usage.md**：高度な設定、カスタマイズ方法
- **references/troubleshooting.md**：トラブルシューティングガイド

[Progressive Disclosure パターン](progressive-disclosure-pattern.md)の詳細設計については、リンク先を参照してください。

## 再利用可能プロセス定義と業務知識標準化

Skill System により定義された再利用可能なプロセスは、単なる自動補完ツールではなく、**開発チーム全体の機能を実現する仕組み**です。特に製造業においては：

- **ルーティン作業のスキル化**：ログ解析、レポート生成、バックアップ検証といった定期的なメンテナンス作業をスキル化すれば、季節ごと・プロジェクトごとに異なる条件下でも一貫性のある実行が可能。人為的ミスリスクが低下し、新人育成時間が大幅短縮
- **「正しいフォーム」の定義と継続実行**：ランニングで「正しいフォーム」を習得した後は反復練習に専念するのと同じく、Claudeスキルで一度業務フロー（プロセス）を正確に定義すれば、その後は一貫性を保ったまま繰り返し実行可能。組織全体の実行ばらつきを削減
- **知識資産化と継承**：[再利用可能プロセス定義によるSKILL.md：製造業の保守手順・トラブルシューティングの標準化と新人育成効率化](skill-reusable-process-definition-manufacturing-knowledge-management.md)にて詳述

## 自動化との連携

Skill System は以下の形式で外部ツール・MCP から参照されることを想定：

```json
{
  "skill_id": "notion-project-setup",
  "folder_path": "skills/notion-project-setup/",
  "skill_file": "skills/notion-project-setup/SKILL.md",
  "description": "...",
  "triggers": ["...", "..."],
  "executable": "skills/notion-project-setup/script.py"
}
```

SKILL.md の命名と構造が厳密であるほど、自動トリガーの成功率が向上します。

## 関連ページ

- [Skill System](skill-system.md): フォルダ単位のClaudeへの命令セット定義
- [AI オーケストレーター](ai-orchestrator-role.md): 100倍エンジニアの役割と自動化ナレッジ活用
- [Claude Code スキル設計の落とし穴](claude-code-skill-design-gotchas.md): 実装失敗パターンと回避方法
- [MCP と Skill の役割分担](mcp-skill-architecture.md): ツール連携とワークフロー設計の方針
- [Progressive Disclosure パターン](progressive-disclosure-pattern.md): 段階的情報開示による効率化
- [descriptionフィールドの最適化](description-field-best-practices.md): トリガー条件と機能説明の書き方
- [Skill テスト戦略](skill-testing-strategy.md): トリガー・機能・パフォーマンス検証の3段階
- [Skillのメモリアーキテクチャ](skill-memory-architecture-stateful-workflow.md): SQLite/JSONによる状態管理と自己参照型ワークフロー
- [Claude 67コアスキル：$20サブスクリプション活用と業務プロセス標準化](claude-skill-67-core-competencies-standardization.md): ビジネス価値化と組織展開
- [再利用可能プロセス定義によるSKILL.md：製造業の保守手順・トラブルシューティングの標準化と新人育成効率化](skill-reusable-process-definition-manufacturing-knowledge-management.md): ドメイン知識の資産化と継承

## 更新履歴

- 2026-03-08: descriptionフィールドが最重要
- 2026-03-08: SKILL.md作成の厳守ルール
- 2026-03-08: テスト戦略
- 2026-03-18: Claude Code スキルの設計知見（Thariq氏 / Anthropic）
- 2026-03-23: 書籍メタデータ補完の完了
- 2026-05-04: [XユーザーのMr. Buzzoniさん: 「Top 67 Claude Skills That Turn a $20 Subscription Into a Full Dev Team」](https://x.com/polydao/status/2044317956893471081) より、再利用可能プロセス定義による業務知識標準化と新人育成効率化、製造業のルーティン作業スキル化による人為的ミスリスク低下の知見を追加