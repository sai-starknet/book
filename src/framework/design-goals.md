# Design Goals

**Key Design Goals:**

- **Modularity**: Easy modification and extension for specific use cases
- **Efficiency**: Optimized performance out of the box
- **Developer Experience**: Intuitive APIs using familiar patterns
- **Extensibility**: Well-defined interfaces that can be easily extended or replaced

Parts to consider for the framework:

- **Storage Layout**: Define how data is stored in the blockchain, including how to hash and access it efficiently.
- **Indexer Interface**: Provide a way to emit events that can be indexed by external systems
- **Contract Interfaces**: Defined ABIs that allow contracts to interact with other contracts with known standardised interfaces.
- **Common Tools and Utilities**: Provide a set of common tools and utilities that can be used across different projects, such as permissions management and token utils.
