# semver-action-test

Test repository for verifying the pagination fix in [ietf-tools/semver-action](https://github.com/ietf-tools/semver-action).

## Purpose

This repository contains **150 valid semver tags** (`v1.0.0` through `v150.0.0`) to test that the action correctly paginates through all tags to find the highest version.

## Test Setup

- 150 semver tags: `v1.0.0`, `v2.0.0`, ... `v150.0.0`
- `maxTagsToFetch: 150` configured in workflows
- Action must paginate to find `v150.0.0` as the highest version

## Workflows

- **Test IETF Version**: Uses `ietf-tools/semver-action@v1`
- **Test Fork Version**: Uses `hughesjs/semver-action@remove-100-tag-limit`

## The Bug

The original action has a pagination bug where `perPage` resets to 10 after the first page:

```javascript
// Bug: perPage becomes 10 after first fetch
const perPage = allTags.length === 0 ? Math.min(maxTags, 100) : 10
```

This means:
- First page: 100 tags
- Subsequent pages: 10 tags each

The fix ensures consistent page sizes:

```javascript
// Fix: perPage stays at 100
const perPage = Math.min(maxTags, 100)
```

## Results

Both versions correctly identify `v150.0.0` as current and `v150.1.0` as next. The pagination fix improves efficiency by using consistent 100-tag pages instead of 100→10→10→10...

## Related PR

- [ietf-tools/semver-action#69](https://github.com/ietf-tools/semver-action/pull/69)
