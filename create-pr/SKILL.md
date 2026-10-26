---
name: create-pr
description: Create or update a pull request from the current branch using GitHub CLI, then merge after checks pass and publication is confirmed. Use when user asks to create a PR or merge a branch.
disable-model-invocation: true
---

# Create and merge a pull request

Use GitHub CLI (`gh`) for all GitHub operations. Run Git commands from the selected repository, not an unrelated workspace. Preserve uncommitted changes; publish only the intended commits. A failed required local check blocks publication.

## 1. Identify the repository and branches

1. Verify `command -v gh` and `gh auth status`. If unavailable or unauthenticated, stop and report the required action. Ask the user to authenticate directly in their terminal; never request tokens in chat.
2. Read the Git root, remote, current branch and HEAD with `git rev-parse --show-toplevel`, `git remote -v`, `git branch --show-current` and `git rev-parse HEAD`. Record the validated HEAD.
3. Resolve `<repo>` as `OWNER/REPO` from the verified remote. Use `--repo <repo>` for every `gh` command that supports it.
4. Use the repository's documented integration branch as `<base>`, including a base already verified by the calling workflow. Otherwise inspect `gh repo view <repo> --json defaultBranchRef`. Ask when the target is ambiguous. Stop if the current branch is the base.
5. Find existing PRs with `gh pr list --repo <repo> --head <branch> --base <base> --state all --json number,url,title,state,headRefOid`. Reuse an open PR. If a matching PR is already merged or closed, inspect it before deciding whether any work remains. Ask if multiple open PRs match.

Complete when the repository, head, base, validated SHA and existing PR state are known.

## 2. Parse the branch name

The branch must start with a recognized type followed by `/`: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `build`, `ci`, `perf`, or `revert`. Otherwise stop and ask the user to rename it.

After removing the type prefix, extract an optional leading ticket ID matching `[A-Z][A-Z0-9]*-[0-9]+`, such as `ALTER-65` or `DEEP-13`. The remaining text is the description. Replace hyphens with spaces and capitalize the description words.

Generate the title:

- With a ticket: `[TICKET] type: Description`, for example `[ALTER-65] feat: Image Expandable Tabs`.
- Without a ticket: `type: Description`, for example `chore: Update Deps`.

## 3. Confirm publication

Show these settings before any push, PR write or merge:

```text
Repository: <repo>
Branch:     <branch>
Title:      <generated-title>
Base:       <base>
Method:     merge
```

Disclose known merge-triggered deployments and unresolved caveats. Use the VS Code question tool with an explicit "Continue" option and wait for confirmation. Confirmation covers only the presented repository and settings. If the question tool is unavailable, ask in chat and stop until the user confirms.

## 4. Create or update the PR

1. Confirm the local HEAD still matches the validated SHA. Push committed changes with `git push -u <remote> <branch>`, without force. Preserve uncommitted files.
2. Repeat the open-PR lookup before creation to avoid duplicates.
3. If no PR exists, use `gh pr create --repo <repo> --head <branch> --base <base> --title <title> --body-file <body-file>`. Follow the repository's PR template. Include the change summary, validation and caveats; keep temporary body files outside tracked files.
4. If an open PR exists, read it with `gh pr view <number> --repo <repo> --json number,url,title,state,headRefOid,baseRefName,body`. Update a differing title with `gh pr edit <number> --repo <repo> --title <title>`. Preserve existing body content when adding validation or caveats.
5. Record the PR number, URL and remote head SHA. If the remote head differs from the validated SHA, stop and validate the changed revision before proceeding.

Complete when one open PR has the intended head, base and title.

## 5. Check and merge

1. Read `gh pr view <number> --repo <repo> --json state,headRefOid,mergeable,mergeStateStatus,reviewDecision,statusCheckRollup` and `gh pr checks <number> --repo <repo> --json name,state,bucket,link`. Distinguish a confirmed empty check list from an API or authorization error.
2. Pending checks are not passed. Wait with `gh pr checks <number> --repo <repo> --watch --fail-fast`. Report failed, cancelled or unavailable checks as blockers. Report an interrupted wait as pending, not success.
3. Before merging, refresh the PR state and confirm the head SHA still matches the validated SHA, all checks have passed or are legitimately skipped, required reviews are satisfied and GitHub reports it mergeable without blocking protections. An unknown mergeability state requires another read, not a merge attempt.
4. Merge using `gh pr merge <number> --repo <repo> --merge --subject <title> --match-head-commit <validated-sha>`. Use ordinary merge, without administrator bypass, squash, rebase or branch deletion.
5. Verify with `gh pr view <number> --repo <repo> --json state,mergedAt,mergeCommit,url`. Completion requires `state=MERGED`, a merge timestamp and merge SHA. A queued merge or enabled auto-merge remains pending until verified merged.

## Result

```text
Status:     Merged | Pending checks | Blocked | Already merged
PR:         <url>
Title:      <title>
Branch:     <branch>
Base:       <base>
Merge SHA:  <sha, when merged>
```

For pending or blocked results, include the decisive error and required next action. Preserve the branch and commits.
