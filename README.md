# semver-action-test

Test repository for verifying the pagination fix in [ietf-tools/semver-action](https://github.com/ietf-tools/semver-action).

## Purpose

This repository contains 150 tags:
- 1 valid semver tag (`v150.0.0`) - created first (oldest by date)
- 149 non-semver tags (`build-1` through `build-149`) - created after (newer)

When fetching tags sorted by date (newest first), `v150.0.0` appears at position 150.

## Workflows

- **Test IETF Version**: Uses the original `ietf-tools/semver-action@v1` - expected to fail (pagination bug limits to first 10 tags)
- **Test Fork Version**: Uses `hughesjs/semver-action@remove-100-tag-limit` - expected to pass (pagination fix fetches all 150 tags)

## Evidence

This demonstrates that without proper pagination, the action cannot find semver tags beyond the first page of results.
