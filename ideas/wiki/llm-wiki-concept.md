# LLM Wiki コンセプト

## 概要

Andrej Karpathyが提唱する「LLMをWikiの維持者として使う」知識管理パターン。
従来のRAG（都度検索・都度合成）に替わり、LLMが**永続的・複利的なMarkdown知識ベース**を育て続ける設計。
このwiki（`ideas/wiki/`）はその思想を `git scrap` フローに組み込んだ実装。

## 3層アーキテクチャ

| 層 | 役割 | life リポジトリでの対応 |
|----|------|------------------------|
| Raw Sources | 不変の一次情報置き場 | `scrap/archive/` |
| Wiki Layer | LLMが管理する知識ベース | `ideas/wiki/`（このディレクトリ） |
| Schema Layer | 運用ルール・Skillの定義 | `CLAUDE.md` + `.claude/rules/` + [Skill System](skill-system.md) |

## 設計思想のコア

> 「知識ベース維持の面倒な部分は読むことでも考えることでもなく、**ブックキーピング（相互参照の管理）**だ。LLMはこれが得意。」 — Karpathy

- **RAGとの違い**: RAGは毎回原文から再合成。WikiパターンはLLMが一度合成した知識を蓄積・更新する
- **複利的蓄積**: 1記事のインジェストが関連する複数ページを更新し、知識が網目状に広がる
- **人間の限界をカバー**: 人間が諦めがちな「全ページの相互参照の一貫性維持」をLLMが自動化

このアプローチは [AIオーケストレーター](ai-orchestrator-role.md) の考え方と相通じている。人間が「すべてのブックキーピングを手動で維持する」ことは運用の限界だが、LLMエージェントに管理を委譲することで複利的な効果が生まれる。

また、LLMが知識ベースを効果的に維持するためには、[Skill System](skill-system.md)「フォルダ単位のClaudeへの命令セット」を活用することが重要である。**一度教えて何度でも使う** コンセプトにより、wiki更新タスク自体もスキル化でき、会話のたびに同じ指示を繰り返す手間が削減される。

## life リポジトリでの実装

`git scrap` を実行するたびに以下が自動実行される:

1. Haikuが記事内容を分析し関連wikiページを最大3件選定
2. 各ページに新情報を統合し、関連ページへのリンクを自動挿入
3. 新概念・エンティティは最大2件のページとして新規作成
4. `index.md`（目次）と `log.md`（更新履歴）を自動更新

### ニューロン的リンク構造

```
[git scrap 実行]
        ↓
  claude-code.md ──→ mcp.md
        ↓                ↓
  ai-career.md ←── manufacturing-ai.md
```

スクラップのたびに既存ページ間の神経が増え、知識グラフが密になる。

この自動化フロー自体が [Skill System](skill-system.md) として実装されることで、Claude Code、Claude API など複数のプラットフォーム間での一貫性が保たれ、どのインターフェースからでも同じ知識管理パターンが動作する。

## ベストプラクティス

- **粒度の調整**: ページは「説明可能な単一概念」単位で設計し、粗すぎず細すぎずを保つ
- **相互参照の責任**: 新しい情報追加時は必ず関連ページへのバックリンクも忘れずに更新する
- **スキル化の意識**: 繰り返し行う wiki 操作は [Skill System](skill-system.md) として記録し、再利用可能にする
- **定期的なRefactor**: 月単位で古い情報の鮮度チェックと、過度に肥大化したページの分割を検討

## v2 から採用した改善（2026-04-13）

[LLM Wiki v2 提案](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2)のうち、現在の運用規模（シングルユーザー・約130ページ）に適合する4件を採用した：

### 1. Broken Link 自動修正（Auto-Healing）
`git scrap` 後に `lint_wiki.py` が更新ページのリンクを自動検査・修正する。
3段階：① fuzzy match → ② Haiku 照合 → ③ リンク削除（テキスト保持）

### 2. Supersession（情報更新の明示化）
情報が変化・矛盾する場合は `~~旧記述~~` → 新記述 形式で明示。
サイレント上書きを禁止し、「いつどこで情報が変わったか」を追跡可能にする。

### 3. Quality Check（必須セクション・行数上限）
生成されたページが `## 概要` `## 主要な知見` `## 関連ページ` を持つか確認。
200行超のページには分割推奨 WARN を出力。

### 4. Schema Page（スキーマ一元化）
wikiルール・ページ構造・品質基準を [Wiki スキーマ定義](schema.md) に集約。
CLAUDE.md に散在していたルールを Haiku が参照しやすい形に整理。

**不採用の v2 提案**（規模・用途に合わず）: Confidence Scoring・Knowledge Graph・Hybrid Search・Forgetting Curves・Multi-Agent 同期・Governance/Privacy フィルタリング

## 関連ページ

- [Wiki スキーマ定義](schema.md): ページ構造・品質基準・自動化パイプラインの詳細
- [Skill System](skill-system.md): スキル化によるワークフロー再利用
- [AIオーケストレーター](ai-orchestrator-role.md): LLM への管理委譲と複利的知識蓄積
- [Git hooks セキュリティ防御](git-hooks-security-defense.md): git scrap フロー関連

## 更新履歴
- 2026-03-08: 「AIオーケストレーター」という自分の役割
- 2026-03-08: Skillとは何か（基礎）
- 2026-03-08: 3段階の段階的開示（Progressive Disclosure）
- 2026-04-05: [Karpathy LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 2026-04-13: [LLM Wiki v2 提案](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2)から4件採用（Broken Link自動修正・Supersession・Quality Check・Schema Page）
