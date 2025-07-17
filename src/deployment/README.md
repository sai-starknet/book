# Deployment System

Part of this project is a universal deployment system that allows for easy deployment of smart contracts and their associated data structures. This system is designed to be modular and extensible with addons/plugins that can be used to extend its functionality and be easily modifiable to specific use cases. Deployment can be either config or code driven or a combination of the two, allowing for both quick deployments and more complex setups.

An example config could look like this:

```toml

[account]
rpc_url = "http://localhost:5050/"
account_address = "0x2af9427c5a277474c079a1283c880ee8a6f0f8fbf73ce969c08d88befec1bba"
private_key = "0x1800000000300000180000000000030000000000003006001800006600"

# [caller] # Optional
# account_address = "0x127fd5f1fe78a71f8bcd1fec63e3fe2f0486b6ecd5c86a0466c3a21fa5cfcec"
# private_key = "0xc5b2fcab997346f3ea1c00b002ecf6f382c5f9c9659a3894eb783c5320f912"

[classes.bar]
class_hash = "0x1234567890abcdef1234567890abcdef123456789"

[contracts.pragma]
contract_address = "0x2af9427c5a277474c079a1283c880ee8a6f0f8fbf73ce969c08d88befec1bba"

[declare.foo]
name = "my_contract"
contract_path = "example/src/contract.cairo"
casm_path = "example/target/release/my_contract.json"

[deploy.foo]
class = "foo" # can either be a tag of a declared contract or a
class_hash = "0x1234567890abcdef1234567890abcdef123456789" # can be used instead of tag
salt = "0x0"
unique = false
calldata = ["$account::account_address", "$dojo::models:my_model:class_hash"]


[dojo]
models = "all"
events = []
```

It should also include the use of variables (e.g., `$dojo::models:my_model:class_hash`) and allow for plugins to be included that interact with defined interfaces such as permissions or token management.

Separate plugins can be created for Dojo-style deployment allowing for easy management and upgrading of contracts.
