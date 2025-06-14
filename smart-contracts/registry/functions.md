# Registry Functions

## Overview

This page documents all functions in the WhackRockFundRegistry contract, organized by their purpose and access control.

## Initialization

### initialize()

```solidity
function initialize(
    address _initialOwner,
    address _aerodromeRouterAddress,
    uint256 _maxInitialFundTokensLength,
    address _usdcTokenAddress,
    address _whackRockRewardsAddr,
    uint256 _protocolCreationFeeUsdc,
    uint256 _totalAumFeeBps,
    address _protocolAumRecipient,
    uint256 _maxAgentDepositFeeBpsAllowed
) public initializer
```

**Purpose**: Initializes the registry proxy with all required parameters

**Access**: Can only be called once during deployment

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `_initialOwner` | `address` | Address that will own the registry |
| `_aerodromeRouterAddress` | `address` | Aerodrome DEX router for fund operations |
| `_maxInitialFundTokensLength` | `uint256` | Maximum tokens allowed in new funds |
| `_usdcTokenAddress` | `address` | USDC token for fee collection |
| `_whackRockRewardsAddr` | `address` | Recipient of creation fees |
| `_protocolCreationFeeUsdc` | `uint256` | Fee amount for fund creation |
| `_totalAumFeeBps` | `uint256` | Maximum AUM fee (basis points) |
| `_protocolAumRecipient` | `address` | Protocol's AUM fee recipient |
| `_maxAgentDepositFeeBpsAllowed` | `uint256` | Maximum deposit fee allowed |

**Requirements**:
- Must not be already initialized
- Router address cannot be zero
- Max tokens must be > 0
- USDC address cannot be zero
- Rewards address cannot be zero
- Protocol AUM recipient cannot be zero

**Events Emitted**:
- `RegistryParamsUpdated`

## Fund Creation

### createWhackRockFund()

```solidity
function createWhackRockFund(
    address _initialAgent,
    address[] memory _fundAllowedTokens,
    uint256[] memory _initialTargetWeights,
    string memory _vaultName,
    string memory _vaultSymbol,
    string memory _vaultURI,
    string memory _description,
    address _agentAumFeeWalletForFund,
    uint256 _agentSetTotalAumFeeBps
) external returns (address fundAddress)
```

**Purpose**: Creates a new WhackRockFund with specified parameters

**Access**: Public - anyone can create a fund

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `_initialAgent` | `address` | Agent who will manage the fund |
| `_fundAllowedTokens` | `address[]` | Tokens the fund can hold |
| `_initialTargetWeights` | `uint256[]` | Target weights (must sum to 10000) |
| `_vaultName` | `string` | ERC20 name for fund shares |
| `_vaultSymbol` | `string` | ERC20 symbol (must be unique) |
| `_vaultURI` | `string` | Metadata URI |
| `_description` | `string` | Fund description |
| `_agentAumFeeWalletForFund` | `address` | Agent's fee recipient |
| `_agentSetTotalAumFeeBps` | `uint256` | Total AUM fee rate |

**Requirements**:
- At least one token
- Token count ≤ `maxInitialAllowedTokensLength`
- Symbol must not be empty or already taken
- All tokens must be in allowlist
- Agent fee wallet cannot be zero
- AUM fee ≤ protocol maximum
- Caller must have sufficient USDC for creation fee

**Process**:
1. Validates all parameters
2. Collects creation fee in USDC (if set)
3. Deploys new WhackRockFund contract
4. Records fund in registry
5. Marks symbol as taken

**Returns**: Address of the newly created fund

**Events Emitted**:
- `WhackRockFundCreated`

**Example**:
```solidity
address[] memory tokens = [DAI, USDC, WBTC];
uint256[] memory weights = [3000, 4000, 3000]; // 30%, 40%, 30%

address newFund = registry.createWhackRockFund(
    agentAddress,
    tokens,
    weights,
    "AI Growth Fund",
    "AIGF",
    "ipfs://metadata",
    "AI-managed growth strategy",
    agentFeeWallet,
    200 // 2% annual fee
);
```

## Token Allowlist Management

### addRegistryAllowedToken()

```solidity
function addRegistryAllowedToken(address _token) external onlyOwner
```

**Purpose**: Adds a single token to the global allowlist

**Access**: Owner only

**Parameters**:
- `_token`: Token address to add

**Requirements**:
- Token not already allowed
- Token cannot be address(0)
- Token cannot be WETH

**Events Emitted**:
- `RegistryAllowedTokenAdded`

### batchAddRegistryAllowedToken()

```solidity
function batchAddRegistryAllowedToken(address[] memory _tokens) external onlyOwner
```

**Purpose**: Adds multiple tokens to the allowlist in one transaction

**Access**: Owner only

**Parameters**:
- `_tokens`: Array of token addresses to add

**Requirements**:
- Same as `addRegistryAllowedToken` for each token

**Events Emitted**:
- `RegistryAllowedTokenAdded` (for each token)

### removeRegistryAllowedToken()

```solidity
function removeRegistryAllowedToken(address _token) external onlyOwner
```

**Purpose**: Removes a token from the allowlist

**Access**: Owner only

**Parameters**:
- `_token`: Token address to remove

**Requirements**:
- Token must be in allowlist

**Process**:
- Removes token from mapping
- Removes from array (replaces with last element)

**Events Emitted**:
- `RegistryAllowedTokenRemoved`

{% hint style="info" %}
Removing a token doesn't affect existing funds that already hold it
{% endhint %}

## Parameter Management

### setMaxInitialAllowedTokensLength()

```solidity
function setMaxInitialAllowedTokensLength(uint256 _newMaxLength) external onlyOwner
```

**Purpose**: Updates the maximum number of tokens new funds can hold

**Access**: Owner only

**Parameters**:
- `_newMaxLength`: New maximum token count

**Requirements**:
- Must be > 0

**Events Emitted**:
- `MaxInitialAllowedTokensLengthUpdated`

### updateRegistryParameters()

```solidity
function updateRegistryParameters(
    address _usdcTokenAddress,
    address _whackRockRewardsAddress,
    uint256 _protocolFundCreationFeeUsdc,
    uint256 _totalAumFeeBps,
    address _protocolAumRecipient,
    uint256 _newMaxAgentDepositFeeBps
) external onlyOwner
```

**Purpose**: Updates multiple registry parameters at once

**Access**: Owner only

**Parameters**: Same as initialization parameters

**Requirements**:
- USDC address cannot be zero
- Rewards address cannot be zero
- Protocol AUM recipient cannot be zero

**Events Emitted**:
- `RegistryParamsUpdated`

## View Functions

### getDeployedFundsCount()

```solidity
function getDeployedFundsCount() external view returns (uint256)
```

**Purpose**: Returns the total number of funds created

**Returns**: Length of `deployedFunds` array

### getFundAddressByIndex()

```solidity
function getFundAddressByIndex(uint256 _index) external view returns (address)
```

**Purpose**: Returns a fund address by its index

**Parameters**:
- `_index`: Index in the deployed funds array

**Requirements**:
- Index must be < deployed funds count

**Returns**: Fund contract address

### getRegistryAllowedTokens()

```solidity
function getRegistryAllowedTokens() external view returns (address[] memory)
```

**Purpose**: Returns the complete allowlist of tokens

**Returns**: Array of all allowed token addresses

**Use Case**: Useful for UIs to show available tokens

## Internal Functions

### _authorizeUpgrade()

```solidity
function _authorizeUpgrade(address newImplementation) internal override onlyOwner
```

**Purpose**: Authorizes upgrade to new implementation

**Access**: Internal, called by UUPS upgrade mechanism

**Requirements**:
- Caller must be owner

### _addRegistryAllowedToken()

```solidity
function _addRegistryAllowedToken(address _token) internal
```

**Purpose**: Internal logic for adding tokens to allowlist

**Called By**: 
- `addRegistryAllowedToken()`
- `batchAddRegistryAllowedToken()`

## Access Control Summary

| Function | Access Level |
|----------|--------------|
| `initialize` | Once only |
| `createWhackRockFund` | Public |
| `addRegistryAllowedToken` | Owner |
| `batchAddRegistryAllowedToken` | Owner |
| `removeRegistryAllowedToken` | Owner |
| `setMaxInitialAllowedTokensLength` | Owner |
| `updateRegistryParameters` | Owner |
| All view functions | Public |

## Gas Optimization Tips

1. **Batch Operations**: Use `batchAddRegistryAllowedToken` for multiple tokens
2. **Parameter Updates**: Use `updateRegistryParameters` to update multiple values
3. **Token Validation**: Check allowlist before calling `createWhackRockFund`

## Common Integration Patterns

### Creating a Fund
```solidity
// 1. Check allowlist
address[] memory allowedTokens = registry.getRegistryAllowedTokens();

// 2. Approve creation fee
uint256 fee = registry.protocolFundCreationFeeUsdcAmount();
IERC20(registry.USDC_TOKEN()).approve(address(registry), fee);

// 3. Create fund
address fund = registry.createWhackRockFund(...);
```

### Querying Funds
```solidity
// Get all funds
uint256 count = registry.getDeployedFundsCount();
for (uint256 i = 0; i < count; i++) {
    address fund = registry.getFundAddressByIndex(i);
    // Process fund...
}
```