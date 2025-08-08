# Time Library

The `std::time` library provides utilities for handling time durations and timestamps in Sway smart contracts.

## Duration

Represents a span of time in seconds.

### Creating Durations

```sway
fn create_durations() {
    // Using constants
    let zero = Duration::ZERO;
    let second = Duration::SECOND;
    let minute = Duration::MINUTE;
    let hour = Duration::HOUR;
    let day = Duration::DAY;
    let week = Duration::WEEK;

    // Using constructor methods
    let thirty_seconds = Duration::seconds(30);
    let two_hours = Duration::hours(2);
    let three_days = Duration::days(3);
}
```

### Converting Durations

Time `std::time` library supports conversion between different time scales such as `seconds`, `minutes`, `hours`, `days`, and `weeks`.

```sway
fn convert_durations() {
    let two_days = Duration::days(2);

    assert(two_days.as_seconds() == 172800); // 2 * 86400
    assert(two_days.as_minutes() == 2880); // 2 * 1440
    assert(two_days.as_hours() == 48); // 2 * 24
    assert(two_days.as_days() == 2);
    assert(two_days.as_weeks() == 0); // Truncated value
}
```

### Operations

The `std::time` supports operations on the `Duration` type.

```sway
fn duration_operations() {
    let day1 = Duration::DAY;
    let day2 = Duration::days(1);

    // Equality
    assert(day1 == day2);

    // Addition
    let two_days = day1 + day2;
    assert(two_days.as_days() == 2);

    // Subtraction
    let half_day = two_days - Duration::days(1).add(Duration::hours(12));
    assert(half_day.as_hours() == 12);

    // Comparison
    assert(Duration::MINUTE < Duration::HOUR);
}
```

## Time

Represents a UNIX timestamp (seconds since Jan 1, 1970).

### Creating Timestamps

There are 3 major ways to create a new timestamp.

```sway
fn create_timestamps() {
    // Current block time
    let now = Time::now();

    // Specific block time
    let block_time = Time::block(12345);

    // From UNIX timestamp
    let custom_time = Time::new(1672531200); // Jan 1, 2023 00:00:00 UTC
}
```

### Time Operations

Operations on the `Time` type are supported with conjunction of the `Duration` type.

```sway
fn time_operations() {
    let now = Time::now();
    let yesterday = now.subtract(Duration::DAY);
    let tomorrow = now.add(Duration::DAY);

    // Duration calculations
    let elapsed = now.duration_since(yesterday).unwrap();
    assert(elapsed.as_days() == 1);

    // Comparison
    assert(yesterday < now);
    assert(tomorrow > now);
}
```

### TAI64 Conversion

The Fuel VM internally uses TAI64 time. Conversions between UNIX and TAI64 are maintained with the `Time` type.

```sway
fn tai64_conversion() {
    let now = Time::now();

    // Convert to TAI64
    let tai64 = now.as_tai64();

    // Convert back to UNIX time
    let converted = Time::from_tai64(tai64);
    assert(now == converted);
}
```

### TAI64 vs UNIX Time

#### Conversion Details

The library uses:

```sway
const TAI_64_CONVERTER: u64 = 10 + (1 << 62);
```

(1 << 62) (0x4000000000000000) marks value as TAI64. 10 accounts for initial TAI-UTC offset in 1970.

Conversion formulas:

`UNIX → TAI64: tai64 = unix + TAI_64_CONVERTER`

`TAI64 → UNIX: unix = tai64 - TAI_64_CONVERTER`

#### Key Differences

| Feature      | TAI64                    | UNIX                      |
|--------------|--------------------------|---------------------------|
| Epoch        | 1970-01-01 00:00:00 TAI  | 1970-01-01 00:00:00 UTC   |
| Leap Seconds | No leap seconds          | Includes leap seconds     |
| Stability    | Continuous time scale    | Discontinuous adjustments |
| Value Range  | (1 << 62) + offset (10s) | Seconds since epoch       |

#### Why TAI64?

* Deterministic execution: No leap second ambiguities
* Monotonic time: Always increases steadily
* Blockchain-friendly: Aligns with Fuel's timestamp mechanism

## Best Practices

1. Use `Duration` for time spans instead of raw seconds
2. Always handle `TimeError` results from `duration_since()` and `elapsed()`
3. Convert to TAI64 when interacting with blockchain primitives
4. Use `Time::block()` for historical time comparisons
5. Prefer duration constants (`SECOND`, `HOUR`, etc.) for readability

## Limitations

1. Durations only support second-level precision
2. Time comparisons are limited to u64 range (584 billion years)
3. No calendar/date functionality (only timestamps)
4. Duration conversions truncate fractional units
