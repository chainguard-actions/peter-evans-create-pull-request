<!-- markdownlint-disable -->

# Hardening Report: peter-evans--create-pull-request/v8.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--create-pull-request/v8.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag-based or version-based refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved.

Failing references:
- automerge-dependabot.yml: `peter-evans/enable-pull-request-automerge@v3`
- ci.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/upload-artifact@v6` (×2), `actions/download-artifact@v7` (×2), `peter-evans/close-pull@v3`, `peter-evans/find-comment@v4`, `peter-evans/create-or-update-comment@v5`, `peter-evans/create-pull-request@v8`
- cpr-example-command.yml: `actions/checkout@v6`, `peter-evans/create-or-update-comment@v5`
- slash-command-dispatch.yml: `peter-evans/slash-command-dispatch@v5`
- update-major-version.yml: `actions/checkout@v6`

Locations:

- `.github/workflows/automerge-dependabot.yml:8`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:39`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:100`
- `.github/workflows/cpr-example-command.yml:8`
- `.github/workflows/cpr-example-command.yml:40`
- `.github/workflows/slash-command-dispatch.yml:8`
- `.github/workflows/update-major-version.yml:18`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings, enabling script injection.

(a) update-major-version.yml — `workflow_dispatch` inputs are interpolated directly into git commands:
  - `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}`
  - `run: git push origin ${{ github.event.inputs.main_version }} --force`
  An attacker with dispatch access could inject arbitrary shell commands via the `target` or `main_version` inputs.

(a) cpr-example-command.yml — step outputs are interpolated directly into echo commands:
  - `echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"`
  - `echo "Pull Request URL - ${{ steps.cpr.outputs.pull-request-url }}"`
  These values flow through YAML template substitution before the shell sees them, allowing metacharacter injection.

Locations:

- `.github/workflows/update-major-version.yml:22`
- `.github/workflows/update-major-version.yml:24`
- `.github/workflows/cpr-example-command.yml:36`
- `.github/workflows/cpr-example-command.yml:37`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN inherits the repository's default permissions (which may be broad), violating the principle of least privilege.

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

1. unpinned-uses: Pinned all 15 unpinned action references to full SHA hashes with tag comments:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f
   - actions/download-artifact@v7 → @37930b1c2abaa49bbe596cd826c3c89aef350131
   - peter-evans/close-pull@v3 → @a192af8d70f2d49c49643134605c3b73d4f80fae
   - peter-evans/find-comment@v4 → @b30e6a3c0ed37e7c023ccd3f1db5c6c0b0c23aad
   - peter-evans/create-or-update-comment@v5 → @e8674b075228eee787fea43ef493e45ece1004c9
   - peter-evans/create-pull-request@v8 → @5f6978faf089d4d20b00c7766989d076bb2fc7f1
   - peter-evans/enable-pull-request-automerge@v3 → @a660677d5469627102a1c1e11409dd063606628d
   - peter-evans/slash-command-dispatch@v5 → @9bdcd7914ec1b75590b790b844aa3b8eee7c683a

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in update-major-version.yml (git tag/push commands) and cpr-example-command.yml (echo commands).

3. missing-permissions: Added top-level permissions blocks to automerge-dependabot.yml (pull-requests: write, contents: write), cpr-example-command.yml (contents: write, pull-requests: write), slash-command-dispatch.yml (issues: write, pull-requests: write), and update-major-version.yml (contents: write).

