<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->

# typo3-upgrade-effort-model-skill

Agent skill with a generic effort model for TYPO3 LTS major version upgrades: risk multipliers, baselines, a version-compatibility matrix, Rector coverage adjustments and a 7-phase assessment workflow. The repository is text only: Markdown, JSON and YAML, no scripts.

## Repo Structure

```
skills/typo3-upgrade-effort-model/
  SKILL.md                  skill definition and trigger description
  references/               multipliers, baselines, compatibility matrix, workflow, Rector coverage
evals/evals.json            eval cases that assert the answers the skill gives
docs/SECURITY-ASSURANCE.md  threat model, trust boundaries, countermeasures
plugin.json                 Agent Plugins manifest
.claude-plugin/plugin.json  Claude Code manifest; shared fields must match plugin.json
composer.json               Composer distribution metadata
.pre-commit-config.yaml     local hooks, the same linters as CI
.github/workflows/          CI callers of reusable workflows
.github/template.yaml       skill template this repo follows, with intentional drift
.github/labeler.yml         path-to-label rules for the Labeler workflow
```

## Commands

- Hooks and linters: `pre-commit install --install-hooks`, then `pre-commit run --all-files`
- Manifest and eval checks, with a checkout of `netresearch/skill-repo-skill` at `<tools>`, from the repository root: `bash <tools>/skills/skill-repo/scripts/sync-plugin-manifest.sh --check` and `bash <tools>/skills/skill-repo/scripts/validate-evals.sh evals/evals.json`

There is no build and no test suite; see [README.md](README.md) for the checks that run on pull requests.

## Rules

- Licensing is split: code, configuration and workflows are MIT ([LICENSE-MIT](LICENSE-MIT)); documentation and skill content are CC-BY-SA-4.0 ([LICENSE-CC-BY-SA-4.0](LICENSE-CC-BY-SA-4.0)). `composer.json` declares `(MIT AND CC-BY-SA-4.0)`.
- Keep `plugin.json` and `.claude-plugin/plugin.json` in step, and the `SKILL.md` version equal to the plugin version; Skill Validation (`.github/workflows/lint.yml`) fails otherwise.
- A change to `SKILL.md` or a reference that changes an answer the skill gives comes with a case in `evals/evals.json` that asserts the new answer ([README.md](README.md), Contributing).
- The workflow files that also exist in the `skill` template of `netresearch/.github` are governed by it; `.github/template.yaml` lists the intentional exceptions (`lint.yml`, `release.yml`), and Template Drift fails when any other governed file differs.
- Reusable workflows are called by `@main` from `netresearch/.github`, `netresearch/skill-repo-skill` and `netresearch/typo3-ci-workflows`.
- Every commit needs a `Signed-off-by` trailer (`git commit -s`) and a signature; branch protection on `main` requires signed commits and the DCO check.
- The skill ships no executable code and asks for no write to the assessed project; see [docs/SECURITY-ASSURANCE.md](docs/SECURITY-ASSURANCE.md) before adding either.

## References

- [README.md](README.md): installation, usage, checks that run on pull requests, dependencies
- [skills/typo3-upgrade-effort-model/SKILL.md](skills/typo3-upgrade-effort-model/SKILL.md): skill content
