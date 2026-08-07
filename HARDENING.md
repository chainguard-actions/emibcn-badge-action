<!-- markdownlint-disable -->

# Hardening Report: emibcn--badge-action/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **emibcn--badge-action/v2.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/test.yml references four Actions using mutable tags or branch names instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag or branch is moved to a malicious commit.

Failing references:
- Line 33: `uses: actions/checkout@v3` (tag)
- Line 43: `uses: emibcn/badge-action@v2.0.1` (tag)
- Line 49: `uses: actions/upload-artifact@v3` (tag)
- Line 63: `uses: ad-m/github-push-action@master` (branch)

Locations:

- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:49`
- `.github/workflows/test.yml:63`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key and the single job `test` also has no `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access), violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`

### github-env-injection (severity: high)

The 'Prepare environment' step writes a value derived from `steps.extract_branch.outputs.branch` (a `steps.*.outputs.*` value, which is untrusted per the check scope) into `$GITHUB_OUTPUT` without first sanitizing it with `printf '%s' ... | tr -d '\n\r'`. The env var `BADGE_PATH` is set to `${{ steps.extract_branch.outputs.branch }}/test-badge.svg` and then written via `echo "path=${BADGE_PATH}" >> "${GITHUB_OUTPUT}"`. An attacker who can influence the branch name (e.g. via a crafted branch push) could inject newlines into the output file, potentially overwriting subsequent output entries.

Locations:

- `.github/workflows/test.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

Fixed all three findings in .github/workflows/test.yml:
1. unpinned-uses: Pinned all 4 action references to full SHAs — actions/checkout@v3 → a37ce9120846195fa4ece8f58b268e6043cb2f26, emibcn/badge-action@v2.0.1 → 402bc57ea8b5882e5fda2c5ca751ecf8e0b49394, actions/upload-artifact@v3 → ff15f0306b3f739f7b6fd43fb5d26cd321bd4de5, ad-m/github-push-action@master → 881a6320fdb16eb5318c5054f31c218aec2b324c. Original tags/branch preserved as inline comments.
2. missing-permissions: Added top-level `permissions: contents: write` (write access is required because the workflow commits and pushes badge SVG files to the repository).
3. github-env-injection: In the 'Prepare environment' step, the BADGE_PATH env var (which contains the untrusted branch name from steps.extract_branch.outputs.branch) is now sanitized via `safe_path=$(printf '%s' "${BADGE_PATH}" | tr -d '\n\r')` before being written to GITHUB_OUTPUT, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Extract branch name' step in .github/workflows/test.yml (line 19). The unsanitized write `echo "branch=${GITHUB_REF#refs/heads/}" >> "${GITHUB_OUTPUT}"` was replaced with a two-step approach: first capture the stripped value into a variable sanitized with `printf '%s' ... | tr -d '\n\r'`, then write the safe value to $GITHUB_OUTPUT. This prevents newline injection via attacker-controlled branch/tag names.

