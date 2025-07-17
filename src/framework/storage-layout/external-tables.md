# External Tables

For fixed external structures, we have two possible hash arrangements with their corresponding costs in number of hash operations:

**Hash Arrangement 1:**

```
[[STORAGE_BASE, row], [table, member-attribute]]
```

$$H = R + RA$$

**Hash Arrangement 2:**

```
[[STORAGE_BASE, [table, member-attribute]], row]
```

$$H = A + RA$$

If a simple caller based permissions system is used but if the permissions are based on the table hash, then the cost becomes:

**Hash Arrangement 3:**

```
[[STORAGE_BASE, table], row], member-attribute]
```

$$H = R + RA + 1$$

**Hash Arrangement 4:**

```
[[[STORAGE_BASE, table], member-attribute], row]
```

$$H = A + RA + 1$$

**Hash Arrangement 5:**

```
[[STORAGE_BASE, table], [member-attribute, row]]
```

$$H = 2RA + 1$$

**Where:**

- _H_ = Number of hash operations
- _R_ = Number of rows
- _A_ = Number of attributes
