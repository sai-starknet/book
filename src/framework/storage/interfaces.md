# Storage Interfaces

Dojo uses a world-based storage system with one contract (the world) containing all the storage for the entire system. This centralized approach provides excellent security and consistency guarantees, making it ideal for many applications. However, for certain use cases where performance is critical or where developers need more control over storage patterns, individual contract storage can offer advantages such as more compile-time optimizations and reduced hashing overhead.

For applications that benefit from a world storage system, the storage interface should be designed to allow any contract to interact with the world storage in a predictable and standardized way.

The read interface is straightforward, requiring only the table ID, row, and storage layout:

```rust
pub impl IContractStoreRead of IStoreRead<Contract> {
    fn store_read_entity(
        self: @Contract, table: felt252, fields: Span<FieldLayout>, id: felt252
    ) -> Span<felt252>;
    fn store_read_entities(
        self: @Contract, table: felt252, fields: Span<FieldLayout>, ids: Span<felt252>
    ) -> Array<Span<felt252>>;
}
```

When it comes to writing, there are more considerations that need to take place, such as permissions and emitting events for indexing. The dojo system uses models to define tables, which provides a solid foundation that we can build upon.

To extend the dojo system, we can generalize the model format in several ways:

- Replace fixed key structures with more flexible approaches
- Categorize values as either read-write (retrievable by the contract) or write-only (accessible only off-chain)
- Allow row IDs to be computed using whatever method makes sense for the specific use case

Interfaces can then be defined to allow for easy interaction with the storage system, including one that allows for direct integration with existing dojo implementations.
