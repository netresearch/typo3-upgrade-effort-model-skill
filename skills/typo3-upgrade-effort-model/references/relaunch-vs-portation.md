# Relaunch vs. portation of the theme layer (v13/v14)

When the frontend/theme is proprietary or built on libraries with no v14 path,
the estimate is NOT extension-count-driven. A cluster of theme-serving
extensions is *replaced by one sitepackage*, not migrated one by one. Estimating
each as a separate "port this extension" line double-counts and misreads the
shape of the work.

## The cluster that collapses

Under a v14 target these theme-layer packages typically have no forward path and
are absorbed by one new sitepackage build rather than ported. Verify each at
Packagist at estimate time — this table is a pattern, not a fact source.

| Extension (typical) | v14 status | Absorbed into |
|---|---|---|
| gridelements | none (max 12.0.0) | Content Blocks / b13 container |
| laxap bootstrap_grids | none, tied to gridelements | Content Blocks / container CEs |
| fluidtypo3/vhs | no stable v14 | Core ViewHelpers + custom rebuild |
| fluidtypo3/flux | check dev branch | Content Blocks |
| LESS compiler (ws_less …) | none | JS build (Vite) — see note |
| bootstrap_package | has v14 (16.x) | own sitepackage if relaunching |
| proprietary custom theme | n/a | the new sitepackage |

- Vite note: v14 removed core-side asset concatenation/compression (#108055).
  Projects that relied on it need an external build step (e.g. Vite) instead,
  which makes a LESS-compiler extension obsolete.
- bootstrap_package is path-dependent: "keep" under a pure portation, "drop"
  under a relaunch (Bootstrap pulled via npm and built). Do not classify it
  before the portation/relaunch decision is made.

## Decision rule: portation or relaunch

Prefer *relaunch* when any of these hold (each raises portation cost without
delivering v14 value):

- the theme depends on a library with no v14 path (vhs, gridelements, LESS
  compiler) — a port must replace it first, before any v14 benefit exists
- the load-bearing defects (SEO / performance / accessibility) are
  template-level, so a port carries them forward
- the theme is foreign or undocumented proprietary code (unknown quality)

Prefer *portation* when the theme is your own, clean, already on v14-viable
libraries, and free of the defects above.

On relaunch, a whole "port extension X" cluster leaves the estimate and is
replaced by two work-packages: *sitepackage build* (the systematic
underestimator) and *content/CE migration* (see
`flux-to-content-blocks-migration.md`).

## Measure the content-migration base, don't assume it

Migration effort scales with the *editorial page count*, not the crawl-URL
count. A crawler hitting its URL limit reflects dynamic module URLs (real-estate
details, news, faceted parameters), not pages an editor must re-lay-out.

- Editorial pages: count the live pages sitemap —
  `curl -s '[site]/sitemap.xml?sitemap=pages&cHash=…' | grep -c '<loc>'`, or
  `SELECT COUNT(*) FROM pages WHERE deleted=0 AND hidden=0 AND doktype IN (1,4)`.
- Exclude from the manual count: records rendered by an extension (EXT:news
  carries over via the database, template-only work) and data-driven detail
  pages (rendered by a module). Count those under their own module or QA line.
- Drop cruft the sitemap reveals (test / template / 404 pages left indexable) —
  a cleanup line, not a migration line.

## Estimate impact

- Estimate by work-package (assessment/spike, core upgrade, sitepackage
  relaunch, functional modules, SEO, content migration, QA, PM), not by summing
  a per-extension baseline across the extension count.
- Extension-count calibration anchors only the *core-upgrade* portion, never the
  relaunch.
- Best/worst hinges on the one thing unreadable without system access: the
  quality of the existing theme code. Keep the spread wide until an assessment
  with code access collapses it.
- Verify every extension's v14 status at Packagist
  (`repo.packagist.org/p2/<vendor>/<pkg>.json`, real `require.typo3/cms-core` per
  version). "Legacy" packages get updated (e.g. `friendsoftypo3/typo3db-legacy`
  1.3.0 supports v14); dead-looking ones may have only a dev branch. Never assume
  from memory.
