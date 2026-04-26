# How to create a Pull Request for Hermes Agent

> Note: the filename intentionally matches the requested spelling: `HOW_TO_PULL_REQEUST.md`.

This guide describes a practical workflow for contributing changes to `NousResearch/hermes-agent` from a personal fork.

## Repository layout for this machine

The fork is cloned here:

```bash
cd ~/Code/opensource/hermes-agent
```

Expected remotes:

```bash
git remote -v
```

Typical output:

```text
origin    git@github.com:<your-user>/hermes-agent.git (fetch)
origin    git@github.com:<your-user>/hermes-agent.git (push)
upstream  git@github.com:NousResearch/hermes-agent.git (fetch)
upstream  git@github.com:NousResearch/hermes-agent.git (push)
```

Use:

- `origin` for your fork.
- `upstream` for the official Hermes Agent repository.

## 1. Sync your local `main`

Before starting any change, update from upstream:

```bash
cd ~/Code/opensource/hermes-agent

git fetch upstream
git checkout main
git merge --ff-only upstream/main
```

Then update your fork's `main`:

```bash
git push origin main
```

If `main` does not track your fork yet, use:

```bash
git push -u origin main
```

## 2. Create a topic branch

Create a small, focused branch for one change:

```bash
git checkout -b feat/configurable-cron-grace
```

Suggested branch prefixes:

```text
feat/...      new feature
fix/...       bug fix
docs/...      documentation only
test/...      tests only
refactor/...  internal cleanup without behavior change
ci/...        CI or workflow changes
chore/...     maintenance
```

Examples:

```bash
git checkout -b fix/cron-catchup-window
git checkout -b docs/cron-grace-setting
git checkout -b feat/configurable-cron-grace
```

## 3. Make the change

Edit files normally. Keep the PR focused and avoid mixing unrelated changes.

For example, for a cron grace-window improvement, likely files could include:

```text
cron/jobs.py
hermes_cli/config.py
tests/...
website/docs/...
```

Check what changed:

```bash
git status
git diff --stat
git diff
```

## 4. Add or update tests

If changing behavior, add tests. For cron-related work, run at least cron-focused tests:

```bash
python -m pytest tests/ -o 'addopts=' -q -k cron
```

Before opening the PR, ideally run the full suite:

```bash
python -m pytest tests/ -o 'addopts=' -q
```

If the full suite is too expensive locally, document exactly what you ran in the PR body.

## 5. Commit the change

Stage only the files that belong to the PR:

```bash
git add path/to/file1 path/to/file2 tests/path/to/test_file.py
```

Commit with a concise conventional commit message:

```bash
git commit -m "feat: make cron catch-up grace configurable"
```

Useful commit prefixes:

```text
feat:     new user-visible feature
fix:      bug fix
docs:     documentation-only change
test:     tests-only change
refactor: code cleanup without behavior change
ci:       CI/workflow change
chore:    maintenance
```

## 6. Push the branch to your fork

```bash
git push -u origin HEAD
```

## 7. Open the Pull Request

With GitHub CLI:

```bash
gh pr create \
  --repo NousResearch/hermes-agent \
  --base main \
  --head <your-user>:$(git branch --show-current) \
  --title "feat: make cron catch-up grace configurable" \
  --body "## Summary
- Adds a configurable cron catch-up grace window
- Keeps existing default behavior unless configured
- Adds tests for delayed recurring jobs

## Test Plan
- [x] python -m pytest tests/ -o 'addopts=' -q -k cron"
```

Replace `<your-user>` with your GitHub username.

Alternative: open GitHub in the browser and create a PR from:

```text
<your-user>:<branch-name> -> NousResearch:main
```

## 8. Monitor CI

With GitHub CLI:

```bash
gh pr checks --repo NousResearch/hermes-agent --watch
```

If CI fails, inspect logs:

```bash
gh run list --repo NousResearch/hermes-agent --branch $(git branch --show-current) --limit 5
gh run view --repo NousResearch/hermes-agent <RUN_ID> --log-failed
```

Then fix, commit, and push again:

```bash
# edit files
python -m pytest tests/ -o 'addopts=' -q -k cron

git add <fixed-files>
git commit -m "fix: address CI failure"
git push
```

The PR updates automatically.

## 9. Respond to review feedback

For requested changes:

```bash
# edit files
git add <files>
git commit -m "fix: address review feedback"
git push
```

If you need to sync with latest upstream while the PR is open:

```bash
git fetch upstream
git rebase upstream/main
git push --force-with-lease
```

Use `--force-with-lease`, not plain `--force`.

## 10. After merge

Clean up local and remote branches:

```bash
git checkout main
git fetch upstream
git merge --ff-only upstream/main
git push origin main

git branch -d <branch-name>
git push origin --delete <branch-name>
```

## PR quality checklist

Before opening a PR, verify:

```text
[ ] Branch is based on fresh `upstream/main`.
[ ] Change is small and focused.
[ ] No secrets, local paths, tokens, or credentials in the diff.
[ ] Tests were added or updated for behavior changes.
[ ] Relevant tests pass locally.
[ ] `git diff` contains only intended changes.
[ ] PR body has Summary and Test Plan sections.
```

## Example PR body

```markdown
## Summary
- Adds `cron.max_grace_seconds` to make recurring cron catch-up configurable
- Keeps default max grace at 2 hours for backward compatibility
- Documents the behavior and adds regression tests

## Test Plan
- [x] python -m pytest tests/ -o 'addopts=' -q -k cron
```
