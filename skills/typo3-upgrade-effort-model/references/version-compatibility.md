# TYPO3/PHP Version Compatibility Matrix

## Hard-Coded TYPO3/PHP Support Matrix

**Reference for all upgrade estimations:**

| TYPO3 Version | Min PHP | Max PHP | Supported PHP Versions | Status |
|---------------|---------|---------|------------------------|--------|
| 10.4 LTS | 7.2 | 7.4 | 7.2, 7.3, 7.4 | ELTS only |
| 11.5 LTS | 7.4 | 8.3 | 7.4, 8.0, 8.1, 8.2, 8.3 | ELTS only |
| 12.4 LTS | 8.1 | 8.4 | 8.1, 8.2, 8.3, 8.4 | Active (until Apr 2026) |
| 13.4 LTS | 8.2 | 8.4 | 8.2, 8.3, 8.4 | Active (until Dec 2027) |

## Recommended PHP Target Versions

**By Upgrade Path:**

| Upgrade Path | Recommended PHP | Rationale |
|--------------|-----------------|-----------|
| 10.4 → 11.5 | **8.3** | Highest version supported by 11.5, good long-term support |
| 11.5 → 12.4 | **8.4** | Highest version supported by 12.4, best performance |
| 12.4 → 13.4 | **8.4** | Already supported in 12.4, smooth transition |

## PHP Version Upgrade Strategy

1. Upgrade PHP to highest version compatible with **current** TYPO3
2. Test thoroughly on current TYPO3 version
3. Fix all PHP compatibility issues (primarily undefined array keys for PHP 8.1+)
4. THEN upgrade TYPO3 version
5. This isolates "Is this PHP or TYPO3?" debugging

## Version-Specific Breaking Changes

### TYPO3 10 → 11

- **PHP**: 7.4 (min) → 8.0/8.1 recommended
- **Major Changes**: Backend routing, DataHandler changes, Fluid changes
- **Extension Impact**: Medium (most extensions compatible)

### TYPO3 11 → 12

- **PHP**: 8.1 (min) → 8.2/8.3 recommended
- **Major Changes**: Asset handling, Configuration API, PSR-14 events
- **Extension Impact**: Medium-High (more breaking changes)

### TYPO3 12 → 13

- **PHP**: 8.2 (min) → 8.4 recommended
- **Major Changes**: File structure (LocalConfiguration.php → settings.php), command names (typo3cms → typo3)
- **Extension Impact**: High (structural changes affect all projects)

### Skip-Version (v9 → 12)

- **PHP**: 7.2/7.4 → 8.1+ (major jump)
- **Cumulative Changes**: All v10, v11, v12 breaking changes apply
- **Extension Impact**: High (cumulative), but single effort vs 3 separate upgrades
- **Timeline**: Comparable to single upgrade (6 months) per MFAG validation
