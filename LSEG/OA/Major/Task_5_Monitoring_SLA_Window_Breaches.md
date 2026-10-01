# Task 5 - Monitoring SLA Window Breaches

**Classification:** Major

## Question

A monitoring system records one sample per minute:

```text
[requests, errors, p95_latency]
```

Implement:

```text
solution(metrics, k, errorThreshold, latencyThreshold)
```

Consider every consecutive window of `k` minutes and return the starting indices of windows that breach the SLA.

A window breaches if either:

- its aggregate error rate is greater than `errorThreshold`; or
- its maximum `p95_latency` is greater than `latencyThreshold`.

Hitting a threshold exactly does not count as a breach. The required solution must handle large inputs efficiently.

## Answer

```javascript
function solution(metrics, k, errorThreshold, latencyThreshold) {
  const n = metrics.length;

  if (n < k) return [];

  const result = [];

  let totalRequests = 0;
  let totalErrors = 0;

  // Monotonic deque storing indices.
  // Front always contains the maximum latency in current window.
  const deque = [];
  let front = 0;

  for (let i = 0; i < n; i++) {
    const requests = metrics[i][0];
    const errors = metrics[i][1];
    const latency = metrics[i][2];

    // Add current element to window sums
    totalRequests += requests;
    totalErrors += errors;

    // Maintain decreasing deque of latencies
    while (
      deque.length > front &&
      metrics[deque[deque.length - 1]][2] <= latency
    ) {
      deque.pop();
    }

    deque.push(i);

    // Remove element that has left the window
    if (i >= k) {
      totalRequests -= metrics[i - k][0];
      totalErrors -= metrics[i - k][1];
    }

    // Remove expired latency indices
    while (front < deque.length && deque[front] <= i - k) {
      front++;
    }

    // Full window has been formed
    if (i >= k - 1) {
      let errorBreach = false;

      if (totalRequests > 0) {
        // Important: use division exactly as required by the problem
        const errorRate = totalErrors / totalRequests;
        errorBreach = errorRate > errorThreshold;
      }

      const maxLatency = metrics[deque[front]][2];
      const latencyBreach = maxLatency > latencyThreshold;

      if (errorBreach || latencyBreach) {
        result.push(i - k + 1);
      }
    }
  }

  return result;
}

module.exports = solution;
```
