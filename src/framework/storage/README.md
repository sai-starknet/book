# Storage

We can think of the address of a single storage value as consisting of:

- a base storage salt (for security)
- a table ID (which could be composed of a namespace and a table)
- a row
- a member attribute, where this attribute is the combined hash of the member and its path down to the value.

There are many ways to order the hashing of these elements, and the most efficient approach depends on several factors. Generally, we aim to move as much hashing to **compile time** as possible, since **run-time hashing is expensive**.
