# Referenceable Contracts

One of the useful things about Dojo is the ability to upgrade contracts without directly knowing the contract address. This can be implemented through various approaches including registries and deterministic deployment patterns.

## Static Class Hash

One approach is to deploy all contracts from a known class hash and upgrade them immediately as part of a multicall. The tag can then be hashed and used as the salt, so no matter what class the contract gets upgraded to, the address will always be obtainable.

```rust
#[starknet::contract]
mod base_contract {
use starknet::{ClassHash, SyscallResultTrait};

    #[storage]
    struct Storage {}

    #[external(v0)]
    fn upgrade(
        ref self: ContractState, class_hash: ClassHash, selector: felt252, calldata: Span<felt252>,
    ) -> Span<felt252> {
        starknet::syscalls::replace_class_syscall(class_hash).unwrap_syscall();

        starknet::syscalls::call_contract_syscall(
            starknet::get_contract_address(), selector, calldata,
        )
            .unwrap_syscall()
    }
}
```

An issue with this is that the constructor will not be called when the contract is upgraded, so any initialization logic will need to be handled in the upgrade function or through a separate initialization call. This could be made into a macro that creates a function that can only be called by the contract itself.

```rust

#[sai::constructor]
fn constructor(ref self: ContractState, args...) {
    // Initialisation logic here
}
```

that gets converted to:

```rust
#[external(v0)]
fn __constructor__(ref self: ContractState, args...) {
    assert(starknet::get_caller_address() == starknet::get_contract_address(), 'Cannot call constructor');
    // Initialisation logic here
}

```

This way the constructor can be called after the contract is upgraded, but only by the contract itself, which unless someone was looking to break it should mean it's only called on upgrade.

## DNS

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
