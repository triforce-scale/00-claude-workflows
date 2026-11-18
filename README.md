# 00-claude-workflows

Central workflow for running Claude (Claude Code GitHub Actions) across every repository in the `triforce-scale` organization.

Claude runs **only when someone mentions `@claude`** in an issue or pull request. There is no automatic review.

> **Do not rename this repository.** Every repository calls `triforce-scale/00-claude-workflows/.github/workflows/claude-reusable.yml@main`, and GitHub Actions does not follow redirects for reusable workflows. Renaming it breaks `@claude` in all repositories.

## Everyday use

Write `@claude` followed by a request in any of these places:

- A comment on a pull request's **Conversation** tab
- A comment on a line of code in a pull request
- The comment box when submitting a pull request review
- A comment on an issue
- The title or body of a new issue

Examples:

```text
@claude Please review this PR.
@claude Implement this issue and open a PR.
```

Only users with write access to the repository can trigger Claude.

## Things to remember

### When you create a new repository

New repositories are **not** set up automatically. Run the rollout script again on `eris`.

1. Set the token for the current terminal session only. The token does not appear on screen or in your shell history.

   ```bash
   cd ~/claude-gh
   read -rs CLAUDE_CODE_OAUTH_TOKEN   # paste the token, then press Enter
   export CLAUDE_CODE_OAUTH_TOKEN
   echo ${#CLAUDE_CODE_OAUTH_TOKEN}   # prints the length only; 0 means it is empty
   ```

2. Run the script without `--force`, so repositories that are already set up are skipped.

   ```bash
   ./rollout.sh triforce-scale --set-secret --dry-run   # preview
   ./rollout.sh triforce-scale --set-secret             # apply
   ```

3. Clear the token from the terminal session. This does not affect the secrets stored on GitHub.

   ```bash
   unset CLAUDE_CODE_OAUTH_TOKEN
   ```

### When the token expires (around November 2027)

The `CLAUDE_CODE_OAUTH_TOKEN` was created on 2026-11-18 with `claude setup-token` and is valid for **one year**. Once it expires, Claude stops replying to `@claude`, and the runs on the **Actions** tab fail with an authentication error.

Renew it shortly before it expires:

1. Create a new token on a machine with Claude Code installed:

   ```bash
   claude setup-token
   ```

   The token is shown only once and is not saved anywhere, so store it somewhere safe, such as a password manager.

2. Register the new token in every repository by following the steps in [When you create a new repository](#when-you-create-a-new-repository). `--set-secret` overwrites the existing secret in every repository.

The workflow files do not need to change.

### Add a `CLAUDE.md` to each repository

Put a `CLAUDE.md` file in the root of a repository to tell Claude how to work there: for example, "write reviews in Japanese", the coding conventions to follow, or the areas reviews should focus on. Claude reads it on every run.

## Setup notes

- **Authentication:** a Claude Pro subscription token (`CLAUDE_CODE_OAUTH_TOKEN`). Usage counts against the same Pro limits as Claude chat and Claude Code.
- **Where the secret is stored:** a repository secret in each repository. The organization is on GitHub Free, and organization-level secrets are not available to private repositories on that plan.
- **Limits per run:** `--max-turns 30` and `timeout-minutes: 40`, set in `.github/workflows/claude-reusable.yml`.
- **GitHub App:** the official Claude GitHub App is installed on the organization with access to all repositories.
