# Flux → Content Blocks migration (v13/v14)

When a project uses `fluidtypo3/flux` (usually with `fluidtypo3/vhs`) for content elements, factor this in as a dedicated cost block. As of mid-2026 neither library has a stable TYPO3 v14 release, but they differ on roadmap, so check both on Packagist at estimate time:

- **Flux** has active v14 development: `dev-development` and `dev-feature/v14` declare `^12 || ^13 || ^14.3` (as of 2026-06-18). A stable Flux v14 may therefore land, which would make a Flux fork-lead unnecessary.
- **VHS** has no v14 even on its dev branches (`dev-development` is still `^12 || ^13` as of 2026-06-30) and is the harder blocker.

Until a stable v14 of the library actually in use exists, an upgrade to v14 forces either a fork-lead of the missing library (VHS in particular) or a migration of the content to `friendsoftypo3/content-blocks` (+ `b13/container` for grid elements). The migration path below stays the safe planning assumption.

## Measure the effort-driving dimension

- Count the **populated** Flux content-element definitions, not the templates: `SELECT COUNT(DISTINCT CType) FROM tt_content WHERE CType LIKE '<ext>_%' AND deleted=0`. Definitions drive the modelling effort.
- Count records for the data-migration / QA scope: `SELECT COUNT(*) ... deleted=0` (include hidden — they migrate too).
- Split definitions into **field-based** vs **container** elements (discriminator below); they take different targets.

## CB native nesting vs EXT:container (the key distinction)

- Content Blocks native nesting is a `Collection` with a parent-reference field, and **`colPos` is always 0** (a single nested area). This reproduces single-area "wrapper" elements only.
- Multi-column grids, where editors drop arbitrary content elements into several named columns each with its own `colPos`, are the **EXT:container model** and are **not** reproducible by pure Content Blocks. Source: Content Blocks docs "Nested Content Elements" — "Extensions like EXT:container ... assign a specific colPos value to the child elements".
- **Discriminator in the Flux templates:** an element is a true multi-column container only if its template uses `contentContainer="true"` on a `flux:form.object` AND renders foreign content via `flux:content.render area="{...colPos}"` across multiple `flux:grid.column colPos=...`. A single fixed `colPos="1"` is single-area → CB native. No `contentContainer` / no `content.render` → field-based → CB.
- Architecture for a grid-heavy project is therefore **Content Blocks + b13/container**, not CB alone.

## Tooling: webcoast/migrator (verify maturity per project)

A modular toolchain exists (GPL-3.0, `typo3/cms-core: ^13.4`, runs on the **v13 intermediate stage** where both Flux and Content Blocks are installable):

- `webcoast/migrator` — core, DataHandler-based `RecordDataMigrator`
- `webcoast/migrator-from-flux` — provider; reads field config AND record data from `pi_flexform` (`FluxContentTypeProvider::getRecordData()`); requires `fluidtypo3/flux: ^11.1`
- `webcoast/migrator-to-content-blocks` — Content Blocks builder; requires `friendsoftypo3/content-blocks: ^1.3`
- `webcoast/migrator-to-container` — container builder; **requires `b13/container: ^3.2`**

How it routes: there is no automatic routing. The CB builder's `supports()` returns `true` for every type but has no grid handling (a grid migrated through it loses its column structure); only the container builder reproduces grids (`supports()` = `!empty($contentType->getGrid())`). The builder is chosen interactively per element.

**Maturity caveat — re-check at estimate time:** in mid-2026 all four packages are `dev-main`, no stable tag, no tests, dormant since early 2026. Treat the tool as effort-reducing for standard elements but verify coverage with a spike before lowering the estimate.

Documented tool limits: section fields only with one child type (else `UnsupportedContentTypeException`); `password` / `none` / `passthrough` / nested `flex` not supported.

## Dependency note: b13/container version line

- The migrator requires `b13/container: ^3.2` (which covers the v13 migration stage).
- The **v14 target needs `^4.0`** (`^13.4 || ^14.3`); `3.2.x` declares only `^14.0`, not `^14.3`, so the target install cannot stay on 3.2.

## Estimate impact

The tool removes most of the data migration and field-element modelling, but the following stay manual and belong in the delta: VHS ViewHelper replacement (no tool covers VHS — map to Core equivalents plus a custom rebuild of FAL / math / page helpers), the per-container `ContainerConfiguration` TCA + `ContainerProcessor` TypoScript + frontend template, and the multi-child sections. Keep the Flux-migration delta in the middle of its range until a verification spike on a v13 copy confirms the Flux provider covers the project's actual grid/section structures. The whole tool path is bound to the v13 window — if v13 is skipped, the content rebuild becomes fully manual (upper-end driver).
