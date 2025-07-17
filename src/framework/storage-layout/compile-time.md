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

- _H_ = Number of hash operations
- _R_ = Number of rows
- _A_ = Number of attributes
