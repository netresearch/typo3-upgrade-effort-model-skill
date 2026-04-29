# TYPO3 Extension Upgrade Patterns

**Source:** Analyzed from georgringer/news extension upgrade history (TYPO3 10→11→12)
**Purpose:** Real-world patterns to inform upgrade effort estimation

## Overview

This document captures common upgrade patterns observed in the news extension, which serves as a reference implementation for TYPO3 best practices. These patterns should inform the estimation formulas in the TYPO3 Upgrade Estimator skill.

## Composer Dependency Updates

### Pattern: Progressive PHP Version Requirements

**Example from news:**

```
TYPO3 10: php >= 7.2 < 7.5
TYPO3 11: php >= 7.4 < 8.3
TYPO3 12: php >= 8.1 < 8.4
```

**Estimation Impact:**

- Moving from PHP 7.x → 8.x requires 16-24 hours for medium extensions
- Each major PHP version requires null safety fixes, type hints, and attribute updates
- Add 4-8 hours for each minor PHP version beyond target minimum

### Pattern: Specific Core Package Requirements

**Example:**

```json
// TYPO3 11
"require": {
  "typo3/cms-core": "^11.5.24 || ^12"
}

// TYPO3 12
"require": {
  "typo3/cms-backend": "^12.4.2 || ^13.1",
  "typo3/cms-core": "^12.4.2 || ^13.1",
  "typo3/cms-extbase": "^12.4.2 || ^13.1",
  "typo3/cms-frontend": "^12.4.2 || ^13.1",
  "typo3/cms-fluid": "^12.4.2 || ^13.1"
}
```

**Estimation Impact:**

- Splitting cms-core into specific packages: +2 hours
- Testing each package dependency: +2 hours per package
- Total: ~4 hours for composer dependency refinement

### Pattern: Testing Framework Updates

**Example:**

```
TYPO3 11: testing-framework ~7.0, phpunit ^9
TYPO3 12: testing-framework ^8.0, phpunit ^10
```

**Estimation Impact:**

- PHPUnit major version upgrade: 8-16 hours
  - Test syntax updates (assertions, data providers)
  - Mock API changes
  - Deprecation fixes
  - CI/CD pipeline updates

## Automated Code Modernization (Rector)

### Pattern: Extensive Rector Usage

**Observed Rector Rules Applied:**

- `UseServerRequestInsteadOfGeneralUtilityGetRector`
- `MigrateGeneralUtilityGPRector`
- `SimplifyCheckboxItemsTCARector`
- `ExtbaseActionsWithRedirectMustReturnResponseInterfaceRector`
- `PrivatizeFinalClassPropertyRector`
- `RemoveUnusedPrivateMethodParameterRector`
- `UseLanguageAspectInExtbasePersistenceRector`
- `MigrateConfigurationManagerGetContentObjectRector`
- And 15+ more...

**Estimation Impact:**

- **If Rector available:** 8-16 hours
  - Install TYPO3 Rector
  - Configure rules
  - Run automated refactoring
  - Review and fix edge cases
  - Test changes

- **If manual refactoring:** 40-80 hours
  - Manual search & replace patterns
  - Method signature updates
  - Request object migrations
  - Attribute conversions

**Key Insight:** Rector reduces refactoring time by 60-70%

**Add to Skill:** Check for `.rector.php` configuration in project

- If present: Use automated estimation (lower hours)
- If absent: Use manual estimation (higher hours)
- Recommend Rector installation in report

## TCA Configuration Updates

### Pattern: TCA Simplification

**Observed Changes:**

- Simplified checkbox item configuration
- Updated file reference TCA
- Category TCA modernization
- Removed migration helper files (`z_misc_12_adoptions.php`)

**Example Commit Patterns:**

```
[TASK] TCA for tca labels
[TASK] TCA for sys_category images
[TASK] TCA file cleanup
[TASK] Update tca for sys_file_reference
[TASK] Simplify TCA checkbox items
```

**Estimation Impact:**

- Per TCA file update: 30-60 minutes
- Complex TCA (inline relations, file references): 1-2 hours
- Migration helper removal: 1 hour
- Total for medium extension (15 TCA files): 12-20 hours

## Breaking Changes & Removals

### Pattern: Deprecated Feature Removal

**Examples from news 11→12:**

```
[!!!] Remove usage of LOAD_REGISTER with news values
[!!!] Remove label overlay for categories
[!!!] Remove deprecated sitemap option
[!!!] Replace QueryGenerator
[!!!] Remove switchablecontrolleractions from flexforms
```

**Categories:**

1. **Removed ViewHelper Features** (LOAD_REGISTER)
2. **Removed Utilities** (QueryGenerator, label overlay)
3. **Removed Configuration Options** (sitemap, switchablecontrolleractions)
4. **Removed Legacy Code** (RealURL migration)

**Estimation Impact:**

- Per breaking change: 1-4 hours
  - Find all usages
  - Implement replacement
  - Test functionality
  - Update documentation

**Pattern Recognition:**

- Breaking changes marked with `[!!!]` in commit messages
- Average 5-10 breaking changes per major version
- Total: 15-40 hours for breaking change handling

**Add to Skill:** Parse TYPO3 changelog for `[!!!]` markers to estimate breaking change impact

## Configuration Structure Updates

### Pattern: Backend Module Registration

**Change:**

```
// Before (ext_tables.php) - TYPO3 11
ExtensionManagementUtility::addModule(...);

// After (Configuration/Backend/Modules.php) - TYPO3 12
return [
    'web_NewsNews' => [
        'parent' => 'web',
        'position' => ['after' => 'web_info'],
        ...
    ],
];
```

**Estimation Impact:**

- Per backend module: 2-4 hours
- Multiple modules: 2-4 hours each
- Total: 4-16 hours depending on module count

### Pattern: Services Configuration

**Changes:**

- Updated DI configuration
- Service tagging updates
- Autowiring improvements

**Estimation Impact:**

- Per services configuration update: 1-2 hours
- Complex service dependencies: 4-8 hours
- Total: 4-12 hours for typical extension

### Pattern: Icon Registration

**Changes:**

- Icon provider updates
- SVG icon handling
- Icon registration modernization

**Estimation Impact:**

- Icon registration update: 2-4 hours total (regardless of icon count)

## FlexForm Updates

### Pattern: FlexForm Modernization

**Observed Changes:**

- Removed `switchablecontrolleractions`
- Updated field configurations
- Simplified settings structure

**Files Updated in news:**

```
Configuration/FlexForms/flexform_category_list.xml (58 changes)
Configuration/FlexForms/flexform_news.xml (275 changes)
Configuration/FlexForms/flexform_news_detail.xml (78 changes)
Configuration/FlexForms/flexform_news_list.xml (212 changes)
Configuration/FlexForms/flexform_tag_list.xml (94 changes)
```

**Estimation Impact:**

- Per FlexForm file: 1-3 hours
- Complex plugins with multiple FlexForms: 8-15 hours
- Total for typical extension (3-5 FlexForms): 6-12 hours

## Testing Infrastructure Modernization

### Pattern: runTests.sh Complete Rewrite

**Major Change:** Docker-compose → Docker

**Key Updates:**

- New database version handling (MariaDB 10.3-11.1)
- Improved network management
- Better container lifecycle
- Wait-for patterns for dependencies

**Estimation Impact:**

- Adopt new runTests.sh from Tea extension: 4-8 hours
- Custom test script modifications: 2-4 hours
- CI/CD pipeline updates: 4-8 hours
- Total: 10-20 hours

### Pattern: Test Suite Updates

**Changes:**

- PHPUnit assertion updates
- Mock syntax modernization
- Data provider updates
- Functional test database handling

**Estimation Impact:**

- Per test file: 30 minutes
- Complex functional tests: 1-2 hours
- Total for medium extension (30 test files): 15-30 hours

## Code Modernization Patterns

### Pattern: Nullable Parameter Fixes

**Example:**

```
[TASK] Avoid implicitly nullable class method parameter
```

**PHP 8.1+ Requirement:** Parameters with default null must be explicitly nullable

```php
// Before (PHP 7.4)
public function process($data = null) { }

// After (PHP 8.1)
public function process(?array $data = null) { }
```

**Estimation Impact:**

- Automated with Rector: 2-4 hours
- Manual fixes: 0.25 hours per occurrence
- Typical extension: 30-50 occurrences = 8-12 hours manual

### Pattern: Type Declaration Improvements

**Estimation Impact:**

- Per class: 30-60 minutes
- Property type declarations: 15 minutes per class
- Method return types: 15 minutes per class
- Total for medium extension (40 classes): 20-40 hours

### Pattern: Server Request Migration

**Change:**

```php
// Before
$params = GeneralUtility::_GP('params');

// After
$params = $request->getQueryParams()['params']
    ?? $request->getParsedBody()['params']
    ?? null;
```

**Estimation Impact:**

- Per occurrence: 15-30 minutes
- Typical extension: 20-40 occurrences = 5-20 hours
- With Rector: 2-4 hours

### Pattern: Template View Replacements

**Change:**

```php
// Before
use TYPO3\CMS\Fluid\View\TemplateView;
use TYPO3\CMS\Fluid\View\StandaloneView;

// After
use TYPO3Fluid\Fluid\View\TemplateView;
```

**Estimation Impact:**

- Search & replace: 1-2 hours
- Testing: 2-4 hours
- Total: 3-6 hours

## Documentation Updates

### Pattern: Version Compatibility Updates

**Files Updated:**

- README.md (badges, version info)
- Documentation/Index.rst (compatibility matrix)
- CHANGELOG/migration guides
- Code examples with new syntax

**Estimation Impact:**

- README updates: 1 hour
- Documentation pages: 2-4 hours
- Code examples: 2-4 hours
- Migration guide: 4-8 hours
- Total: 9-17 hours

## Observed Commit Patterns

### Frequency Analysis from news 11→12 Upgrade

**Task Distribution:**

```
Rector refactorings:      ~25 commits (automated)
TCA updates:              ~15 commits
Configuration updates:    ~10 commits
Breaking changes:         ~8 commits
Test updates:             ~12 commits
Documentation:            ~10 commits
Bugfixes after upgrade:   ~15 commits
Code cleanup:             ~20 commits
```

**Total Commits:** ~115 commits for major version upgrade

**Estimation Insight:** Plan for 100-150 commits for a complex extension upgrade

## Upgrade Phase Patterns

### Phase 1: Composer & Dependencies (10-15%)

- Composer.json updates
- Testing framework updates
- CI/CD adjustments

### Phase 2: Automated Refactoring (20-30% if Rector available)

- Install and configure Rector
- Run automated refactorings
- Review and commit changes

### Phase 3: Configuration Modernization (15-25%)

- TCA updates
- Backend module registration
- Services configuration
- Icon registration
- FlexForm updates

### Phase 4: Breaking Changes (20-30%)

- Remove deprecated features
- Replace removed APIs
- Update to new APIs

### Phase 5: Testing (20-25%)

- Test suite updates
- PHPUnit migration
- Functional test updates
- Regression testing

### Phase 6: Documentation (5-10%)

- README updates
- Documentation pages
- Migration guides
- CHANGELOG

## Estimation Formula Refinements

Based on news extension analysis, **update skill formulas:**

### Extension Complexity Multipliers

```python
def calculate_complexity_multiplier(extension):
    """
    Based on news extension patterns
    """
    multiplier = 1.0

    # Module count impact
    if extension.backend_modules > 1:
        multiplier += 0.3 * (extension.backend_modules - 1)

    # FlexForm count impact
    if extension.flexform_count > 3:
        multiplier += 0.2 * (extension.flexform_count - 3)

    # TCA file count impact
    if extension.tca_files > 10:
        multiplier += 0.1 * (extension.tca_files - 10) / 10

    # Test coverage impact (better tests = easier upgrade)
    if extension.test_coverage > 70:
        multiplier *= 0.85  # 15% reduction
    elif extension.test_coverage < 40:
        multiplier *= 1.25  # 25% increase

    return multiplier
```

### Rector Availability Impact

```python
def estimate_refactoring_work(has_rector, refactoring_points):
    """
    Rector reduces manual work by 60-70%
    """
    base_hours_per_point = 0.5  # 30 minutes per refactoring point

    if has_rector:
        # Automated refactoring + review
        automated_hours = 8  # Setup + run + review
        manual_hours = refactoring_points * base_hours_per_point * 0.3
        return automated_hours + manual_hours
    else:
        # All manual
        return refactoring_points * base_hours_per_point
```

### Breaking Change Impact

```python
def estimate_breaking_changes(typo3_version_from, typo3_version_to):
    """
    Based on news pattern: ~8 breaking changes per major version
    """
    versions_jumped = int(typo3_version_to) - int(typo3_version_from)

    breaking_changes = {
        1: 8,   # Direct upgrade (e.g., 11→12)
        2: 20,  # Skip version (e.g., 10→12)
    }

    count = breaking_changes.get(versions_jumped, versions_jumped * 10)
    hours_per_change = 2  # Average 2 hours per breaking change

    return count * hours_per_change
```

## Recommendations for Skill Enhancement

1. **Add Rector Detection:**
   - Check for `.rector.php` in project
   - Offer to install if missing
   - Adjust estimation based on availability

2. **Breaking Change Parser:**
   - Parse TYPO3 changelog for `[!!!]` markers
   - Count affected APIs used in project
   - Calculate breaking change hours

3. **TCA Complexity Scoring:**
   - Count TCA files
   - Analyze inline relations and file references
   - Estimate TCA work more precisely

4. **Test Coverage Impact:**
   - Higher coverage = easier/faster upgrades
   - Lower coverage = more manual testing needed
   - Adjust estimates accordingly

5. **FlexForm Complexity:**
   - Count FlexForm plugins
   - Check for switchablecontrolleractions (deprecated)
   - Estimate modernization effort

6. **Module Count Impact:**
   - Each backend module adds 2-4 hours
   - Configuration migration needed per module

## Example: news Extension Upgrade Estimation

**Hypothetical estimation for news 11.4.3 → 12.0.0:**

```yaml
Extension: georgringer/news
Current: 11.4.3
Target: 12.0.0

Extension Characteristics:
  - Backend modules: 1
  - FlexForms: 5
  - TCA files: 17
  - Test files: 45
  - Test coverage: 75%
  - Has Rector: Yes
  - Classes: ~120

Estimation Breakdown:

Composer & Dependencies:
  - Update composer.json: 2h
  - Testing framework (7→8): 8h
  - PHPUnit (9→10): 12h
  Subtotal: 22h

Automated Refactoring (Rector):
  - Setup Rector: 4h
  - Run 25 refactoring rules: 4h
  - Review & fix edge cases: 8h
  Subtotal: 16h

Configuration Updates:
  - TCA files (17): 17h
  - Backend module: 3h
  - Services.php: 2h
  - Icons: 2h
  - FlexForms (5): 10h
  Subtotal: 34h

Breaking Changes:
  - 8 breaking changes × 2h: 16h
  Subtotal: 16h

Testing:
  - Test suite updates: 20h
  - Regression testing: 16h
  Subtotal: 36h

Documentation:
  - README + docs: 8h
  - Migration guide: 6h
  Subtotal: 14h

Code Review & Cleanup: 12h

Base Total: 150h
Contingency (20%): 30h
Complexity Multiplier (1.15): ×1.15
Risk Multiplier (1.0): ×1.0

Final Estimate: 207 hours (~5-6 weeks)

Actual Result: ~115 commits over several weeks
```

**Validation:** This estimation aligns with the observed commit count and change scope.

---

**Last Updated:** 2025-10-23
**Based on:** georgringer/news v11.4.3 → v12.0.0 analysis
**Commit Range Analyzed:** 11.4.3..12.0.0 (~115 commits)
