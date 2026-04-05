# Git Hooksセキュリティ防御：権限スキップ時の多層防御設計

## 概要

Claude Codeなどの権限スキップ機能を使用する場合でも、Gitの仕組みそのものであるGit Hooksは有効という重要な発見に基づくセキュリティ設計です。GitHub FreeのプライベートリポジトリではBranch Protection Rulesが利用できないため、pre-push hookによるローカル検証が最後の防線となります。権限スキップ時の多層防御を実装することで、意図しないforce pushやコード品質低下を防ぎます。

## 主要な知見

- **Git Hooksの権限独立性**: --dangerously-skip-permissions 実行時でも、Gitの標準機能であるGit Hooksは有効に動作する
- **GitHub Free tier の制約**: プライベートリポジトリではBranch Protection Rulesが適用されず、サーバー側の保護が無い状態になる
- **多層防御の必要性**: 権限スキップ時にはローカル側のpre-push hookがセキュリティの最後の砦となる
- **force push の阻止**: pre-push hookを用いてローカルレベルでforce pushを検出・ブロックできる
- **Git Hooks の信頼性**: Gitの仕組みだからこそ、権限体系の外で一貫性を保つ

## 詳細

### 権限スキップとセキュリティのギャップ

[agentic-engineering-supervisor-model.md](agentic-engineering-supervisor-model.md) で示される監督者モデルにおいて、Claude Codeが直接コード実行をする際に --dangerously-skip-permissions フラグを使用することがあります。この権限スキップにより、通常のファイルアクセス権限やシステム保護機構が迂回されます。

しかし重要な発見として、**Git Hooksはこの権限スキップの影響を受けない**という特性があります。これはGit Hooksが Gitコマンド実行そのものの仕組みに組み込まれているため、権限層の上位に存在するからです。

### Branch Protection との併用戦略

GitHub Freeのプライベートリポジトリでは以下の状況が発生します：

- ✗ Branch Protection Rules が使用不可
- ✗ サーバー側での強制ルール適用が出来ない
- ✓ ローカル側のGit Hooksは常に有効

このため、以下のような多層防御設計が必要になります：

```
権限スキップ実行
  ↓
Git push コマンド実行
  ↓
pre-push hook 発動（権限に関係なく実行）
  ↓
Force push 検出 → ブロック
コード品質チェック → 失敗時ブロック
  ↓
リモート反映
```

### pre-push Hook の実装パターン

force pushを阻止するpre-push hookの基本実装：

```bash
#!/bin/bash
# .git/hooks/pre-push

while read local_ref local_sha remote_ref remote_sha; do
  # Force push の検出（local_sha が 0000... の場合は削除、通常フローで阻止可能）
  if [ "$local_sha" != "0000000000000000000000000000000000000000" ] && 
     [ "$remote_sha" != "0000000000000000000000000000000000000000" ]; then
    # リモートが既に存在する場合、fast-forward可能