# WhackRockFundRegistry Overview

## Introduction

The WhackRockFundRegistry is the core factory contract of the WHACKROCK protocol. It serves as the central deployment and configuration hub for all WhackRock investment funds.

## Key Responsibilities

### 1. Fund Deployment
The registry acts as a factory, deploying standardized WhackRockFund contracts with consistent parameters and security guarantees.

### 2. Token Curation
Maintains a global allowlist of approved tokens that funds can hold, ensuring only vetted assets are included in portfolios.

### 3. Fee Management
Sets protocol-wide fee limits and collection addresses, preventing excessive fees while ensuring sustainable protocol development.

### 4. Access Control
Implements role-based permissions for protocol administration while maintaining decentralization principles.

## Contract Details

- **License**: BUSL-1.1
- **Solidity Version**: ^0.8.20
- **Upgradeability**: UUPS (Universal Upgradeable Proxy Standard)
- **Dependencies**: OpenZeppelin Contracts v5.0+

## Inheritance Structure

```solidity
contract WhackRockFundRegistry is 
    Initializable,           // Proxy initialization
    UUPSUpgradeable,         // Upgrade mechanism
    OwnableUpgradeable,      // Access control
    IWhackRockFundRegistry   // Interface implementation
```

## Core Components

### Aerodrome Integration
The registry integrates with Aerodrome DEX for:
- WETH address derivation
- Router configuration for funds
- Ensuring DEX compatibility

### Fund Tracking
- Maintains array of all deployed funds
- Maps funds to their creators
- Tracks total fund count
- Ensures symbol uniqueness

### Parameter Management
- Configurable creation fees
- Maximum AUM fee limits
- Protocol fee recipients
- Token allowlist limits

## Design Philosophy

The registry follows these principles:

1. **Standardization**: All funds follow the same deployment pattern
2. **Safety**: Only approved tokens can be included in funds
3. **Flexibility**: Parameters can be adjusted by governance
4. **Transparency**: All configurations are publicly readable

## Upgrade Considerations

{% hint style="warning" %}
The registry uses UUPS upgradeability. Only the owner can authorize upgrades, and careful consideration must be given to maintaining state compatibility.
{% endhint %}

### Upgrade Process
1. Deploy new implementation
2. Call `upgradeTo()` or `upgradeToAndCall()`
3. Verify state preservation
4. Update documentation

### Upgrade Restrictions
- Only owner can upgrade
- Implementation must be UUPS-compatible
- State layout must be preserved
- Extensive testing required

## Integration Points

### For Fund Creators
- Call `createWhackRockFund()` with parameters
- Pay creation fee in USDC
- Receive deployed fund address

### For Protocol Admins
- Manage token allowlist
- Update fee parameters
- Monitor fund deployments

### For External Protocols
- Query deployed funds
- Verify fund authenticity
- Read protocol parameters

## Next Steps

- [State Variables](state-variables.md) - Detailed variable documentation
- [Functions](functions.md) - Complete function reference
- [Events](events.md) - Event definitions and usage