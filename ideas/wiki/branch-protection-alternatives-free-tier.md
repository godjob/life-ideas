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

### 1. Hook ファ