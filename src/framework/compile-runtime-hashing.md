# Compile-Time vs Run-Time Hashing

We want to split hashing into compile-time and run-time components, keeping as much in the compile-time phase as possible.

- Compile-time storage base: When table is known locally
- Run-time storage base: When accessed externally

When parts of the struct are fixed (no arrays), member attributes can also be hashed at compile time.

## Example: Partial Compile-Time Hashing

```rust
struct FooBar {
    a: Array<Bar>,
    b: u32,
}

struct Bar {
    c: Array<felt252>,
    d: felt252
}
```

We can split the member attribute hash into:

- Static part: \[a, c\]
- Dynamic part (indices): \[N, M\]

The static parts can be pre-hashed with the table.
