<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: Netresearch DTT GmbH -->
# typo3-upgrade-effort-model-skill

Generic effort model for TYPO3 LTS major version upgrades. Provides per-version risk multipliers, breaking-change baselines, version-compatibility matrix, Rector coverage adjustments, and a 7-phase assessment workflow. Calibration-free — pair with your team's historical project data for tuned estimates.

## 🔌 Compatibility

Agent Skill following the [open standard](https://agentskills.io). Works with Claude Code, Cursor, GitHub Copilot, and any skills-compatible AI agent.

## What this skill does

- v14 risk-multiplier table (Fluid 5 strict VHs, HashService removal, TSFE removal, asset-pipeline removal, magic finders, EXT:form hooks, …)
- per-extension baselines for v10→v14 paths
- TYPO3 ↔ PHP version-compatibility matrix
- generic extension classification (core / public-active / public-stale / customer-custom / heavily-customised)
- 7-phase assessment workflow with command-level detail
- Rector coverage discount per target version

## What this skill does NOT do

- It doesn't supply your team's calibration factor — you measure that against your historical projects.
- It doesn't ship case studies for specific projects (those belong in your private estimator skill).
- It doesn't set deadlines or pricing — that's commercial, not technical.

## Installation

### Via Claude Code Marketplace

```
/plugin install typo3-upgrade-effort-model@netresearch-claude-code-marketplace
```

### Via Composer

```bash
composer require netresearch/typo3-upgrade-effort-model-skill
```

## Usage

Trigger on prompts like:

- *"Estimate effort to upgrade TYPO3 11.5 to 14.3 LTS"*
- *"What's the risk level for upgrading our extension to v14?"*
- *"Should we skip-version v11 → v13 or go incremental?"*

The skill walks the 7-phase workflow and produces an effort range per extension and per project. Apply your team's calibration factor on top.

## Companion skills

- `typo3-conformance-skill` — extension quality scoring (feeds risk multiplier)
- `typo3-testing-skill` — test coverage assessment (feeds regression risk)
- `typo3-extension-upgrade-skill` — performs the upgrade after the estimate
- `typo3-project-upgrade-skill` — deployed-project upgrade workflow

For Netresearch-internal use with historical-data calibration, see `coding-ai/typo3-upgrade-estimator-skill` (which consumes this model).

## Contributing

Issues and pull requests are welcome. Every commit needs a `Signed-off-by` trailer (`git commit -s`); the DCO check enforces it.

A change to `SKILL.md` or a reference that changes an answer the skill gives (a multiplier, a baseline, a workflow step) comes with a case in `evals/evals.json` that asserts the new answer.

### Checks

The repository ships no executable code, so it has no behavioural tests. The checks validate the skill's structure, its evaluation cases and the syntax of every file:

- **Skill Validation** (`.github/workflows/lint.yml`, calling `validate.yml` of `netresearch/skill-repo-skill`): skill structure and front matter (`validate-skill.sh`), manifest sync between `plugin.json` and `.claude-plugin/plugin.json`, `SKILL.md` version against the plugin version, markdownlint, yamllint, actionlint, JSON syntax, ShellCheck at severity `style`, ruff check and format, checkpoint schema. Steps that find no file of their kind (shell, Python, checkpoints) say so and pass.
- **Eval Validation** (`.github/workflows/eval-validate.yml`): checks the structure of `evals/evals.json` with `validate-evals.sh`. It does not run the prompts against a model.

Both run on every pull request and on every push to `main`. To run the same checks locally:

```bash
pre-commit install --install-hooks   # once per clone
pre-commit run --all-files           # exit 0 = all hooks passed
```

The hooks in `.pre-commit-config.yaml` run the same linters, the skill validator and the version-parity check; they use `netresearch/skill-repo-skill` at the pinned `rev:`, while CI uses its `main`. The manifest-sync and eval checks have no hook; with a checkout of `netresearch/skill-repo-skill` at `<tools>`, run them from the root of this repository:

```bash
bash <tools>/skills/skill-repo/scripts/sync-plugin-manifest.sh --check
bash <tools>/skills/skill-repo/scripts/validate-evals.sh evals/evals.json
```

A failure names the file and the rule: an `MD…` rule for markdownlint, a yamllint rule name, an `ERROR:` line from `validate-skill.sh`, a `FAIL:` line from `validate-evals.sh`. `WARNING:` lines do not fail the check.

## Dependencies

- **Runtime:** none. The skill is text. The commands in the references use tools of the assessed project (Composer, PHP, `jq`, Rector), which the user provides.
- **Composer:** `composer.json` requires `netresearch/composer-agent-skill-plugin`, which installs the skill into a Composer project. There is no lock file; the package is consumed as a library.
- **CI:** the workflows call shared workflows in `netresearch/skill-repo-skill` and `netresearch/.github` by `@main`; those pin third-party actions by commit SHA and tools by version. The pre-commit hooks are pinned by `rev:` in `.pre-commit-config.yaml`.
- **Updates:** Renovate (`renovate.json`, extending the organisation preset `local>netresearch/renovate-config`, with the `pre-commit` manager enabled) opens update pull requests. `auto-merge-deps.yml` passes pull requests from Renovate and Dependabot to the shared auto-merge workflow in `netresearch/.github`.
- **Selection:** a new dependency is added only when the skill or its checks cannot work without it, and is declared where its consumer reads it (`composer.json` for Composer, `.pre-commit-config.yaml` for hooks).

## Governance and policies

This repository follows the Netresearch organisation policies:

- [Governance](https://github.com/netresearch/.github/blob/main/GOVERNANCE.md): ownership, roles, and how decisions are made and disputes resolved.
- [Roadmap](https://github.com/netresearch/.github/blob/main/ROADMAP.md): planned and explicitly excluded work for the coming year.
- [Handling of dependency and code analysis findings](https://github.com/netresearch/.github/blob/main/SECURITY.md#handling-of-dependency-and-code-analysis-findings): thresholds, deadlines and the exception process for dependency (SCA) and static analysis (SAST) findings.
- [Secret management](https://github.com/netresearch/.github/blob/main/SECURITY.md#secret-management): how CI and release credentials are stored, accessed and rotated.
- [Access roster](https://github.com/netresearch/.github/blob/main/docs/access-roster.md): who holds administrative and write access to this repository and the organisation.

The security assurance case for this skill (threat model, trust boundaries, countermeasures and limits) is in [`docs/SECURITY-ASSURANCE.md`](docs/SECURITY-ASSURANCE.md).

Checks that run on pull requests in this repository:

- Skill Validation (`lint.yml`) and Eval Validation (`eval-validate.yml`), described under [Checks](#checks).
- DCO: every commit carries a `Signed-off-by` trailer.
- CodeQL default setup (a repository setting, not a workflow file) analyses the GitHub Actions workflows with the extended query suite.
- CodeRabbit (a GitHub App configured for the organisation, not a workflow file) reviews pull requests and reports a `CodeRabbit` status; it is not a required check.
- Branch protection on `main` requires Skill Validation, CodeQL `Analyze (actions)` and DCO to pass before a merge; it is not enforced for repository admins.
- No workflow here runs dependency review, a dependency audit, Opengrep or Betterleaks. Secret detection is GitHub secret scanning with push protection, which is enabled for this repository.

## License

Code: MIT. Documentation/content: CC-BY-SA-4.0. See `LICENSE-MIT` and `LICENSE-CC-BY-SA-4.0`.
