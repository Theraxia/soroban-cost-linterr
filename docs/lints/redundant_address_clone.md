## What it does

Detects unnecessary `.clone()` calls on the Soroban `Address` object when the
original value is not used afterward and could be moved instead.

## Why is this bad

Cloning a Soroban `Address` duplicates a host-side handle; each clone is a
metered operation. Unnecessary clones increase CPU and memory overhead and can
outnumber the real work (authorisation, transfers, events) in a function.

## Examples

```rust
// Bad: original `addr` is not used after clone
fn bad(addr: soroban_sdk::Address) {
    let _c = addr.clone();
}

// Good: move the value instead of cloning
fn good(addr: soroban_sdk::Address) {
    takes_addr(addr);
}
```

## False positives

Known false-positive patterns:

- Cloning a reference `&Address` to obtain an owned `Address` (satisfies the
  borrow checker) is legitimate and is not flagged.
- When the original `Address` is used after the clone, the clone is necessary
  and not reported.
- Generic code where the clone semantics are required by a trait bound may be
  conservatively skipped.

If this lint flags a site that you intend to keep, suppress it with
`#[allow(redundant_address_clone)]`.
