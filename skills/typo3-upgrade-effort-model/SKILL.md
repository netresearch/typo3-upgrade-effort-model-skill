---
name: typo3-upgrade-effort-model
description: "Use when estimating effort for TYPO3 LTS major version upgrades (current target: v14.3 LTS, released 2026-04-21 — 98 breaking + 31 deprecation entries). Provides risk multipliers (Fluid 5 strict VHs #108148, HashService removal, TSFE removal #107831, core asset-concat removal #108055, magic Extbase finders, EXT:form hooks), per-extension baselines, version-compatibility matrix, Rector coverage adjustments, and a 7-phase assessment workflow. Calibration-free — pair with your own historical project data for tuned estimates."
metadata:
  version: "1.0.0"
---

# TYPO3 Upgrade Effort Model

Generic effort estimation model for TYPO3 LTS major version upgrades. The model captures *what* makes an upgrade hard (per-version breaking-change deltas, Rector coverage, ecosystem readiness); your team's historical data captures *how fast* you handle each unit.

> Calibration is intentionally external. The model gives you per-extension hour ranges; you (or a follow-up internal skill) supply the calibration multiplier from your own historical projects.

## When to Use This Skill

Trigger this skill when:

- Estimating TYPO3 upgrades: *"Estimate effort to upgrade TYPO3 11.5 to 14.3"*, *"v13 → v14.3 LTS effort?"*
- Analyzing upgrade feasibility: *"What's the risk level for upgrading to TYPO3 14.3?"*
- Planning version migrations: *"Should we do skip-version or incremental?"*
- Assessing extension compatibility: *"Which extensions need updates for TYPO3 14?"*

## v14.3 LTS Headline (released 2026-04-21)

v14 is the largest breaking-change cycle in several years — **98 breaking + 31 deprecation entries**, all landed in v14.0. Sprint releases 14.0–14.2 are unsupported as of the 14.3 release.

### Risk multipliers when target is v14

| Factor | Multiplier | Why |
|--------|-----------|-----|
| Fluid 5 strict VH typing ([#108148](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.0/Breaking-108148-Fluid50.html)) | × 1.2 | Every custom VH needs typed args + `render(): string`; LenientArgumentProcessor deprecated |
| `Extbase HashService` removal (#105377 umbrella) | × 1.1 | Trigger-words grep finds callers quickly, but each needs replacement + HMAC rotation |
| `TypoScriptFrontendController` class removal ([#107831](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.0/Breaking-107831-RemovedTypoScriptFrontendController.html)) | × 1.3 | Sites heavily using `$GLOBALS['TSFE']` need refactoring to `$request` attributes |
| Magic Extbase repo finders (`findByX`) removal | × 1.1 | Usually mechanical via Rector |
| Core asset concat/compression removal ([#108055](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.0/Breaking-108055-RemovedFrontendAssetConcatenationAndCompression.html)) | × 1.2 | If no build tool yet, Vite/webpack integration work needed |
| TCA sweep (list_type, searchFields, is_static, eval=year) | × 1.1 | Per-extension breaking change discovery |
| EXT:form 10 hook removals | × 1.05–1.2 | Only if project uses form framework |
| `ext_tables.php` deprecation ([#109438](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.3/Deprecation-109438-ExtTablesPhpInExtensions.html)) | × 1.05 | Split into `Configuration/Backend/Modules.php`, `Routes.php`, TCA overrides |

**Rector coverage boost:** 47 v14-specific Rector rules (46 under `TYPO314/v0/`, 1 in `v2/`). Auto-fix ratio significantly higher than v12→v13. Reduce hand-coded migration estimates by ~15–20% vs naïve breaking-count math.

**Post-upgrade ops overhead:** +0.5–1 h per project for Important [#109585](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/14.3/Important-109585-SerializedCredentialDataInBeUsersDatabaseTable.html) serialized-credentials wizard (only if project transited through v14.2).

## Baseline Estimates

| Path | Per own extension (with Rector) | Per third-party w/o v14 release | Per project ops overhead |
|------|---------------------------------|----------------------------------|--------------------------|
| v13 → v14.3 | 4–8 h | +2–4 h | +0.5–1 h |
| v12 → v13.4 | 3–6 h | +2–3 h | +0.5 h |
| v11 → v12.4 | 4–7 h | +3–5 h | +1 h |
| v10 → v11.5 | 5–8 h | +4–6 h | +1 h |

These are **before calibration**. Multiply by your team's calibration factor (typically 0.6–1.4) when applying to a real project.

## Extension Classification

Generic types (apply to any TYPO3 project regardless of organisation):

| Type | Pattern | Risk | Estimation |
|------|---------|------|------------|
| TYPO3 Core | `typo3/cms-*` | LOW | 0 h (handled by core) |
| Public third-party (active) | TER / Packagist with v14 release | LOW | 0–4 h (composer bump + smoke test) |
| Public third-party (stale) | TER / Packagist without v14 release | MEDIUM | 4–12 h (fork + patch or replace) |
| Customer-specific custom | Owned by you, project-scoped | MEDIUM | 8–24 h |
| Heavily customised | Custom + complex business logic | HIGH | 24–60 h |

Add organisation-specific patterns (e.g. internal vendor extensions) as a thin overlay in your calibration skill.

## 7-Phase Assessment Workflow

The model drives this generic workflow:

1. **Project Discovery** — current/target TYPO3 version, PHP version, extension inventory, build tooling.
2. **Extension Classification** — categorise each by ownership and maintenance status (above table).
3. **Compatibility Matrix** — check each extension against target TYPO3/PHP versions; flag gaps.
4. **Custom Code Analysis** — grep for breaking-change trigger words (`HashService`, `$GLOBALS['TSFE']`, `findBy*`, untyped Fluid VHs, raw `config.concatenateCss`, `ext_tables.php`).
5. **Risk Level Calculation** — compound from per-multiplier hits above.
6. **Work Estimation** — apply baselines × multipliers × calibration factor.
7. **Timeline Recommendation** — phased approach with validation gates (lint → test → smoke).

See `references/assessment-workflow.md` for command-level detail.

## Companion Skills

- `typo3-conformance-skill` — extension-quality scoring; conformance score can feed risk multiplier.
- `typo3-testing-skill` — test-coverage assessment; coverage influences regression risk.
- `typo3-extension-upgrade-skill` — actually performs the upgrade after the estimate.
- `typo3-project-upgrade-skill` — deployed-project upgrade workflow.

For Netresearch-internal estimations with team-specific calibration constants and historical project data, see `coding-ai/typo3-upgrade-estimator-skill` (which consumes this model).

## References

- `references/version-compatibility.md` — TYPO3 ↔ PHP support matrix
- `references/risk-multipliers.md` — full table with rationale per factor
- `references/extension-classification.md` — generic categories (no organisation patterns)
- `references/assessment-workflow.md` — 7-phase command-level workflow
- `references/rector-coverage.md` — per-version Rector rule counts and auto-fix ratios
