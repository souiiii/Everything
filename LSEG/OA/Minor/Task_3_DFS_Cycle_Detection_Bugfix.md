# Task 3 - DFS Cycle Detection Bug Fix

**Classification:** Minor

## Question

The exact original prompt is not fully preserved. The supplied material shows a Java DFS implementation in which already-visited nodes were handled with:

```java
if (visited.contains(node)) {
    return false;
}
```

The task was to correct the bug. A previously visited node can already be known to be blocked because it leads to a cycle, so returning `false` loses that information.

## Answer

Replace:

```java
if (visited.contains(node)) {
    return false;
}
```

with:

```java
if (visited.contains(node)) {
    return cycleNodes.contains(node);
}
```

Corrected DFS:

```java
private boolean dfs(
        String node,
        Map<String, List<String>> graph,
        Set<String> visiting,
        Set<String> visited,
        Set<String> cycleNodes) {

    if (visiting.contains(node)) {
        cycleNodes.add(node);
        return true;
    }

    if (visited.contains(node)) {
        return cycleNodes.contains(node);
    }

    visiting.add(node);

    for (String nxt : graph.getOrDefault(node, Collections.emptyList())) {
        if (dfs(nxt, graph, visiting, visited, cycleNodes)) {
            cycleNodes.add(node);
        }
    }

    visiting.remove(node);
    visited.add(node);

    return cycleNodes.contains(node);
}
```

Everything else in `Solution.java` can remain unchanged.
