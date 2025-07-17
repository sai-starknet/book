# Describing the Storage Layout

For contracts to interact with each other's storage systems, they need a standardized way to communicate storage layouts. This is particularly important when contracts need to read from or write to external storage systems or when building composable applications. Different applications will benefit from different storage layouts, so the system should be flexible enough to allow for different layouts while still being able to communicate them effectively.

One example is a solution that allows for the member's full key to be parsed to the contract allowing compile-time hashing of the member attributes. For a fixed layout it could just consist of a list of values and member-attribute storage locations which would use the `[[STORAGE_BASE, table], [member-attribute, row]]` format. To read a fixed it would make sense to send the full layout and the row ID separately as with enums and we are unsure which bits are needed.
