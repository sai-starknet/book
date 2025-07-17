# DNS

Another way to solve the issue is to use a central DNS system that allows contracts to be referenced by a human-readable name. This would allow for easy upgrades and management of contracts by only knowing the world address of the DNS contract, which could be formulated from a tag like above (similar to how Dojo currently does it). This could look something like this:

```rust
use starknet::ContractAddress;

#[starknet::interface]
trait IContractRegistry<TContractState> {
    fn register(
        ref self: TContractState, namespace_hash: felt252, contract_address: ContractAddress,
    );
    fn lookup(self: @TContractState, namespace_hash: felt252) -> ContractAddress;
    fn grant_owner(ref self: TContractState, owner: ContractAddress);
    fn revoke_owner(ref self: TContractState, owner: ContractAddress);
    fn is_owner(self: @TContractState, owner: ContractAddress) -> bool;
}

#[starknet::contract]
mod contract_registry {
    use starknet::storage::{Map, StorageMapReadAccess, StorageMapWriteAccess};
    use starknet::{ContractAddress, get_caller_address};

    #[storage]
    struct Storage {
        contracts: Map<felt252, ContractAddress>,
        owners: Map<ContractAddress, bool>,
    }

    #[abi(embed_v0)]
    impl IContractRegistryImpl of super::IContractRegistry<ContractState> {
        fn register(
            ref self: ContractState, namespace_hash: felt252, contract_address: ContractAddress,
        ) {
            self.assert_caller_is_owner();
            self.contracts.write(namespace_hash, contract_address);
        }

        fn lookup(self: @ContractState, namespace_hash: felt252) -> ContractAddress {
            self.contracts.read(namespace_hash)
        }

        fn grant_owner(ref self: ContractState, owner: ContractAddress) {
            self.assert_caller_is_owner();
            self.owners.write(owner, true);
        }

        fn revoke_owner(ref self: ContractState, owner: ContractAddress) {
            self.assert_caller_is_owner();
            self.owners.write(owner, false);
        }

        fn is_owner(self: @ContractState, owner: ContractAddress) -> bool {
            self.owners.read(owner)
        }
    }

    #[generate_trait]
    impl PrivateImpl of PrivateTrait {
        fn assert_caller_is_owner(self: @ContractState) {
            assert(self.owners.read(get_caller_address()), 'Caller is not owner');
        }
    }
}
```

This could also be extended to actually deploy contracts.

Each of these systems could have their own plugin that works with the deployment system to allow for easy management of contracts and their associated data structures.
