# Task 4 - Order Book Zero-Quantity Bug Fix

**Classification:** Minor

## Question

The supplied Java order-book implementation updates BID/ASK price levels with:

```java
book.put(event.price, event.quantity);
```

The task was to correct the implementation so that an event with `quantity == 0` removes the price level instead of storing a zero quantity. The function then returns the best bid and best ask.

## Answer

Replace:

```java
book.put(event.price, event.quantity);
```

with:

```java
if (event.quantity == 0)
    book.remove(event.price);
else
    book.put(event.price, event.quantity);
```

Full corrected implementation:

```java
import java.util.*;

public class Solution {
    public Double[] solution(List<Event> events) {
        Map<Double, Double> bids = new HashMap<>();
        Map<Double, Double> asks = new HashMap<>();

        for (Event event : events) {
            Map<Double, Double> book =
                event.side.equals("BID") ? bids : asks;

            if (event.quantity == 0) {
                book.remove(event.price);
            } else {
                book.put(event.price, event.quantity);
            }
        }

        Double bestBid = bids.isEmpty()
            ? null
            : Collections.max(bids.keySet());

        Double bestAsk = asks.isEmpty()
            ? null
            : Collections.min(asks.keySet());

        return new Double[]{bestBid, bestAsk};
    }

    public static class Event {
        public String side;
        public double price;
        public double quantity;

        public Event(String side, double price, double quantity) {
            this.side = side;
            this.price = price;
            this.quantity = quantity;
        }
    }
}
```
