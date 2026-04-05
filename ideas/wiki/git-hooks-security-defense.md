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
    # リモートが既に存在する場合、fast-forward可能か確認
    if ! git merge-base --is-ancestor "$remote_sha" "$local_sha"; then
      echo "Error: Force push detected on $remote_ref"
      echo "このブランチはfast-forwardのみ許可されています"
      exit 1
    fi
  fi
done

# コード品質チェック（オプション）
echo "Running pre-push code quality checks..."
npm run lint || exit 1
npm run test || exit 1

exit 0
```

### 設定と有効化

Git Hooksを確実に有効化するための手順：

```bash
# フック設定ディレクトリの作成
mkdir -p .git/hooks

# pre-push フックを配置
cp scripts/pre-push .git/hooks/pre-push
chmod +x .git/hooks/pre-push

# フックが実行可能か確認
ls -la .git/hooks/pre-push
```

プロジェクトメンバー全員がこれらのフックを使用するため、以下の運用方法を推奨します：

```bash
# hooks ディレクトリをリポジトリ管理下に置く
mkdir -p git-hooks
mv .git/hooks/pre-push git-hooks/

# .gitignore で .git/hooks を除外対象から除去
# セットアップスクリプトを作成
```

### 権限スキップ時の防御メカニズム

権限スキップ実行時でも以下の防御が機能します：

| 防御層 | メカニズム | 権限スキップの影響 | 効果 |
|--------|-----------|------------------|------|
| pre-push hook | Gitコマンド実行時の割り込み | **影響なし** | Force push ブロック ✓ |
| コード品質チェック | lint/test の自動実行 | **影響なし** | 品質低下防止 ✓ |
| ブランチ保護 | サーバー側ルール | 影響あり | GitHub Free で不可 ✗ |
| ファイルパーミッション | OS レベルアクセス制御 | 影響あり | --dangerously-skip-permissions で迂回可能 ✗ |

## ベストプラクティス

### 1. Hook の信頼性向上

```bash
# pre-push hook を厳格に設定
set -euo pipefail

# 複数の検証段階を実装
validate_commits()
validate_code_quality()
validate_branch_rules()
```

### 2. チームへの周知

- Git Hooksの役割と重要性を文書化
- セットアップガイドに明記
- 開発環境のOnboarding に組み込む

### 3. ログと監視

```bash
# Hook実行ログを記録
{
  echo "[$(date)] pre-push hook executed"
  echo "Branch: $remote_ref"
  echo "Status: $?"
} >> .git/hooks.log
```

### 4. 段階的な導入

1. **警告フェーズ**: force pushを警告するが許可
2. **試験フェーズ**: チーム内で検証
3. **強制フェーズ**: force pushを完全にブロック

## 制限事項と注意点

- Git Hooksは **ローカル環境でのみ有効** - リモートリポジトリでの強制は不可
- `--no-verify` フラグで回避可能 - ただし意図が明確
- フックのバイパスは管理者権限ではなく、開発者の判断による
- GitHub Free での強力な保護が必要な場合は上位プランへの移行を検討

## 関連ページ

- [agentic-engineering-supervisor-model.md](agentic-engineering-supervisor-model.md)
- Git 公式ドキュメント: Customizing Git - Git Hooks
- GitHub Branch Protection Rules 設定ガイド

## 更新履歴
- 2026-03-21: --dangerously-skip-permissions のリスク対策
