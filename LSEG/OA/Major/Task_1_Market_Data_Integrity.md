# Task 1 - Market Data Integrity

**Classification:** Major

## Question

The exact original wording is not preserved in the supplied PDF. From the preserved solution, the task was to process market-data messages of the form:

```text
[sequenceNumber, messageId, instrument, price]
```

and return three results:

1. Duplicate messages, reported as:
   ```text
   [messageId, firstIndex, duplicateIndex]
   ```
2. Missing sequence-number ranges.
3. The latest price for each instrument, using the greatest sequence number. If two messages for an instrument have the same sequence number, the later arrival wins.

## Answer

```javascript
function solution(messages) {
  // 1. Duplicate message detection
  const firstIndex = new Map();
  const duplicateMessages = [];

  // 2. Sequence numbers seen
  const sequenceSet = new Set();

  // 3. Latest price for each instrument
  // instrument -> { sequence, price }
  const latest = new Map();

  for (let i = 0; i < messages.length; i++) {
    const [sequenceNumber, messageId, instrument, price] = messages[i];

    // ----- Duplicate messages -----
    if (firstIndex.has(messageId)) {
      duplicateMessages.push([
        messageId,
        firstIndex.get(messageId),
        i
      ]);
    } else {
      firstIndex.set(messageId, i);
    }

    // ----- Sequence numbers -----
    sequenceSet.add(sequenceNumber);

    // ----- Latest price by instrument -----
    if (
      !latest.has(instrument) ||
      sequenceNumber >= latest.get(instrument).sequence
    ) {
      // >= is intentional:
      // if sequence numbers are equal, later arrival wins
      latest.set(instrument, {
        sequence: sequenceNumber,
        price: price
      });
    }
  }

  // ----- Find missing sequence ranges -----
  const sequences = [...sequenceSet].sort((a, b) => a - b);
  const missingSequenceRanges = [];

  for (let i = 1; i < sequences.length; i++) {
    const previous = sequences[i - 1];
    const current = sequences[i];

    if (current > previous + 1) {
      missingSequenceRanges.push([
        previous + 1,
        current - 1
      ]);
    }
  }

  // ----- Build required instrument -> price object -----
  const latestPriceByInstrument = {};

  for (const [instrument, data] of latest) {
    latestPriceByInstrument[instrument] = data.price;
  }

  return [
    duplicateMessages,
    missingSequenceRanges,
    latestPriceByInstrument
  ];
}

module.exports = solution;
```
