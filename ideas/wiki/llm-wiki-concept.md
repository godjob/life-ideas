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
| Schema Layer | 運用ルールの定義 | `CLAUDE.md` + `.claude/rules/` |

## 設計思想のコア

> 「知識ベース維持の面倒な部分は読むことでも考えることでもなく、**ブックキーピング（相互参照の管理）**だ。LLMはこれが得意。」 — Karpathy

- **RAGとの違い**: RAGは毎回原文から再合成。WikiパターンはLLMが一度合成した知識を蓄積・更新する
- **複利的蓄積**: 1記事のインジェストが関連する複数ページを更新し、知識が網目状に広がる
- **人間の限界をカバー**: 人間が諦めがちな「全ページの相互参照の一貫性維持」をLLMが自動化

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

## 将来の拡張（Phase 2以降）

- `git wiki-lint`: 矛盾・孤立ページ・古くなった記述を定期検出
- 月次まとめとwikiの自動連携（weeklyがwikiを参照）
- Obsidian互換表示（`[[ページ名]]` 形式への変換）

## 関連ページ

- [index.md](index.md) — このwikiの全ページ目次
- [log.md](log.md) — ingest履歴ログ

## 更新履歴

- 2026-04-05: 初期作成 — [Karpathy LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
