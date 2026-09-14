# Wiki スキーマ定義

## 概要

`ideas/wiki/` ディレクトリの運用ルール・ページ構造・品質基準を一元定義するスキーマ文書。
Haiku によるページ生成・更新の際にこのスキーマに従うことで一貫した品質を保つ。

## ページ構造（必須セクション）

| セクション | 必須 | 説明 |
|-----------|------|------|
| front matter | ✅ | `category` / `tags`（下記「front matter」参照。本文の前に置く） |
| `# タイトル` | ✅ | 日本語タイトル（H1、front matter 直後の空行の次） |
| `## 概要` | ✅ | テーマの概要を 2〜3 文 |
| `## 主要な知見` | ✅ | インジェストから得た知見を箇条書き |
| `## 実装パターン` | 任意 | 具体的な実装例・手順 |
| `## アーキテクチャパターン` | 任意 | 設計パターンの図解や分類 |
| `## 運用設計の考慮事項` | 任意 | 運用上の注意点 |
| `## 関連ページ` | ✅ | 関連ページへのリンク一覧 |
| `## 更新履歴` | ✅ | インジェスト日・出典を記録 |

## front matter（category / tags）

各ページの先頭に、カテゴリとタグを YAML 風の front matter で持つ。
ダッシュボードのナビ（カテゴリ・タグで近い話題をまとめる）のための機械可読メタデータで、
スラッグのキーワード一致による従来の分類（48% が「その他」）を置き換えるもの。

```markdown
---
category: 製造業・業務DX
tags: [因果推論, 設備保全, 品質管理]
---

# ページタイトル
```

| 項目 | 規約 |
|------|------|
| 形式 | ファイル先頭から `---` / `category: <値>` / `tags: [a, b, c]` / `---` / 空行 / 本文（`# タイトル` から）。キー順は category → tags 固定 |
| category | 下記11種のいずれか。判定不能なら `その他` |
| tags | 1〜5個（無ければ `[]`）。各タグ12文字以内、`,` `[` `]` `:` を含まない、前後空白なし。日本語・英数字どちらも可（全角英数は NFKC で半角に正規化） |
| タグの選び方 | **既存タグの再利用を優先**。同じ概念に別表記のタグを増やさない（LLM には既存タグ上位80件を出現数付きで渡している） |

### カテゴリ語彙（11種・この順序）

`Claude/AI開発ツール` / `AIエージェント設計` / `製造業・業務DX` / `AI戦略・社会` / `技術実装・エンジニアリング` /
`ビジネス・組織` / `キャリア・働き方` / `プロンプト・AI活用` / `個人OS・知識管理` / `資産・投資` / `健康・習慣`

語彙の唯一の正は `scripts/git_scrap.py` の `WIKI_CATEGORIES`。ビューア（`index.html`）と
`backfill_wiki_tags.py` がこの文字列に依存しているため、変更時は3者を同時に更新する。

### 付与のタイミング

- **新規・更新ページ**: `git scrap`（`update_wiki()`）が生成後に `classify_wiki_page()` で自動付与する。
  front matter の無いレガシーページは、更新されたタイミングで付与される
- **既存ページの一括付与**: `python3 scripts/backfill_wiki_tags.py`（`--dry-run` で確認、`--force` で再分類）。
  本文は1バイトも変えず、front matter を先頭に足すだけ
- 分類は LLM（Gemini 優先）。category が語彙外なら1回再問合せし、それでもダメならスラッグの
  キーワード一致（`WIKI_CATEGORY_MAP`）にフォールバックする

### 利用側

- **ビューア（index.html）** は描画時に front matter を除去して本文だけを表示する
- **`dashboard/wiki-graph.json`** のノードは `group` に category、`tags` に tags を反映する
  （front matter が無いページは `group` をスラッグから判定、`tags` は `[]`）
- `index.md` の列構成は変えない（`| [title](slug.md) | 登録日 | 更新日 | 概要 |`）
- `lint_wiki.py --quality` で front matter 未付与・語彙外 category を警告として確認できる

## スラッグ命名規則

- 英数字・ハイフンのみ（`claude-code.md`, `manufacturing-ai.md` など）
- 日本語不可・スペース不可
- 既存スラッグと重複しないこと
- テーマを簡潔に表現（20文字以内推奨）

## リンクフォーマット

- **本文中リンク**: `[日本語タイトル](slug.md)` 形式（スラッグをリンクテキストに使わない）
- **関連ページセクション**: `- [日本語タイトル](slug.md): 一言説明`
- **外部リンク**: `[短いタイトル](https://...)` 形式
- 存在しないスラッグへのリンクは禁止（`lint_wiki.py --fix` で自動修正）

## 品質基準

| 項目 | 基準 |
|------|------|
| 必須セクション | `## 概要` `## 主要な知見` `## 関連ページ` の 3 セクションを含むこと |
| 行数上限 | 本文 + 全セクション合計 **200行以下**（超過時は分割を検討） |
| 途切れ禁止 | 括弧未閉じで終了しないこと（`generate_wiki_content` が自動検出・再生成） |
| Supersession | 情報が変化・矛盾する場合は `~~旧記述~~` → 新記述 形式で明示（サイレント上書き禁止） |
| リンク健全性 | 存在しないスラッグへのリンクをゼロに保つ（`git scrap` 後に `lint_wiki.py` が自動修正） |

## Supersession（情報更新の明示化）

新情報が既存の記述と矛盾・変化する場合は、更新を透明化する：

```markdown
~~旧バージョンは v1.2 までしか対応していない~~ → v2.0 以降で対応済み（2026-04-13 更新）
```

サイレントな上書きを禁止することで「どこで情報が変わったか」を追跡可能にする。

## ファイル管理

| ファイル | 役割 | 自動更新 |
|---------|------|---------|
| `index.md` | 全ページ目次（スラッグ→タイトルマップ） | ✅ `_update_wiki_index()` |
| `log.md` | ingest 履歴（日付・出典・更新ページ） | ✅ `update_wiki()` |
| `schema.md` | このスキーマ文書 | 手動更新 |
| `*.md`（その他） | テーマ別 wiki ページ | ✅ Haiku による生成・更新 |

## 自動化パイプライン

```
git scrap 実行
    ↓
1. analyze_scrap()     — 要約・タグ・気づき生成
2. update_wiki()       — 関連ページ選定・更新・新規作成
3. generate_wiki_content() — 生成 + 途切れ検出 + 品質チェック
3.5 classify_wiki_page() — category / tags を決めて front matter を付与
4. lint_pages()        — broken link 検出・自動修正（更新ページのみ）
5. _update_wiki_index() — index.md 再生成
6. log.md 追記
7. generate_wiki_graph() — dashboard/wiki-graph.json 再生成（group / tags に front matter を反映）
```

## スキーマの変更手順

1. このファイル（`schema.md`）を更新
2. `git_scrap.py` のプロンプト（`update_prompt` / `create_prompt`）を必要に応じて修正
3. 既存ページへの遡及適用が必要な場合は `scripts/repair_wiki.py` を参照

## 関連ページ

- [LLM Wiki コンセプト](llm-wiki-concept.md): Karpathy の LLM Wiki パターンと life リポジトリでの実装
- [Skill System](skill-system.md): スキル化によるワークフロー再利用
- [AIオーケストレーター](ai-orchestrator-role.md): LLM への管理委譲と複利的知識蓄積

## 更新履歴

- 2026-04-13: 初版作成（LLM Wiki v2 提案から採用した品質基準・Supersession・lint自動修正を統合）
- 2026-09-14: front matter（category / tags）セクションを追加。`backfill_wiki_tags.py` による一括付与と `wiki-graph.json` への反映を明記
