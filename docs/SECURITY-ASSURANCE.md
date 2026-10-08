<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->

# Security Assurance Case

This document states what users of the typo3-upgrade-effort-model skill can and cannot expect in terms of security, and argues why the expectations hold. Every claim names the file that supports it. Vulnerabilities are reported privately as described in the [organisation security policy](https://github.com/netresearch/.github/blob/main/SECURITY.md).

## What the project ships

| Part | Files | Runs code? |
|------|-------|------------|
| Skill instructions | `skills/typo3-upgrade-effort-model/SKILL.md` | No. Text an AI agent loads. |
| References | `skills/typo3-upgrade-effort-model/references/*.md` | No. Text an AI agent loads. Some contain shell commands that the agent or the user runs in the project being assessed (see below). |
| Evaluation cases | `evals/evals.json` | No. Prompts and expected answer patterns, validated in CI; not loaded by the skill. |
| Manifests | `plugin.json`, `.claude-plugin/plugin.json`, `composer.json` | No. Package metadata. |

The skill contains no scripts and no executable program, so a user of the skill runs no code from this repository. Two places parse untrusted input. In CI, on every pull request the workflows here call shared workflows of `netresearch/skill-repo-skill`, which check out the pull request and run linters and validators over its Markdown, YAML and JSON files with `contents: read`, and `check-template-drift.yml` of `netresearch/.github` checks it out with `contents: read` and compares its `.github/` files with the organisation template; on pull requests into `main`, `security.yml` also checks out the pull request and runs Betterleaks, zizmor and Opengrep over it with `contents: read` and `security-events: write`, and the Composer Audit job runs `composer install` with all Composer plugins allowed and then `composer audit`, with `contents: read` (see requirement 5). On the user's machine, the commands the references ask for parse data of the assessed project with the user's tools (`jq`, `grep`, Composer) and with the project's own installed Rector; none of that is code shipped here (see requirement 2 and the limits below).

## Actors and trust boundaries

- **User**: asks for an estimate and chooses the TYPO3 project to assess. Trusted.
- **AI agent**: loads `SKILL.md` and the references and follows the 7-phase workflow. It acts with the user's permissions and the tools the user's agent platform grants.
- **Assessed project**: the user's TYPO3 project, including its `composer.json`, `composer.lock`, `vendor/`, `rector.php` and source code. The workflow reads it.
- **External services**: Packagist and the project's live site, queried by some workflow commands.
- **Maintainers and CI**: change and release this repository.

Boundary 1 lies between the skill text and the assessed project: the commands in the references run inside that project with the user's permissions. Boundary 2 lies between this repository and the user's machine: releases are built and signed in CI.

## Security requirements

1. The skill contains no executable code of its own; every command it asks the agent to run is visible in the reference text.
2. The workflow asks for no command that writes to the assessed project. Some of its commands execute code the project controls, with the user's permissions (see [limits](#what-the-skill-does-not-protect-against)).
3. The skill asks for, stores and transmits no credentials.
4. The skill and its releases are delivered unmodified from this repository.
5. A pull request into `main` merges only with signed commits and with the checks that branch protection requires passing; repository admins can bypass this.

## Argument per requirement

### 1. No executable code of its own

`git ls-files` lists Markdown, JSON, JSONC, YAML and licence files and `.gitignore`; the Skill Validation job finds no shell script and no Python file to lint (`.github/workflows/lint.yml`). The commands the agent is told to run are fenced blocks in `references/assessment-workflow.md` (Phases 1, 3 and 4) and `references/rector-coverage.md` ("How to detect Rector applicability"), two inline commands in `references/relaunch-vs-portation.md` ("Measure the content-migration base"), three SQL queries and one `grep` inline in `references/flux-to-content-blocks-migration.md`, and `grep` searches named without a full command line in the Detection column of `references/risk-multipliers.md` and in `references/upgrade-patterns.md` ("grep changelogs for this marker").

### 2. The workflow asks for no write to the project

- Phases 1 and 4 of `references/assessment-workflow.md` use `composer show`, `php --version`, `jq`, `ls`, `find` and `grep`. None of them writes to the project.
- Phase 3 uses `composer info --available` and reads version data from `repo.packagist.org`.
- `references/rector-coverage.md` runs `vendor/bin/rector process --dry-run`. `--dry-run` reports changes without writing them.
- `references/relaunch-vs-portation.md` counts pages with `curl` against the site's sitemap or a `SELECT COUNT(*)` query.
- `references/flux-to-content-blocks-migration.md` counts content elements with `SELECT` queries and searches configuration exports with `grep`.
- `references/risk-multipliers.md` and `references/upgrade-patterns.md` detect affected code and changelog entries with `grep`.
- `SKILL.md` and the references contain no `composer require`, `composer update`, `rm` or write-mode Rector command.
- Read-only does not mean no project code runs: `vendor/bin/rector process --dry-run` loads the project's `rector.php` and installed Rector, and Composer commands may load the project's installed Composer plugins. That code runs with the user's permissions and can have side effects the skill does not control.

### 3. No credentials

`SKILL.md` and the references ask for no token, password or key, and none of the commands takes one. The skill has no storage of its own.

### 4. Delivered content is the reviewed content

- Releases are built by `.github/workflows/release.yml`, which calls the `netresearch/skill-repo-skill` release workflow with `id-token: write` and `attestations: write`. That workflow signs `SHA256SUMS.txt` keyless with `cosign sign-blob` and attests the release archives and checksums with `actions/attest-build-provenance`.
- The Skill Validation job checks that `plugin.json` and `.claude-plugin/plugin.json` agree and that the `SKILL.md` version matches the plugin version.

### 5. Pull requests pass the required checks

Branch protection on `main` (a repository setting) requires a pull request, signed commits, and passing Skill Validation, Eval Validation, Secret Scanning, Composer Audit, SAST (Opengrep), dependency review, CodeQL `Analyze (actions)` and DCO checks on a branch that is up to date with `main`. It is not enforced for repository admins, so an admin can merge without them.

`lint.yml` sets `contents: read` for the workflow; `eval-validate.yml`, `harness-verify.yml` and `check-template-drift.yml` set `permissions: {}` and give their job `contents: read`. `security.yml` sets `permissions: {}` at the top and gives each of its four jobs `contents: read` plus `security-events: write` (to upload results to code scanning) or, for dependency review, `pull-requests: write` (to comment on the pull request). `auto-merge-deps.yml` runs on `pull_request_target` and calls the shared workflow in `netresearch/.github`, which contains no checkout step and runs no pull request code; this repository passes it two secrets, `PROJECT_APP_ID` and `PROJECT_APP_PRIVATE_KEY`, which the shared workflow uses to mint a GitHub App token for the merge, and the shared workflow skips authors other than Renovate and Dependabot. `labeler.yml` also runs on `pull_request_target`, with `contents: read` and `pull-requests: write`, and calls a shared workflow in `netresearch/.github` that only applies labels and checks out nothing. `scorecard.yml` is not run on pull requests. The checks themselves are listed in [README.md](../README.md#governance-and-policies).

## Common weaknesses

| Weakness | Where it could arise | Countermeasure |
|----------|---------------------|----------------|
| CWE-78 OS command injection | Shell commands in the references | The placeholders (`<vendor>`, `<pkg>`, `<ext>`, `[site]`, `[config-exports]`) are filled in by the user or agent for their own project. The one loop that reuses project data (Phase 3 in `references/assessment-workflow.md`) passes each Composer package name to `composer info` as a quoted argument; no command string is evaluated. |
| CWE-94 code injection | Project code executed during assessment | `vendor/bin/rector process --dry-run` executes the project's Rector configuration, and Composer commands may load the project's installed Composer plugins; the other commands read files, query a database, query package metadata or fetch the project's live sitemap. The skill cannot contain what that project code does; the limits section of this document advises assessing only projects whose `vendor/` and configuration you trust, or using an isolated environment. The skill text itself carries no such warning. |
| CWE-798 credential exposure | Commits to this repository | GitHub secret scanning with push protection is enabled for the repository, and Betterleaks scans pull requests into `main` and pushes to `main` (`security.yml`). No file reads or stores credentials. |
| CWE-829 inclusion of functionality from an untrusted source | CI workflows | The workflows call shared workflows inside the `netresearch` organisation; those pin third-party actions by commit SHA. |
| CWE-1104 unmaintained third-party components | Composer dependency | The only dependency is `netresearch/composer-agent-skill-plugin`, required as `^2.0` in `composer.json` (the current major line), so a new major needs a change here. Dependency review and Composer Audit (`security.yml`) check dependency changes and known advisories on pull requests into `main`. Renovate opens update pull requests for the pre-commit hooks pinned in `.pre-commit-config.yaml` (`renovate.json`). |

## What the skill does not protect against

- **An estimate is not a security review.** The model produces effort ranges; it does not assess whether the project or its extensions are secure.
- **Commands run in the assessed project.** They run with the user's permissions. `vendor/bin/rector process --dry-run` executes the project's installed Rector and its `rector.php`, and Composer commands may load the project's installed Composer plugins. Assess only projects whose `vendor/` and configuration you trust, or run the workflow in an isolated environment.
- **Network access.** Phase 3 and the Packagist note contact `repo.packagist.org`; the page count may contact the project's live site.
- **Instructions inside project files.** The agent reads the project's files. The skill does not defend against text in those files that tries to steer the agent; that is the agent platform's responsibility.
- **`allowed-tools`.** `SKILL.md` declares none. Where a skill declares `allowed-tools`, it only pre-approves tools; it does not remove tools the agent already has.
