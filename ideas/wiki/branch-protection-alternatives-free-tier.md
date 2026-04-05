# GitHub Free プライベートリポジトリのBranch Protection代替：pre-push hook活用

## 概要

GitHub Freeではプライベートリポジトリに対してBranch Protection Rulesが適用されないため、force pushなどの危険な操作を防ぐことができません。本ページは、pre-push hookを活用してローカルで強制的に保護ルールを実装し、多層防御を構築する実践的なアプローチを解説します。Claude Codeの`--dangerously-skip-permissions`フラグ使用時でも、Git hooksはGitの仕組みそのものであるため有効に機能することが重要な発見です。

## 主要な知見

- GitHub FreeのプライベートリポジトリではBranch Protection Rules機能が利用不可であることを確認
- Claude Codeで権限スキップ（`--dangerously-skip-permissions`）使用時でも、Git hooksはGitの基盤仕組みであるため有効に動作する
- pre-push hookによってforce push（`git push -f`）をローカル段階で阻止可能
- [Git Hooksセキュリティ防御](git-hooks-security-defense.md)として、ローカル保護と権限制御の組み合わせが効果的
- Hook機械化による継続的な防御維持が重要（[CLAUDE.md自動育成](claude-md-auto-cultivation-hooks.md)参照）

## 背景：GitHub Free の制限

GitHub Freeプランでは、プライベートリポジトリに対して以下の保護機能が利用できません：

- **Branch Protection Rules**: 特定ブランチへのダイレクト push を防止
- **Required Status Checks**: CI/CD パイプラインの実行を強制
- **Code Review Requirements**: Pull Request承認を必須化
- **Dismiss Stale Pull Request Approvals**: レビュー有効期限の管理

これにより、意図しない force push や直接 commit によるリポジトリ破損のリスクが高まります。

## Pre-push Hook による多層防御

### Hook の役割

pre-push hookは、`git push`コマンド実行時に**ローカルマシンで**以下を検証します：

- Force push の試行検出
- マージコミット以外の危険な操作
- 保護対象ブランチへのアクセス制御

```bash
#!/bin/bash
# .git/hooks/pre-push

protected_branches="^(main|develop)$"
force_push_flag="$4"

while read local_ref local_sha remote_ref remote_sha
do
    branch_name=$(echo "$remote_ref" | sed 's@refs/heads/@@')
    
    # 保護対象ブランチへの force push 検出
    if [[ "$branch_name" =~ $protected_branches ]]; then
        if [[ "$force_push_flag" == "1" ]]; then
            echo "❌ Force push to protected branch '$branch_name' is forbidden"
            exit 1
        fi
    fi
done

exit 0
```

### Claude Code との相互作用

Claude Codeで`--dangerously-skip-permissions`フラグを使用する場合：

```bash
claude-code --dangerously-skip-permissions
```

このフラグは**Claude Codeの権限チェック**をスキップするものであり、Gitの基盤仕組みであるhooksには影響しません。したがって：

1. Claude Codeが直接 `git push -f` を実行しても
2. .git/hooks/pre-push が有効であれば
3. push操作がローカルで阻止される

## 実装パターン

### 1. Hook ファイルの配置

`.git/hooks/pre-push`をリポジトリルートに配置し、実行権限を付与します：

```bash
chmod +x .git/hooks/pre-push
```

git hooksはリポジトリ毎に存在するため、チーム全体で同じ保護ルールを適用する場合は以下の方法を推奨します：

```bash
# hooks用ディレクトリをVersion Controlに含める
mkdir -p .githooks
cp .git/hooks/pre-push .githooks/pre-push
git config core.hooksPath .githooks
git add .githooks/
```

### 2. 複数ブランチ保護の拡張実装

```bash
#!/bin/bash
# .githooks/pre-push

protected_branches="^(main|develop|release)$"
local_ref=$2
remote_ref=$3
force_push=$4

# リモートリファレンスのブランチ名を抽出
branch_name=$(echo "$remote_ref" | sed 's@refs/heads/@@')

# 保護対象ブランチの確認
if [[ "$branch_name" =~ $protected_branches ]]; then
    # Force push の検出
    if [[ "$force_push" == "1" ]]; then
        echo "❌ ERROR: Force push to '$branch_name' is strictly forbidden"
        echo "💡 Tip: Use 'git revert' or create a new commit instead"
        exit 1
    fi
    
    # ローカルコミットとリモートコミットの差分検出
    if ! git merge-base --is-ancestor "$remote_ref" "$local_ref" 2>/dev/null; then
        echo "⚠️  WARNING: Non-fast-forward push detected on protected branch '$branch_name'"
        read -p "Continue anyway? (y/N): " -n 1 -r
        echo
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            exit 1
        fi
    fi
fi

exit 0
```

### 3. 開発ブランチでの柔軟な運用

開発用ブランチ（feature, bugfix等）では制限を緩和し、main/develop のみ厳格に保護する構成が実用的です：

```bash
#!/bin/bash
# .githooks/pre-push

# 厳格保護ブランチ
strict_branches="^(main|develop)$"

# 警告対象ブランチ
warning_branches="^(staging|pre-release)$"

branch_name=$(echo "$3" | sed 's@refs/heads/@@')

if [[ "$branch_name" =~ $strict_branches ]] && [[ "$4" == "1" ]]; then
    echo "❌ Force push to '$branch_name' is forbidden"
    exit 1
elif [[ "$branch_name" =~ $warning_branches ]] && [[ "$4" == "1" ]]; then
    echo "⚠️  Force push to '$branch_name' detected - be careful!"
fi

exit 0
```

## 検証と動作確認

### Pre-push hook の有効性確認

```bash
# force push を試行（阻止されることを確認）
git push -f origin main
# 出力: ❌ Force push to protected branch 'main' is forbidden

# 通常の push は成功
git push origin feature-branch
# 出力: [正常に push される]
```

### Claude Code での動作確認

```bash
# Claude Code が力押し push を試みた場合
claude-code --dangerously-skip-permissions
# git push -f を実行しても、pre-push hook により阻止される
```

## 制限事項と補完策

### Pre-push Hook の限界

- **ローカル環境のみ保護**: リポジトリへの直接アクセス（SSH接続等）は対象外
- **Hook回避可能**: `--no-verify`フラグで無視できるため、絶対的防御ではない
- **チーム全体への強制**: 全メンバーが同じ設定を保有する必要

### 補完的な多層防御戦略

1. **GitHub Actions による CI/CD検証**: リモート側でのコミット検証
2. **Deploy キーの権限制限**: プッシュ権限を最小限に設定
3. **Audit ログの監視**: リポジトリへのアクセス記録を定期確認
4. **[CLAUDE.md による行動制約](claude-md-auto-cultivation-hooks.md)**: AI tool のシステムプロンプトに保護ルール記載

## ベストプラクティス

- **チーム共有**: `.githooks/`ディレクトリをリポジトリに含める
- **ドキュメント化**: 各Hookの役割と制限事項を README に記載
- **定期レビュー**: Hook設定の妥当性を月次で確認
- **権限との併用**: Pre-push Hook と GitHub の細粒度権限を組み合わせる

## 関連ページ

- [Git Hooksセキュリティ防御](git-hooks-security-defense.md)
- [CLAUDE.md自動育成](claude-md-auto-cultivation-hooks.md)
- GitHub Free プラン制限事項（公式ドキュメント）

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2024-01-15 | 初版作成。pre-push hook基本実装とClaude Code相互作用を記載 |
| 2024-01-20 | 複数ブランチ保護の拡張実装パターンと検証方法を追加 |
| 2024-01-25 | 多層防御戦略の補完策セクションを追加。ベストプラクティス統合 |