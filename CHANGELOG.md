# Changelog

## [0.1.0] - 2026-10-13

### Highlights

This is the first release of pg_vectorsearch, currently at alpha level. The
core works and you can test it on real data today, but there are gaps you
should know about before you do (see Known Issues below). This is what works
today:

- Creating a PRISM index. Index builds on a populated table, including
  parallel builds, and k-nearest-neighbor search with L2, inner
  product and cosine distance.
- No dependencies. pg_vectorsearch ships its own `vec32` and `vec16`
  types so that it can operate independently. But PRISM also works
  with pgvector's `vector` and `halfvec` data types. You can build a
  PRISM index directly on existing pgvector columns with no rewrites
  and no copies, and queries written with pgvector's operators work
  unchanged.
- Simplified tuning. For queries, there is a single parameter to
  tune: `prism.nprobe`, the number of clusters a query scans. The
  default of 0 picks a value automatically to target roughly 95%
  recall.
- Inserts and updates work, but heavy churn can cause recall drift
  without manual intervention. Right now, partitions are not
  re-split automatically under churn, so the partitioning can become
  suboptimal. You can split oversized partitions manually by calling
  `prism_rebalance`, or rebuild the full index from scratch with
  `REINDEX`. We plan to remove the need for these manual steps and
  handle drift automatically. For this version, we don't recommend
  creating the index on an empty table and filling it afterwards.
- Delete operations work but don't yet reuse space. Deletes are
  turned into tombstones by VACUUM so that scans can skip them, but
  the space inside index pages isn't yet reused. Run `REINDEX` to
  reclaim it.

### Breaking changes & upgrade notes

None.

There is no upgrade path from 0.1.0-rc1 or 0.1.0-rc2. To switch, drop
and recreate the extension and its indexes. From this release
on, upgrades between final releases ship an upgrade script.

### Known issues

- A `WHERE` filter can return fewer rows than `LIMIT` when the rows
  that pass are not spread through the candidates the scan ranked. The
  scan does not go back for more
  ([#188](https://github.com/timescale/pg_vectorsearch/issues/188)).
- The number of rows an ordered scan can return is capped by a
  buffer sized from `work_mem`. A `LIMIT` larger than that buffer
  holds -- or no `LIMIT` at all -- is truncated to however many
  rows fit (or the table's estimated row count, if smaller),
  silently, with no error. Raise `work_mem` to return more rows.
- `prism_rebalance` leaves the posting lists it replaced on disk. Only
  `REINDEX` reclaims that space
  ([#224](https://github.com/timescale/pg_vectorsearch/issues/224)).
- Setting `prism.leaf_refine_threshold` above 0 can build an index with
  much lower recall. Leave it at 0
  ([#183](https://github.com/timescale/pg_vectorsearch/issues/183)).
- `prism` refuses unlogged tables and expression indexes, and indexes
  at most 1968 dimensions.

### Changes

- The extension's description in `\dx` now reads "ANN vector search
  with PRISM index"
  ([eedfabe](https://github.com/timescale/pg_vectorsearch/commit/eedfabe376857dab69365dc42329095edf4381d6))
- The README's usage example inserts enough rows for the planner to
  choose the index
  ([da022db](https://github.com/timescale/pg_vectorsearch/commit/da022dbd99310cb64c5aa4e14e0ec9d50a68805f))

Every change is in the commit history:
<https://github.com/timescale/pg_vectorsearch/commits/v0.1.0>

[Full changelog](https://github.com/timescale/pg_vectorsearch/compare/v0.1.0-rc2...v0.1.0)

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
