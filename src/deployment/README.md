# Deployment System

Part of this project is a universal deployment system that allows for easy deployment of smart contracts and their associated data structures. This system is designed to be modular and extensible with addons/plugins that can be used to extend its functionality and be easily modifiable to specific use cases.

```
project/
├── elektra.toml              # Default configuration with multicalls
├── elektra-dev.toml          # Development profile overrides
├── elektra-testnet.toml      # Testnet profile overrides
├── elektra-mainnet.toml      # Mainnet profile overrides
└── contracts/
    ├── declares/             # Declaration manifests
    │   ├── default-manifest.json
    │   ├── dev-manifest.json
    │   ├── testnet-manifest.json
    │   └── mainnet-manifest.json
    └── deployments/          # Deployment manifests
        ├── default-manifest.json
        ├── dev-manifest.json
        ├── testnet-manifest.json
        └── mainnet-manifest.json
```

The system supports **multiple configuration approaches**:

- **Profile-based deployment** (inspired by Starkli & Starknet Foundry)
- **Multicall deployment scripts** for complex atomic deployments
- **Variable interpolation** with plugin ecosystem support
- **State tracking** with auto-generated manifests
- **Security best practices** with separate account management

## Configuration Examples

### Default Configuration (`elektra.toml`)

The main configuration file that contains your default deployment setup:

```toml
# Default deployment configuration
[account]
rpc_url = "http://localhost:5050/"
account_address = "0x2af9427c5a277474c079a1283c880ee8a6f0f8fbf73ce969c08d88befec1bba"
private_key = "0x1800000000300000180000000000030000000000003006001800006600"
keystore_path = "./keystore.json"
keystore_password = "$KEYSTORE_PASSWORD"  # Secure env var usage

# Optional caller account for multi-account operations
[caller]
account_address = "0x127fd5f1fe78a71f8bcd1fec63e3fe2f0486b6ecd5c86a0466c3a21fa5cfcec"
private_key = "$CALLER_PRIVATE_KEY"

# External class references (multiple formats supported)
[classes]
erc20 = "0x123456789abcdef123456789abcdef1234567890abcdef1234567890abcdef"

[classes.erc721]
hash = "0xabcdef123456789abcdef123456789abcdef123456789abcdef1234567890abcdef"
metadata = { source = "openzeppelin", version = "0.8.0" }

# External contract references
[contracts]
stark = "0x123456789abcdef123456789abcdef1234567890abcdef1234567890abcdef"

[contracts.pragma]
address = "0x2af9427c5a277474c079a1283c880ee8a6f0f8fbf73ce969c08d88befec1bba"
metadata = { oracle_type = "price_feed", version = "1.2.0" }

# Contract declarations with flexible configuration
[declares.foo]
# Minimal - auto-resolve paths
name = "foo"

[declares.bar]
# Detailed configuration
name = "my_contract"
path = "./target/dev/my_tests_bar.contract_class.json"
file_name = "my_tests_bar.contract_class.json"
metadata = { description = "Token contract", version = "1.0.0" }

# Contract deployments with smart referencing
[deploys.foo]
class = "foo"  # References declared class by tag
salt = "0x0"
unique = false
calldata = ["$account:account_address", "$classes:erc20"]

[deploys.bar]
class_hash = "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
salt = "0x123"
unique = true
calldata = ["My Token", "MTK", 18, 1000000]

# Access control and permissions (plugin system)
[access.writers]
foo = "0x1234567890abcdef1234567890abcdef123456789"
bar = "$account:account_address"
foobar = ["$contracts:pragma:address", "$deploys:foo:address"]

# Plugin configurations
[plugins.dojo]
enabled = true
world_address = "$contracts:world:address"
models = ["Position", "Player", "Game"]


# Multicall deployment sequences - atomic deployment script
[[call]]
call_type = "deploy"
class_hash = "erc20_class"
inputs = ["My Game Token", "MGT", 18, 1000000000]
salt = "0x0"
unique = false
id = "game_token"

[[call]]
call_type = "deploy"
class = "some_class"
inputs = ["TokenName", "$account:account_address"]  # Reference deployed contract
salt = "0x123"
unique = true
id = "game_contract"

[[call]]
call_type = "invoke"
contract_address = "game_token"
function = "transfer"
inputs = ["game_contract", 100000000]  # Fund the game contract

[[call]]
call_type = "invoke"
contract_address = "game_contract"
function = "initialize"
inputs = ["0x1", "0x2", "0x3"]
```

### Profile-Specific Overrides (`elektra-<profile>.toml`)

Optional profile-specific configuration files that override the default settings:

#### Development Profile (`elektra-dev.toml`)

```toml
# Override account settings for development
[account]
rpc_url = "http://localhost:5050/"
account_address = "0x1234324234"
private_key = "0x1800000000300000180000000000030000000000003006001800006600"

# Development-specific contracts
[contracts.testnet_oracle]
address = "0x1234324234"

# Development deploy overrides
[deploys.foo]
salt = "0xdev123"  # Different salt for dev

# Development-specific multicall sequence
[[call]]
call_type = "deploy"
sequence = -1
class = "foo"
inputs = ["Dev Token", "DEV", 18, 1000]
salt = "0xdev123"
id = "dev_token"
```

#### Testnet Profile (`elektra-testnet.toml`)

```toml
# Testnet configuration overrides
[account]
rpc_url = "$TESTNET_RPC_URL"
account_address = "$TESTNET_ACCOUNT_ADDRESS"
keystore_path = "~/.elektra_accounts/testnet_keystore.json"

# Testnet-specific class references
[classes]
erc20 = "0xtestnet_erc20_class_hash"

# Testnet contract addresses
[contracts.oracle]
address = "0xtestnet_oracle_address"

# Testnet-specific multicall for setup
[[call]]
call_type = "invoke"
contract_address = "$contracts:oracle:address"
function = "set_admin"
inputs = ["$account:account_address"]
```

#### Release Profile (`elektra-release.toml`)

```toml
# Production configuration overrides
[account]
rpc_url = "$MAINNET_RPC_URL"
account_address = "$MAINNET_ACCOUNT_ADDRESS"
keystore_path = "~/.elektra_accounts/mainnet_keystore.json"

# Production class hashes
[classes]
erc20 = "0xmainnet_verified_erc20_class_hash"

# Production contract addresses
[contracts.oracle]
address = "0xmainnet_oracle_address"
metadata = { verified = true, audit_date = "2025-01-01" }

# Production-specific multicall for contract initialization
[[call]]
call_type = "invoke"
contract_address = "$contracts:oracle:address"
function = "set_production_mode"
inputs = [true]

[[call]]
call_type = "invoke"
contract_address = "$contracts:oracle:address"
function = "transfer_ownership"
inputs = ["$env:MAINNET_ADMIN_ADDRESS"]
```

## Key Features

### 1. **Configuration File Hierarchy**

- **`elektra.toml`** - Default configuration with base settings
- **`elektra-<profile>.toml`** - Profile-specific overrides (e.g., `elektra-dev.toml`, `elektra-testnet.toml`, `elektra-mainnet.toml`)
- **Configuration merging** - Profile files override default settings
- **Environment variable support** for sensitive data

### 2. **Multiple Reference Formats**

The system supports flexible ways to reference classes and contracts:

```toml
# Simple hash reference
[classes]
erc20 = "0x123..."

# Object format with metadata
[classes.erc721]
hash = "0xabc..."
metadata = { source = "openzeppelin", version = "0.8.0" }

# Contract addresses
[contracts]
simple_contract = "0x456..."

[contracts.complex_contract]
address = "0x789..."
metadata = { type = "oracle", version = "1.2.0" }
```

### 3. **Variable Interpolation System**

Powerful variable system allowing references between sections:

- `$account:account_address` - Reference account data
- `$classes:erc20` - Reference class hash by tag
- `$contracts:pragma:address` - Reference contract address
- `$deploys:foo:address` - Reference deployed contract address
- `$dojo:models:my_model:class_hash` - Plugin-specific variables
- `$env:ENV_VARIABLE` - Environment variables

### 4. **Plugin Architecture**

Extensible plugin system for specialized deployment patterns:

```toml
[plugins.dojo]
enabled = true
world_address = "$contracts:world:address"
models = ["Position", "Player", "Game"]

[plugins.permissions]
enabled = true
admin_role = "$account:account_address"
operators = ["$contracts:operator1:address"]
```

### 5. **State Tracking & Manifests**

Auto-generated manifest files track deployment state:

```json
// contracts/deployments/dev-manifest.json
{
  "version": "1.0.0",
  "profile": "dev",
  "deployments": {
    "game_token": {
      "address": "0x...",
      "class_hash": "0x...",
      "transaction_hash": "0x...",
      "deployed_at": "2025-01-15T10:35:00Z",
      "constructor_calldata": ["My Game Token", "MGT", 18, 1000000000]
    }
  }
}
```

### 6. **Multicall Support**

Execute complex deployments atomically in a single transaction using `[[call]]` sections:

- Declare multiple contracts
- Deploy with cross-references
- Initialize contracts
- Set up permissions
- All operations defined directly in config files
- Profile-specific multicall sequences

## Configuration Loading Priority

The system loads configuration with the following priority (highest to lowest):

1. **Command line arguments** (highest priority)
2. **Environment variables** (`$env:VAR_NAME` in config files)
3. **Profile-specific config** (`elektra-<profile>.toml`)
4. **Default config** (`elektra.toml`)

Example configuration loading:

```bash
# Uses elektra.toml as base
elektra deploy

# Merges elektra-dev.toml over elektra.toml
elektra deploy --profile dev

# Override with command line
elektra deploy --profile dev --account custom-account
```

## Usage Examples

### CLI Commands

```bash
# Use default configuration
elektra deploy

# Use profile-specific configuration
elektra deploy --profile dev
elektra deploy --profile testnet
elektra deploy --profile mainnet

# Individual operations with profiles
elektra declare --class erc20 --profile dev
elektra deploy --tag game_token --profile testnet

# Multicall deployment with profiles
elektra multicall run --profile dev

# State inspection by profile
elektra show deployments --profile dev
elektra show classes --profile testnet
elektra manifest export --profile mainnet
```

### File Structure

```
project/
├── elektra.toml              # Default configuration with multicalls
├── elektra-dev.toml          # Development profile overrides
├── elektra-testnet.toml      # Testnet profile overrides
├── elektra-mainnet.toml      # Mainnet profile overrides
└── contracts/
    ├── declares/             # Declaration manifests
    │   ├── default-manifest.json
    │   ├── dev-manifest.json
    │   ├── testnet-manifest.json
    │   └── mainnet-manifest.json
    └── deployments/          # Deployment manifests
        ├── default-manifest.json
        ├── dev-manifest.json
        ├── testnet-manifest.json
        └── mainnet-manifest.json
```

## Security Best Practices

1. **Environment Variables**: Sensitive data like private keys use env vars (`$env:PRIVATE_KEY`)
2. **Keystore Support**: Compatible with encrypted keystore files
3. **Profile Separation**: Different profiles prevent accidental cross-network deployments
4. **Configuration Override Priority**: Command line > env vars > profile config > default config

## Configuration Merging Examples

### Example 1: Development Override

```bash
# elektra.toml (default)
[account]
account_address = "0xdefault..."
rpc_url = "http://localhost:5050/"

# elektra-dev.toml (override)
[account]
account_address = "0xdev..."
# rpc_url inherited from default

# Result when using --profile dev:
# account_address = "0xdev..."
# rpc_url = "http://localhost:5050/"
```

### Example 2: Testnet with Environment Variables

```bash
# elektra.toml (default)
[account]
private_key = "$env:DEV_PRIVATE_KEY"

# elektra-testnet.toml (override)
[account]
private_key = "$env:TESTNET_PRIVATE_KEY"
rpc_url = "$env:TESTNET_RPC_URL"

# Environment variables:
export TESTNET_PRIVATE_KEY="0x..."
export TESTNET_RPC_URL="https://..."

# Result when using --profile testnet:
# All values resolved from environment
```

It should also include the use of variables (e.g., `$dojo:models:my_model:class_hash`) and allow for plugins to be included that interact with defined interfaces such as permissions or token management.

Separate plugins can be created for Dojo-style deployment allowing for easy management and upgrading of contracts.
