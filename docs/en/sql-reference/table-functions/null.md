---
description: 'Creates a temporary table of the specified structure with the Null table
  engine. The function is used for the convenience of test writing and demonstrations.'
sidebar_label: 'null function'
sidebar_position: 140
slug: /sql-reference/table-functions/null
title: 'null'
---

# null Table Function

Creates a temporary table of the specified structure with the [Null](../../engines/table-engines/special/null.md) table engine. According to the `Null`-engine properties, the table data is ignored and the table itself is immediately dropped right after the query execution. The function is used for the convenience of test writing and demonstrations.

## Syntax {#syntax}

```sql
null('structure')
```

## Argument {#argument}

- `structure` — A list of columns and column types. [String](../../sql-reference/data-types/string.md).

## Returned value {#returned_value}

A temporary `Null`-engine table with the specified structure.

## Example {#example}

Query with the `null` function:

```sql
INSERT INTO function null('x UInt64') SELECT * FROM numbers_mt(1000000000);
```
can replace three queries:

```sql
CREATE TABLE t (x UInt64) ENGINE = Null;
INSERT INTO t SELECT * FROM numbers_mt(1000000000);
DROP TABLE IF EXISTS t;
```

## Related {#related}

- [Null table engine](../../engines/table-engines/special/null.md)
