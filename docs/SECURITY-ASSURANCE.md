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

The repository contains no scripts and no executable program. There is no code path in this repository that parses untrusted input.

## Actors and trust boundaries

- **User**: asks for an estimate and chooses the TYPO3 project to assess. Trusted.
- **AI agent**: loads `SKILL.md` and the references and follows the 7-phase workflow. It acts with the user's permissions and the tools the user's agent platform grants.
- **Assessed project**: the user's TYPO3 project, including its `composer.json`, `composer.lock`, `vendor/`, `rector.php` and source code. The workflow reads it.
- **External services**: Packagist and the project's live site, queried by some workflow commands.
- **Maintainers and CI**: change and release this repository.

Boundary 1 lies between the skill text and the assessed project: the commands in the references run inside that project with the user's permissions. Boundary 2 lies between this repository and the user's machine: releases are built and signed in CI.

## Security requirements

1. The skill contains no executable code of its own; every command it asks the agent to run is visible in the reference text.
2. The workflow inspects the assessed project and does not change it.
3. The skill asks for, stores and transmits no credentials.
4. The skill and its releases are delivered unmodified from this repository.
5. Changes reach `main` only through the checks listed in [README.md](../README.md#governance-and-policies).

## Argument per requirement

### 1. No executable code of its own

`git ls-files` lists Markdown, JSON, YAML and licence files only; the Skill Validation job finds no shell script and no Python file to lint (`.github/workflows/lint.yml`). The commands the agent is told to run are fenced blocks in `references/assessment-workflow.md` (Phases 1, 3 and 4), `references/rector-coverage.md` ("How to detect Rector applicability") and one inline command in `references/relaunch-vs-portation.md` ("Measure the content-migration base").

### 2. The workflow inspects and does not change the project

- Phases 1 and 4 of `references/assessment-workflow.md` use `composer show`, `php --version`, `jq`, `ls`, `find` and `grep`. None of them writes to the project.
- Phase 3 uses `composer info --available` and reads version data from `repo.packagist.org`.
- `references/rector-coverage.md` runs `vendor/bin/rector process --dry-run`. `--dry-run` reports changes without writing them.
- `references/relaunch-vs-portation.md` counts pages with `curl` against the site's sitemap or a `SELECT COUNT(*)` query.
- `SKILL.md` and the references contain no `composer require`, `composer update`, `rm` or write-mode Rector command.

### 3. No credentials

`SKILL.md` and the references ask for no token, password or key, and none of the commands takes one. The skill has no storage of its own.

### 4. Delivered content is the reviewed content

- Releases are built by `.github/workflows/release.yml`, which calls the `netresearch/skill-repo-skill` release workflow with `id-token: write` and `attestations: write`. That workflow signs `SHA256SUMS.txt` keyless with `cosign sign-blob` and attests the release archives and checksums with `actions/attest-build-provenance`.
- The Skill Validation job checks that `plugin.json` and `.claude-plugin/plugin.json` agree and that the `SKILL.md` version matches the plugin version.

### 5. Changes pass automated checks

`lint.yml` and `eval-validate.yml` grant `contents: read` only. `auto-merge-deps.yml` runs on `pull_request_target` and calls the shared workflow in `netresearch/.github`, which contains no checkout step and runs no pull request code; this repository passes it no secrets. The checks themselves are listed in [README.md](../README.md#governance-and-policies).

## Common weaknesses

| Weakness | Where it could arise | Countermeasure |
|----------|---------------------|----------------|
| CWE-78 OS command injection | Shell commands in the references | The placeholders (`<vendor>`, `[site]`) are filled in by the user or agent for their own project. The one loop that reuses project data (Phase 3 in `references/assessment-workflow.md`) passes each Composer package name to `composer info` as a quoted argument; no command string is evaluated. |
| CWE-94 code injection | Project code executed during assessment | Only `vendor/bin/rector process --dry-run` executes project code (see limits below); everything else reads files or queries package metadata. |
| CWE-798 credential exposure | Commits to this repository | GitHub secret scanning with push protection is enabled for the repository. No file reads or stores credentials. |
| CWE-829 inclusion of functionality from an untrusted source | CI workflows | The workflows call shared workflows inside the `netresearch` organisation; those pin third-party actions by commit SHA. |
| CWE-1104 unmaintained third-party components | Composer dependency | The only dependency is `netresearch/composer-agent-skill-plugin` (`composer.json`); Renovate opens update pull requests (`renovate.json`). |

## What the skill does not protect against

- **An estimate is not a security review.** The model produces effort ranges; it does not assess whether the project or its extensions are secure.
- **Commands run in the assessed project.** They run with the user's permissions. `vendor/bin/rector process --dry-run` executes the project's installed Rector and its `rector.php`, and Composer commands may load the project's installed Composer plugins. Assess only projects whose `vendor/` and configuration you trust, or run the workflow in an isolated environment.
- **Network access.** Phase 3 and the Packagist note contact `repo.packagist.org`; the page count may contact the project's live site.
- **Instructions inside project files.** The agent reads the project's files. The skill does not defend against text in those files that tries to steer the agent; that is the agent platform's responsibility.
- **`allowed-tools`.** `SKILL.md` declares none. Where a skill declares `allowed-tools`, it only pre-approves tools; it does not remove tools the agent already has.
