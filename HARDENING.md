<!-- markdownlint-disable -->

# Hardening Report: emibcn--badge-action/v2.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **emibcn--badge-action/v2.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file references four Actions by mutable tag or branch instead of a pinned 40-character commit SHA, making the workflow vulnerable to supply-chain attacks if any of those tags are moved or hijacked:
- Line 31: `uses: actions/checkout@v6` (tag)
- Line 43: `uses: emibcn/badge-action@v2.0.3` (tag)
- Line 49: `uses: actions/upload-artifact@v7` (tag)
- Line 62: `uses: ad-m/github-push-action@master` (branch)

Locations:

- `.github/workflows/test.yml:31`
- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:49`
- `.github/workflows/test.yml:62`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single `test` job also has no `permissions:` key. Without explicit permissions, the job runs with the default token permissions (which may include write access to contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`

### github-env-injection (severity: high)

Two `run:` steps write values derived from workflow-controlled environment variables to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. Line 17 (`Extract branch name` step): `echo "branch=${GITHUB_REF#refs/heads/}" >> "${GITHUB_OUTPUT}"` — `GITHUB_REF` is a process env var set by the calling workflow and is treated as untrusted. A branch name containing a newline could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

2. Lines 24–26 (`Prepare environment` step): `BADGE_PATH` is set from `${{ steps.extract_branch.outputs.branch }}` (itself derived from `GITHUB_REF`) and then written via `echo "path=${BADGE_PATH}" >> "${GITHUB_OUTPUT}"` without sanitization. The same injection risk applies.

Locations:

- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

Fixed all three findings in .github/workflows/test.yml:

1. unpinned-uses: Pinned all 4 action references to full 40-char commit SHAs with tag comments preserved:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - emibcn/badge-action@v2.0.3 → @808173dd03e2f30c980d03ee49e181626088eee8
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c

2. missing-permissions: Added 'permissions: contents: write' at both top-level and job level (contents: write is required for pushing badge commits).

3. github-env-injection: Sanitized both GITHUB_OUTPUT writes that used untrusted values:
   - 'Extract branch name' step: moved github.ref into env var, strip prefix, then pipe through 'tr -d \n\r' before writing to GITHUB_OUTPUT.
   - 'Prepare environment' step: sanitize BADGE_PATH (derived from branch name) with 'tr -d \n\r' before writing path= to GITHUB_OUTPUT.

