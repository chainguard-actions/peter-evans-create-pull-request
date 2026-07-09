<!-- markdownlint-disable -->

# Hardening Report: peter-evans--create-pull-request/v5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--create-pull-request/v5** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags (e.g. @v2, @v3, @v5) instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Failing references include: ci.yml — actions/checkout@v3, actions/setup-node@v3, actions/upload-artifact@v3, actions/download-artifact@v3, peter-evans/close-pull@v3, peter-evans/find-comment@v2, peter-evans/create-or-update-comment@v3, peter-evans/create-pull-request@v5; automerge-dependabot.yml — peter-evans/enable-pull-request-automerge@v3; cpr-example-command.yml — actions/checkout@v3, peter-evans/create-or-update-comment@v3; slash-command-dispatch.yml — peter-evans/slash-command-dispatch@v3; update-major-version.yml — actions/checkout@v3.

Locations:

- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:29`
- `.github/workflows/ci.yml:31`
- `.github/workflows/ci.yml:42`
- `.github/workflows/ci.yml:44`
- `.github/workflows/ci.yml:46`
- `.github/workflows/ci.yml:62`
- `.github/workflows/ci.yml:73`
- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:91`
- `.github/workflows/ci.yml:96`
- `.github/workflows/ci.yml:101`
- `.github/workflows/automerge-dependabot.yml:7`
- `.github/workflows/cpr-example-command.yml:7`
- `.github/workflows/cpr-example-command.yml:43`
- `.github/workflows/slash-command-dispatch.yml:7`
- `.github/workflows/update-major-version.yml:18`

### script-injection (severity: high)

Sub-rule (a): In update-major-version.yml, two run: steps directly interpolate ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} into shell commands. These are workflow_dispatch inputs that can be controlled by whoever triggers the workflow. Injecting shell metacharacters via these inputs could lead to arbitrary command execution. Offending lines: `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` and `run: git push origin ${{ github.event.inputs.main_version }} --force`.

Locations:

- `.github/workflows/update-major-version.yml:24`
- `.github/workflows/update-major-version.yml:26`

### missing-permissions (severity: medium)

Four workflow files have no top-level permissions: block and no job-level permissions: blocks, meaning they inherit the default repository permissions (which can be write-all). Each should declare minimal required permissions explicitly. Affected files: automerge-dependabot.yml, cpr-example-command.yml, slash-command-dispatch.yml, update-major-version.yml.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/cpr-example-command.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files:

1. unpinned-uses: Pinned all action references to full 40-character commit SHAs with tag comments:
   - actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744
   - actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610
   - actions/upload-artifact@v3 → @ff15f0306b3f739f7b6fd43fb5d26cd321bd4de5
   - actions/download-artifact@v3 → @9bc31d5ccc31df68ecc42ccf4149144866c47d8a
   - peter-evans/close-pull@v3 → @a192af8d70f2d49c49643134605c3b73d4f80fae
   - peter-evans/find-comment@v2 → @a54c31d7fa095754bfef525c0c8e5e5674c4b4b1
   - peter-evans/create-or-update-comment@v3 → @23ff15729ef2fc348714a3bb66d2f655ca9066f2
   - peter-evans/create-pull-request@v5 → @4e1beaa7521e8b457b572c090b25bd3db56bf1c5
   - peter-evans/enable-pull-request-automerge@v3 → @a660677d5469627102a1c1e11409dd063606628d
   - peter-evans/slash-command-dispatch@v3 → @f996d7b7aae9059759ac55e978cff76d91853301

2. script-injection: In update-major-version.yml, moved ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} out of run: shell commands into env: blocks, referencing them as $MAIN_VERSION and $TARGET (double-quoted) in the shell.

3. missing-permissions: Added minimal permissions blocks to automerge-dependabot.yml (pull-requests: write, contents: write), cpr-example-command.yml (contents: write, pull-requests: write), slash-command-dispatch.yml (issues: read, pull-requests: read), and update-major-version.yml (contents: write).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/cpr-example-command.yml 'Check output' step. Moved ${{ steps.cpr.outputs.pull-request-number }} and ${{ steps.cpr.outputs.pull-request-url }} from the run: shell string into an env: block as PR_NUMBER and PR_URL, then referenced them as plain shell variables in the run: script.

