# Audit Notes

## Overview

This document provides guidance for security auditors reviewing the WHACKROCK protocol. It highlights critical areas requiring special attention, potential vulnerabilities, and security assumptions.

## Critical Security Areas

### 1. Share Price Calculation

#### Location
`WhackRockFund.deposit()` function

#### Code Snippet
```solidity
if (totalSupplyBeforeDeposit == 0) {
    sharesMinted = amountWETHToDeposit;
    if (sharesMinted < MINIMUM_SHARES_LIQUIDITY && amountWETHToDeposit > 0) {
        sharesMinted = MINIMUM_SHARES_LIQUIDITY;
    }
} else {
    if (navBeforeDeposit == 0) revert E5();
    sharesMinted = (amountWETHToDeposit * totalSupplyBeforeDeposit) / navBeforeDeposit;
}
```

#### Audit Focus
- **First Deposit Manipulation**: Verify minimum deposit requirements prevent share price attacks
- **Integer Division**: Check for rounding errors in share calculations
- **Zero NAV Handling**: Ensure proper handling when NAV becomes zero
- **Overflow Protection**: Verify calculations don't overflow with large numbers

#### Potential Issues
- Share price could be manipulated if minimum deposit is too low
- Rounding errors could be exploited for profit
- Edge cases around zero values need careful handling

### 2. Rebalancing Logic

#### Location
`WhackRockFund._rebalance()` internal function

#### Code Snippet
```solidity
function _rebalance() internal {
    // 1. Calculate current asset values
    // 2. Determine required swaps
    // 3. Execute swaps via DEX router
    // 4. Emit events
}
```

#### Audit Focus
- **Slippage Protection**: Verify all swaps include adequate slippage limits
- **Sandwich Attack Prevention**: Check if rebalancing can be front-run profitably
- **Atomic Operations**: Ensure rebalancing is atomic or safely handles partial failures
- **Gas Limits**: Verify rebalancing can complete within block gas limits

#### Potential Issues
- Complex rebalancing logic could fail with many tokens
- DEX integration could be exploited
- Slippage calculations might be incorrect

### 3. Fee Collection Mechanism

#### Location
`WhackRockFund.collectAgentManagementFee()` function

#### Code Snippet
```solidity
uint256 timeElapsed = block.timestamp - lastAgentAumFeeCollectionTimestamp;
uint256 totalFeeValueInAA = (navAtFeeCalc * agentAumFeeBps * timeElapsed) / (TOTAL_WEIGHT_BASIS_POINTS * 365 days);
uint256 totalSharesToMintForFee = (totalFeeValueInAA * sharesAtFeeCalc) / navAtFeeCalc;
```

#### Audit Focus
- **Fee Calculation**: Verify fee math is correct and can't be gamed
- **Timestamp Manipulation**: Check if block timestamp can be exploited
- **Division by Zero**: Ensure safe handling when NAV or shares are zero
- **Fee Limits**: Verify fees can't exceed intended limits

#### Potential Issues
- Time-based calculations could be manipulated
- Minting shares affects NAV calculations
- Integer arithmetic could have precision issues

### 4. Access Control

#### Location
Multiple contracts and functions

#### Key Patterns
```solidity
modifier onlyOwner() { _checkOwner(); _; }
modifier onlyAgent() { if (msg.sender != agent) revert E4(); _; }
```

#### Audit Focus
- **Role Separation**: Verify owners and agents have appropriate permissions
- **Permission Checks**: Ensure all sensitive functions are protected
- **Transfer Functions**: Check ownership and agent transfer mechanisms
- **Emergency Functions**: Verify emergency controls work as intended

#### Potential Issues
- Missing access control on critical functions
- Privilege escalation possibilities
- Emergency functions could be abused

### 5. External Integrations

#### Location
DEX router interactions in rebalancing

#### Code Snippet
```solidity
router.swapExactTokensForTokens(
    amountIn,
    minAmountOut,
    path,
    address(this),
    deadline
);
```

#### Audit Focus
- **Router Trust**: Verify router contract is legitimate and secure
- **Path Validation**: Check if swap paths can be manipulated
- **Return Value Handling**: Ensure swap results are properly validated
- **Approval Management**: Verify token approvals are safe

#### Potential Issues
- Malicious router could steal funds
- Swap paths could be manipulated for profit
- Approval vulnerabilities

## Mathematical Security

### 1. Arithmetic Operations

#### Key Calculations
```solidity
// Share price calculation
sharePrice = nav * 1e18 / totalShares

// Fee calculation  
feeValue = (nav * feeRate * timeElapsed) / (10000 * 365 days)

// Weight calculations
currentWeight = (tokenValue * 10000) / totalNav
```

#### Audit Checklist
- [ ] All divisions check for zero denominators
- [ ] Multiplication operations check for overflow
- [ ] Precision loss in divisions is acceptable
- [ ] Rounding errors don't accumulate dangerously
- [ ] Constants are correct (365 days, basis points)

### 2. Basis Points Handling

#### Usage Throughout Protocol
```solidity
uint256 public constant TOTAL_WEIGHT_BASIS_POINTS = 10000; // 100%
uint256 public constant DEFAULT_SLIPPAGE_BPS = 50;         // 0.5%
uint256 public constant REBALANCE_DEVIATION_THRESHOLD_BPS = 100; // 1%
```

#### Audit Focus
- Verify all basis point calculations use consistent denominators
- Check for off-by-one errors in percentage calculations
- Ensure basis points never exceed 10000 where inappropriate

## Upgrade Security

### 1. Registry Upgradeability

#### Pattern
UUPS (Universal Upgradeable Proxy Standard)

#### Location
`WhackRockFundRegistry.sol`

#### Audit Focus
- **Authorization**: Only owner can authorize upgrades
- **Storage Layout**: New implementations preserve storage layout
- **Initialization**: Upgrades handle initialization correctly
- **Rollback Capability**: Consider if rollback mechanisms are needed

#### Code to Review
```solidity
function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
```

### 2. Fund Immutability

#### Design Decision
Funds are non-upgradeable for investor protection

#### Audit Focus
- Verify funds truly cannot be upgraded
- Check if any proxy patterns exist in fund contracts
- Ensure critical parameters are actually immutable

## Economic Attack Vectors

### 1. MEV Opportunities

#### Potential Attacks
- Front-running rebalancing transactions
- Sandwich attacks on large swaps
- JIT (Just-In-Time) liquidity attacks

#### Audit Questions
- Can rebalancing be predicted and front-run profitably?
- Are slippage protections sufficient?
- Could MEV bots extract value from fund operations?

### 2. Market Manipulation

#### Scenarios
- Pump asset prices before fund rebalancing
- Manipulate low-liquidity tokens in fund portfolios
- Coordinate attacks across multiple funds

#### Defensive Measures to Verify
- Slippage limits prevent excessive losses
- Rebalancing thresholds reduce unnecessary trading
- Token allowlist prevents inclusion of manipulatable assets

## Gas and Performance

### 1. Gas Consumption

#### High-Cost Operations
- Fund creation with many tokens
- Rebalancing with complex swap paths
- Batch token operations

#### Audit Focus
- Verify operations complete within block gas limits
- Check for gas griefing opportunities
- Ensure loops are bounded appropriately

### 2. DoS Vulnerabilities

#### Potential Vectors
- Reverting tokens in fund portfolios
- Extremely high gas price attacks
- Block gas limit attacks on rebalancing

#### Code to Review
```solidity
// Token transfers in withdrawal
for (uint256 i = 0; i < allowedTokens.length; i++) {
    // Could this loop fail with many tokens?
}
```

## Integration Security

### 1. External Dependencies

#### Key Dependencies
- OpenZeppelin contracts
- Aerodrome DEX router
- ERC20 tokens in allowlist

#### Audit Questions
- Are OpenZeppelin contracts the correct versions?
- Is the DEX router contract legitimate and audited?
- Could malicious ERC20 tokens break fund operations?

### 2. Oracle Dependencies

#### Current Design
Uses DEX router for price quotes (no external oracles)

#### Audit Focus
- Could DEX prices be manipulated?
- Are there single points of failure in price discovery?
- How does the system handle illiquid markets?

## Event Security

### 1. Event Completeness

#### Critical Events
```solidity
event WhackRockFundCreated(...)
event WETHDepositedAndSharesMinted(...)
event BasketAssetsWithdrawn(...)
event RebalanceCycleExecuted(...)
```

#### Audit Checklist
- [ ] All state changes emit appropriate events
- [ ] Events contain all necessary information
- [ ] Events cannot be spoofed or manipulated
- [ ] Sensitive information is not leaked in events

### 2. Event Ordering

#### Audit Focus
- Events are emitted after state changes (CEI pattern)
- Multiple events in single transaction are ordered correctly
- External systems can reliably process events

## Test Coverage Requirements

### 1. Critical Path Testing

#### Must Test
- First deposit edge cases
- Share price manipulation attempts
- Rebalancing with various token combinations
- Fee collection edge cases
- Access control boundaries

### 2. Fuzzing Recommendations

#### Areas for Fuzzing
- Share price calculations with random inputs
- Weight distributions and rebalancing
- Fee calculations with extreme time periods
- Token amounts and precision handling

### 3. Integration Testing

#### External Systems
- Test with various ERC20 tokens
- Test with different DEX conditions
- Test with extreme market conditions

## Deployment Security

### 1. Initialization

#### Registry Initialization
```solidity
function initialize(
    address _initialOwner,
    address _aerodromeRouterAddress,
    // ... other parameters
) public initializer
```

#### Audit Checklist
- [ ] All parameters are validated
- [ ] Initialization cannot be called twice
- [ ] Initial state is secure
- [ ] Owner is set correctly

### 2. Constructor Parameters

#### Fund Constructor
```solidity
constructor(
    address _initialOwner,
    address _initialAgent,
    // ... many parameters
)
```

#### Audit Focus
- Verify all address parameters are validated
- Check array length validations
- Ensure immutable variables are set correctly

## Recommendations for Auditors

### 1. Focus Areas by Priority

#### High Priority
1. Share price calculation logic
2. Fee collection mathematics
3. Access control implementation
4. Rebalancing atomicity

#### Medium Priority
1. Event emission completeness
2. Gas optimization and limits
3. External integration security
4. Edge case handling

#### Lower Priority
1. Code style and organization
2. Non-critical optimizations
3. Documentation completeness

### 2. Testing Methodology

#### Recommended Approach
1. **Static Analysis**: Use tools like Slither, Mythril
2. **Unit Testing**: Achieve >95% code coverage
3. **Integration Testing**: Test with mainnet fork
4. **Fuzzing**: Focus on mathematical operations
5. **Manual Review**: Deep dive on critical functions

### 3. Common Pitfalls

#### Watch Out For
- Integer overflow/underflow in calculations
- Reentrancy in external calls
- Front-running opportunities
- Access control bypasses
- Initialization vulnerabilities

## Security Assumptions

### 1. Trust Model

#### Trusted Entities
- Registry owner (expected to be governance/multisig)
- Fund owners (set agent, emergency functions)
- DEX router contract

#### Untrusted Entities
- Fund agents (limited permissions)
- Fund investors
- ERC20 token contracts (partially trusted via allowlist)

### 2. Economic Assumptions

#### Key Assumptions
- DEX has sufficient liquidity for rebalancing
- Gas costs remain economically reasonable
- No coordinated attacks across multiple funds
- Token allowlist prevents malicious tokens

### 3. Technical Assumptions

#### Infrastructure
- Blockchain remains operational and secure
- Block timestamps are reasonably accurate
- Gas limits don't change dramatically
- Ethereum 1559 fee market functions correctly

## Conclusion

This document provides a comprehensive guide for auditing the WHACKROCK protocol. Auditors should pay special attention to the mathematical operations, access control mechanisms, and external integrations. The protocol's design emphasizes security through simplicity, but complex financial logic still requires careful review.

## Related Documentation

- [Access Control](access-control.md) - Detailed permission analysis
- [Economic Security](economic-security.md) - Attack vector analysis
- [Best Practices](best-practices.md) - Security recommendations