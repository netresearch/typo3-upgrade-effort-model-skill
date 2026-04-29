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

## License

Code: MIT. Documentation/content: CC-BY-SA-4.0. See `LICENSE-MIT` and `LICENSE-CC-BY-SA-4.0`.
