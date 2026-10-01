# Task 2 - Hierarchical Risk Limits

**Classification:** Major

## Question

A risk system organizes trading entities such as regions, desks, and traders in a hierarchy. Each entity is represented as:

```text
[entity_id, parent_id, own_limit, usage]
```

The effective limit is:

- the entity's `own_limit` when it is not null;
- otherwise inherited recursively from the parent;
- undefined when an entity has no own limit and cannot inherit one from a valid parent.

Return a lexicographically sorted list of violations:

```text
[entity_id, reason]
```

Possible reasons are:

- `MISSING_PARENT` - a non-null parent ID does not exist;
- `USAGE_EXCEEDS_LIMIT` - usage exceeds the entity's effective limit;
- `CHILD_LIMITS_EXCEED_PARENT` - the sum of the effective limits of direct children exceeds the entity's effective limit.

The comparisons use a small floating-point tolerance.

## Answer

```javascript
function solution(records) {
  const EPS = 1e-9;

  // entity_id -> complete record
  const byId = new Map(records.map(r => [r[0], r]));

  // entity_id -> effective limit
  const memo = new Map();

  function getLimit(id) {
    if (memo.has(id)) return memo.get(id);

    const record = byId.get(id);
    const parentId = record[1];
    const ownLimit = record[2];

    let limit;

    // Explicit limit always wins, even if parent is missing
    if (ownLimit !== null) {
      limit = ownLimit;
    }
    // No own limit and cannot inherit
    else if (parentId === null || !byId.has(parentId)) {
      limit = undefined;
    }
    // Inherit recursively
    else {
      limit = getLimit(parentId);
    }

    memo.set(id, limit);
    return limit;
  }

  // Sum effective limits of direct children for each entity
  const childSum = new Map();

  for (const [id, parentId] of records) {
    const limit = getLimit(id);

    if (
      parentId !== null &&
      byId.has(parentId) &&
      limit !== undefined
    ) {
      childSum.set(
        parentId,
        (childSum.get(parentId) || 0) + limit
      );
    }
  }

  const ans = [];

  for (const [id, parentId, ownLimit, usage] of records) {
    const limit = getLimit(id);

    // 1. Missing parent
    if (parentId !== null && !byId.has(parentId)) {
      ans.push([id, "MISSING_PARENT"]);
    }

    // 2. Usage exceeds effective limit
    if (
      limit !== undefined &&
      usage > limit + EPS
    ) {
      ans.push([id, "USAGE_EXCEEDS_LIMIT"]);
    }

    // 3. Sum of direct children's effective limits exceeds this limit
    if (
      limit !== undefined &&
      (childSum.get(id) || 0) > limit + EPS
    ) {
      ans.push([id, "CHILD_LIMITS_EXCEED_PARENT"]);
    }
  }

  // Lexicographical sorting:
  // first entity_id, then reason
  ans.sort((a, b) => {
    if (a[0] < b[0]) return -1;
    if (a[0] > b[0]) return 1;

    if (a[1] < b[1]) return -1;
    if (a[1] > b[1]) return 1;

    return 0;
  });

  return ans;
}

module.exports = solution;
```
