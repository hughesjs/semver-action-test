# semver-action-test

Test repository demonstrating the pagination bug in [ietf-tools/semver-action](https://github.com/ietf-tools/semver-action).

## The Bug

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

This repository uses **12 annotated tags with controlled tagger dates** to reliably demonstrate the bug:

```
Position in API response:
┌─────────────────────────────────────────────┐
│  1. build-11  (newest tagger date)          │
│  2. build-10                                │
│  3. build-9                                 │
│  4. build-8                                 │
│  5. build-7                                 │
│  6. build-6                                 │  ← IETF version only
│  7. build-5                                 │    sees these 10 tags
│  8. build-4                                 │
│  9. build-3                                 │
│ 10. build-2                                 │
├─────────────────────────────────────────────┤
│ 11. build-1                                 │  ← Fork version
│ 12. v1.0.0   (oldest tagger date)           │    paginates to find this
└─────────────────────────────────────────────┘
```

**Why annotated tags?** Git's GraphQL API sorts tags by tagger date for annotated tags, giving us precise control over ordering. The `build-*` tags have newer tagger dates than `v1.0.0`, pushing the valid semver tag beyond position 10.

## Results

| Workflow | Action Version | Status | Explanation |
|----------|----------------|--------|-------------|
| [Test IETF Version](https://github.com/hughesjs/semver-action-test/actions/runs/21120279507) | `ietf-tools/semver-action@v1` | ❌ FAIL | `maxTagsToFetch: 150` resets to 10, misses `v1.0.0` |
| [Test Fork Version](https://github.com/hughesjs/semver-action-test/actions/runs/21120279871) | `hughesjs/semver-action@remove-100-tag-limit` | ✅ PASS | Pagination finds `v1.0.0` at position 12 |

### IETF Version Output
```
None of the 10 latest tags are valid semver!
```

### Fork Version Output
```
Current: v1.0.0
Next: v1.0.1
```

## Related PR

- [ietf-tools/semver-action#69](https://github.com/ietf-tools/semver-action/pull/69) - Fix: remove 100 tag limit by implementing pagination
