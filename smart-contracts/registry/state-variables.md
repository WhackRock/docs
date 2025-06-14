# Registry State Variables

## Overview

This page documents all state variables in the WhackRockFundRegistry contract, organized by their functional purpose.

## Core Components

### DEX Integration

```solidity
IAerodromeRouter public aerodromeRouter;
```
- **Type**: `IAerodromeRouter`
- **Visibility**: Public
- **Purpose**: Interface to Aerodrome DEX for fund operations
- **Set By**: `initialize()` function
- **Immutable After**: Initialization

```solidity
address public WETH_ADDRESS;
```
- **Type**: `address`
- **Visibility**: Public
- **Purpose**: Wrapped ETH address for accounting
- **Derived From**: `aerodromeRouter.weth()`
- **Immutable After**: Initialization

## Fee Configuration

### Protocol Fees

```solidity
IERC20 public USDC_TOKEN;
```
- **Type**: `IERC20`
- **Visibility**: Public
- **Purpose**: Token used for protocol fee payments
- **Updatable By**: `updateRegistryParameters()`

```solidity
address public whackRockRewardsAddress;
```
- **Type**: `address`
- **Visibility**: Public
- **Purpose**: Recipient of fund creation fees
- **Updatable By**: `updateRegistryParameters()`

```solidity
uint256 public protocolFundCreationFeeUsdcAmount;
```
- **Type**: `uint256`
- **Visibility**: Public
- **Purpose**: Amount of USDC required to create a fund
- **Updatable By**: `updateRegistryParameters()`
- **Typical Range**: 0 - 1000 USDC

### AUM Fee Configuration

```solidity
uint256 public totalAumFeeBpsForFunds;
```
- **Type**: `uint256`
- **Visibility**: Public
- **Purpose**: Maximum annual AUM fee funds can charge
- **Unit**: Basis points (100 = 1%)
- **Updatable By**: `updateRegistryParameters()`
- **Typical Range**: 0 - 500 (0% - 5%)

```solidity
address public protocolAumFeeRecipientForFunds;
```
- **Type**: `address`
- **Visibility**: Public
- **Purpose**: Address receiving protocol's share of AUM fees
- **Updatable By**: `updateRegistryParameters()`

```solidity
uint256 public maxAgentDepositFeeBps;
```
- **Type**: `uint256`
- **Visibility**: Public
- **Purpose**: Maximum deposit fee agents can charge
- **Unit**: Basis points
- **Updatable By**: `updateRegistryParameters()`
- **Note**: Currently unused in fund implementation

## Fund Tracking

### Deployment Records

```solidity
address[] public deployedFunds;
```
- **Type**: `address[]`
- **Visibility**: Public
- **Purpose**: Array of all fund addresses created by this registry
- **Growth**: Append-only via `createWhackRockFund()`
- **Access**: Via index using `getFundAddressByIndex()`

```solidity
mapping(address => address) public fundToCreator;
```
- **Type**: `mapping(address => address)`
- **Visibility**: Public
- **Purpose**: Maps fund address to creator address
- **Key**: Fund contract address
- **Value**: Creator's address
- **Set When**: Fund creation

```solidity
uint256 public fundCounter;
```
- **Type**: `uint256`
- **Visibility**: Public
- **Purpose**: Total number of funds created
- **Increment**: On each fund creation
- **Starting Value**: 0

## Token Management

### Allowlist Storage

```solidity
address[] public allowedTokensList;
```
- **Type**: `address[]`
- **Visibility**: Public
- **Purpose**: Array of approved tokens for funds
- **Management**: Add/remove by owner only
- **Restrictions**: Cannot include WETH or address(0)

```solidity
mapping(address => bool) public isTokenAllowedInRegistry;
```
- **Type**: `mapping(address => bool)`
- **Visibility**: Public
- **Purpose**: Quick lookup for token approval status
- **Key**: Token address
- **Value**: true if allowed, false otherwise

```solidity
uint256 public maxInitialAllowedTokensLength;
```
- **Type**: `uint256`
- **Visibility**: Public
- **Purpose**: Maximum tokens a fund can have at creation
- **Updatable By**: `setMaxInitialAllowedTokensLength()`
- **Typical Value**: 10

### Symbol Management

```solidity
mapping(string => bool) public isSymbolTaken;
```
- **Type**: `mapping(string => bool)`
- **Visibility**: Public
- **Purpose**: Ensures fund symbol uniqueness
- **Key**: ERC20 symbol string
- **Value**: true if already used
- **Set When**: Fund creation

## State Variable Interactions

### Initialization Flow
1. `aerodromeRouter` set → `WETH_ADDRESS` derived
2. Fee parameters configured
3. Initial allowlist populated

### Fund Creation Flow
1. Check `isTokenAllowedInRegistry` for all tokens
2. Verify symbol not in `isSymbolTaken`
3. Collect fee using `USDC_TOKEN`
4. Deploy fund with registry parameters
5. Update `deployedFunds`, `fundToCreator`, `fundCounter`
6. Mark symbol in `isSymbolTaken`

## Storage Layout Considerations

{% hint style="danger" %}
The registry is upgradeable. New state variables must be added at the end to preserve storage layout compatibility.
{% endhint %}

### Current Storage Slots
Storage slots are allocated in declaration order. Any upgrades must:
- Not reorder existing variables
- Not change existing variable types
- Only append new variables at the end
- Consider struct packing for gas optimization

## Getter Functions

All public state variables automatically generate getter functions:
- `aerodromeRouter()` → Returns router interface
- `WETH_ADDRESS()` → Returns WETH address
- `deployedFunds(uint256)` → Returns fund at index
- `allowedTokensList(uint256)` → Returns token at index
- etc.

## Related Documentation

- [Functions](functions.md) - How to modify state variables
- [Events](events.md) - Events emitted on state changes
- [Access Control](../security/access-control.md) - Who can modify what