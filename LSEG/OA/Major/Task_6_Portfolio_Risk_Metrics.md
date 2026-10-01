# Task 6 - Portfolio Risk Metrics

**Classification:** Major

## Question

A risk system provides portfolio-loss scenarios of the form:

```text
[probability, loss]
```

The scenario probabilities are not guaranteed to sum to `1`, and the input is not guaranteed to be sorted by loss.

Implement:

```text
solution(scenarios, alpha)
```

and return:

```text
[expected_loss, value_at_risk, conditional_tail_loss]
```

The required behavior is:

- `expected_loss`: probability-weighted average loss after normalizing by total probability;
- `value_at_risk`: the smallest loss whose cumulative normalized probability reaches `alpha`;
- scenarios with the same loss must be grouped before testing the VaR boundary;
- `conditional_tail_loss`: weighted average loss among scenarios whose loss is at least the VaR;
- floating-point boundary comparisons should use a small tolerance.

For an empty scenario list, return `[0.0, 0.0, 0.0]`.

## Answer

```javascript
function solution(scenarios, alpha) {
  if (scenarios.length === 0) {
    return [0.0, 0.0, 0.0];
  }

  const EPS = 1e-9;

  let totalProbability = 0;
  let weightedLoss = 0;

  for (const [probability, loss] of scenarios) {
    totalProbability += probability;
    weightedLoss += probability * loss;
  }

  // 1. Expected loss
  const expectedLoss = weightedLoss / totalProbability;

  // Sort by loss
  const sorted = scenarios.slice().sort((a, b) => a[1] - b[1]);

  // 2. Value at Risk
  // Equal losses must be combined before checking the cumulative probability.
  let cumulativeProbability = 0;
  let valueAtRisk = sorted[0][1];

  let i = 0;

  while (i < sorted.length) {
    const currentLoss = sorted[i][1];
    let groupProbability = 0;

    // Combine all scenarios having exactly the same loss
    while (
      i < sorted.length &&
      sorted[i][1] === currentLoss
    ) {
      groupProbability += sorted[i][0];
      i++;
    }

    cumulativeProbability += groupProbability;

    const cumulativeRate =
      cumulativeProbability / totalProbability;

    // Tolerance is required for floating-point boundary cases
    if (cumulativeRate + EPS >= alpha) {
      valueAtRisk = currentLoss;
      break;
    }
  }

  // 3. Conditional Tail Loss
  let tailProbability = 0;
  let tailWeightedLoss = 0;

  for (const [probability, loss] of scenarios) {
    if (loss >= valueAtRisk) {
      tailProbability += probability;
      tailWeightedLoss += probability * loss;
    }
  }

  const conditionalTailLoss =
    tailWeightedLoss / tailProbability;

  return [
    expectedLoss,
    valueAtRisk,
    conditionalTailLoss
  ];
}

module.exports = solution;
```
