# Stage Reference Overview

Fluxion implements a large subset of MongoDB aggregation stages. Use this index to jump to detailed pages for each stage and discover related functionality.

## Core Transformation Stages

- [$match](match.md): filter documents with MongoDB query semantics.
- [$project](project.md): reshape documents, include/exclude fields, compute expressions.
- [$addFields](addFields.md) / [$set](set.md): append or overwrite fields with aggregation expressions.
- [$unset](unset.md): remove fields from documents.
- [$replaceRoot](replaceRoot.md) / [$replaceWith](replaceWith.md): promote a sub-document to the root.
- [$limit](limit.md) and [$skip](skip.md): paginate documents flowing through the pipeline.
- [$sort](sort.md) and [$sortByCount](sortByCount.md): reorder documents or tally counts by expression.
- [$unwind](unwind.md): explode arrays into individual documents.

## Grouping & Bucketing

- [$group](group.md): group by `_id` and apply accumulators (`$sum`, `$avg`, etc.).
- [$bucket](bucket.md) and [$bucketAuto](bucketAuto.md): bucket values using explicit boundaries or automatic distribution.
- [$facet](facet.md): run multiple pipelines in parallel and merge the results.

## Windowing & Time Series

- [$setWindowFields](setWindowFields.md): apply window functions with partitioning and ordering.
- [$densify](densify.md): fill gaps in time-series or sequence data.
- [$fill](fill.md): carry forward or interpolate missing values.

## Lookup & Integrations

- [$lookup](lookup.md): join with another collection or pipeline.

## Advanced / Specialized

- [$merge](merge.md), [$out](out.md), [$search](search.md), [$vectorSearch](vectorSearch.md): environment-specific stages with additional deployment prerequisites.

Use the sidebar for the full alphabetical list. Each stage page covers payload syntax, options, and representative examples you can drop into your pipelines.
