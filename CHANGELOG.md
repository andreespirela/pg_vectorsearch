# Changelog

## [0.1.0-rc2] - 2026-10-11

### Highlights

This is the second release candidate for 0.1.0. The extension is the
same as in 0.1.0-rc1. It is still alpha quality and meant for testing,
not production.

0.1.0-rc1 was published without a source tarball and checksum. This
candidate is the first to ship them. If you already run 0.1.0-rc1,
there is no reason to upgrade.

### Breaking changes & upgrade notes

None.

There is no upgrade path from 0.1.0-rc1. `ALTER EXTENSION ... UPDATE`
has no script to run. To switch, drop and recreate the extension and
its indexes.

### Known issues

The known issues listed for 0.1.0-rc1 below all still apply.

### Changes

#### Bug fixes

- *ci*: Release assets are attached before the release is published.
  An immutable release refused them afterwards, which left 0.1.0-rc1
  without a tarball ([d4d3167](https://github.com/timescale/pg_vectorsearch/commit/d4d3167e4b214326869b6cac94af6ceb56a99ace))

[Full changelog](https://github.com/timescale/pg_vectorsearch/compare/v0.1.0-rc1...v0.1.0-rc2)

## [0.1.0-rc1] - 2026-10-08

### Highlights

This is the first release candidate for 0.1.0, and the first release of
pg_vectorsearch. It is alpha quality and meant for testing, not
production.

### Breaking changes & upgrade notes

None.

An upgrade from this release candidate to a later release may not be
possible. `CREATE EXTENSION` warns about this on any prerelease. Plan to
drop and recreate the extension and its indexes.

### Known issues

- A `WHERE` filter can return fewer rows than `LIMIT` when the rows
  that pass are not spread through the candidates the scan ranked. The
  scan does not go back for more
  ([#188](https://github.com/timescale/pg_vectorsearch/issues/188)).
- An ordered scan with no `LIMIT`, or a `LIMIT` larger than `work_mem`
  allows it to rank, returns only as many rows as that budget affords,
  or the table's estimated row count if that is smaller. It does not
  error. Raising `work_mem` covers the budget case.
- `prism_rebalance` leaves the posting lists it replaced on disk. Only
  `REINDEX` reclaims that space
  ([#224](https://github.com/timescale/pg_vectorsearch/issues/224)).
- Setting `prism.leaf_refine_threshold` above 0 can build an index with
  much lower recall. Leave it at 0
  ([#183](https://github.com/timescale/pg_vectorsearch/issues/183)).
- `prism` refuses unlogged tables and expression indexes, and indexes
  at most 1968 dimensions.
