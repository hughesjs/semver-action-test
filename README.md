# semver-action-test

Test repository demonstrating the pagination issue in [ietf-tools/semver-action](https://github.com/ietf-tools/semver-action).

## The Issue

When `maxTagsToFetch` is set to a value greater than 100, the action silently resets it to **10**:

```javascript
// From index.js - the problematic line
const fetchLimit = (maxTagsToFetch < 1 || maxTagsToFetch > 100) ? 10 : maxTagsToFetch
```

This means:
- Setting `maxTagsToFetch: 150` actually fetches only **10 tags**
- If your valid semver tag isn't in the 10 most recent tags, the action fails
- The error message "None of the X latest tags are valid semver!" is misleading

## Test Methodology

This repository uses **150 annotated tags with controlled tagger dates** to reliably demonstrate the issue:

- **149 `build-*` tags** (build-1 through build-149) with incrementing tagger dates
- **1 `v1.0.0` tag** with the oldest tagger date

```
Position in API response:
┌─────────────────────────────────────────────┐
│   1. build-149  (newest tagger date)        │
│   2. build-148                              │
│   ...                                       │
│  10. build-140                              │  ← IETF version stops here
├─────────────────────────────────────────────┤
│  11. build-139                              │
│  ...                                        │
│ 100. build-50                               │  ← End of page 1
├─────────────────────────────────────────────┤
│ 101. build-49                               │  ← Start of page 2
│  ...                                        │
│ 149. build-1                                │
│ 150. v1.0.0   (oldest tagger date)          │  ← Fork finds this!
└─────────────────────────────────────────────┘
```

**Why annotated tags?** Git's GraphQL API sorts tags by tagger date for annotated tags, giving us precise control over ordering. The `build-*` tags have newer tagger dates than `v1.0.0`, pushing the valid semver tag to position 150.

## Test Cases

### Test Case 1: Basic (12 tags)
Demonstrates the issue with minimal tags - `v1.0.0` at position 12.

| Workflow | Action Version | Status | Explanation |
|----------|----------------|--------|-------------|
| [Test IETF Version](https://github.com/hughesjs/semver-action-test/actions/runs/21120279507) | `ietf-tools/semver-action@v1` | ❌ FAIL | `maxTagsToFetch: 150` resets to 10, misses `v1.0.0` |
| [Test Fork Version](https://github.com/hughesjs/semver-action-test/actions/runs/21120279871) | `hughesjs/semver-action@remove-100-tag-limit` | ✅ PASS | Finds `v1.0.0` at position 12 |

### Test Case 2: Full Pagination (150 tags)
Proves the fix works with pagination across multiple pages - `v1.0.0` at position 150.

| Workflow | Action Version | Status | Explanation |
|----------|----------------|--------|-------------|
| [Test 150 Tags - IETF](https://github.com/hughesjs/semver-action-test/actions/runs/21120727881) | `ietf-tools/semver-action@v1` | ❌ FAIL | Only fetches 10 tags, misses `v1.0.0` at position 150 |
| [Test 150 Tags - Fork](https://github.com/hughesjs/semver-action-test/actions/runs/21120728266) | `hughesjs/semver-action@remove-100-tag-limit` | ✅ PASS | Paginates through 100 + 50 tags to find `v1.0.0` |

### Expected Output

**IETF Version (fails):**
```
None of the 10 latest tags are valid semver!
```

**Fork Version (passes):**
```
Current: v1.0.0
Next: v1.0.1
```

## Related PR

- [ietf-tools/semver-action#69](https://github.com/ietf-tools/semver-action/pull/69) - Fix: remove 100 tag limit by implementing pagination
