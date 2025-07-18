# Compile-Time Optimization

For fixed models (i.e., no arrays), the column hash for the struct can be computed entirely at compile time.

- It's safe to assume that:
  - The table is known at compile time.
  - The row is usually only known at run time.

The hash efficiency also depends on whether the tables are:

- Locally controlled: Accessed via typed functions
- Externally controlled: Accessed via generic write functions

If the table is known at compile time and locally controlled, we can precompute as follows:

```
[[[STORAGE_BASE, table], member-attribute], row]
[[STORAGE_BASE, [table, member-attribute]], row]
[[[STORAGE_BASE, member-attribute], table], row]
```

This would result in a cost of:
$$H = RA$$

Where:

- *H* = Number of hash operations
- *R* = Number of rows
- *A* = Number of attributes

## Compile-Time vs Run-Time Hashing Strategy

We want to split hashing into compile-time and run-time components, keeping as much in the compile-time phase as possible.

- **Compile-time storage base**: When table is known locally
- **Run-time storage base**: When accessed externally

When parts of the struct are fixed (no arrays), member attributes can also be hashed at compile time.

### Example: Partial Compile-Time Hashing

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

- **Static part**: \[a, c\]
- **Dynamic part (indices)**: \[N, M\]

The static parts can be pre-hashed with the table.
