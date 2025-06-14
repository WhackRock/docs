# Error Codes Reference

## Overview

This page provides a comprehensive reference for all error codes, revert messages, and error handling patterns used in the WHACKROCK protocol.

## Custom Errors

WHACKROCK uses custom errors for gas efficiency and better error handling.

### WhackRockFund Errors

#### E1() - Zero Address
```solidity
error E1();
```

**Description**: Thrown when a zero address is provided where a valid address is required

**Common Causes**:
- Passing `address(0)` for receiver in deposits
- Setting agent to zero address
- Invalid fee wallet addresses

**Example Scenarios**:
```solidity
// This will revert with E1()
fund.deposit(amount, address(0)); // Zero receiver

// This will revert with E1()
fund.setAgent(address(0)); // Zero agent
```

**Resolution**: Provide a valid, non-zero address

#### E2() - Invalid Amount/Length
```solidity
error E2();
```

**Description**: Thrown for invalid amounts, array lengths, or parameter values

**Common Causes**:
- Deposit amount below minimum (0.01 WETH)
- Array length mismatches (tokens vs weights)
- Zero weights in target allocation
- Weights not summing to 10000

**Example Scenarios**:
```solidity
// This will revert with E2()
fund.deposit(ethers.parseEther("0.005"), receiver); // Below minimum

// This will revert with E2()
fund.setTargetWeights([5000, 6000]); // Sum = 11000, not 10000

// This will revert with E2()
fund.setTargetWeights([5000, 0, 5000]); // Zero weight not allowed
```

**Resolution**: 
- Use minimum deposit amounts
- Ensure arrays have matching lengths
- Ensure weights sum to exactly 10000
- Avoid zero weights

#### E3() - Insufficient Balance
```solidity
error E3();
```

**Description**: Thrown when trying to withdraw more shares than owned

**Common Causes**:
- Attempting to burn more shares than balance
- Insufficient token balance for operations

**Example Scenarios**:
```solidity
// User has 100 shares but tries to withdraw 200
fund.withdraw(ethers.parseEther("200"), receiver, owner); // E3()
```

**Resolution**: Check balance before attempting withdrawal

#### E4() - Unauthorized (Not Agent)
```solidity
error E4();
```

**Description**: Thrown when non-agent tries to call agent-only functions

**Common Causes**:
- Wrong wallet trying to set weights
- Agent permissions not properly configured
- Agent address changed but caller using old address

**Example Scenarios**:
```solidity
// Non-agent trying to set weights
fund.setTargetWeights([5000, 5000]); // E4() if not agent
```

**Resolution**: Ensure caller is the current fund agent

#### E5() - Invalid State
```solidity
error E5();
```

**Description**: Thrown when contract is in an invalid state for the operation

**Common Causes**:
- Zero AUM fee rate when trying to collect fees
- Zero NAV when calculating shares
- Timestamp issues in fee collection

**Example Scenarios**:
```solidity
// Trying to collect fees when fee rate is 0
fund.collectAgentManagementFee(); // E5() if agentAumFeeBps == 0
```

**Resolution**: Verify contract state meets operation requirements

#### E6() - Swap Failed
```solidity
error E6();
```

**Description**: Thrown when DEX swap operations fail during rebalancing

**Common Causes**:
- Insufficient liquidity for swap
- Slippage too high
- Invalid swap path
- DEX router issues

**Example Scenarios**:
- Large rebalance in illiquid market
- Slippage exceeds DEFAULT_SLIPPAGE_BPS (0.5%)

**Resolution**: 
- Wait for better market conditions
- Check token liquidity
- Verify DEX router functionality

## Registry Errors

### Require Statement Errors

The registry uses require statements with descriptive messages:

#### "Registry: Router zero"
```solidity
require(_aerodromeRouterAddress != address(0), "Registry: Router zero");
```
**Cause**: Zero address provided for Aerodrome router during initialization
**Resolution**: Provide valid router address

#### "Registry: Max fund tokens must be > 0"
```solidity
require(_maxInitialFundTokensLength > 0, "Registry: Max fund tokens must be > 0");
```
**Cause**: Zero or negative value for maximum tokens per fund
**Resolution**: Set positive value for token limit

#### "Registry: USDC address zero"
```solidity
require(_usdcTokenAddress != address(0), "Registry: USDC address zero");
```
**Cause**: Zero address provided for USDC token
**Resolution**: Provide valid USDC token address

#### "Registry: Token already allowed"
```solidity
require(!isTokenAllowedInRegistry[_token], "Registry: Token already allowed");
```
**Cause**: Attempting to add token that's already in allowlist
**Resolution**: Check allowlist before adding tokens

#### "Registry: Token not in list"
```solidity
require(isTokenAllowedInRegistry[_token], "Registry: Token not in list");
```
**Cause**: Attempting to remove token that's not in allowlist
**Resolution**: Verify token is in allowlist before removal

#### "Registry: Token zero"
```solidity
require(_token != address(0), "Registry: Token zero");
```
**Cause**: Zero address provided when adding token
**Resolution**: Provide valid token address

#### "Registry: WETH not allowed"
```solidity
require(_token != WETH_ADDRESS, "Registry: WETH not allowed");
```
**Cause**: Attempting to add WETH to allowlist (WETH handled separately)
**Resolution**: Don't include WETH in token allowlist

#### "Registry: Fund must have at least one token"
```solidity
require(_fundAllowedTokens.length > 0, "Registry: Fund must have at least one token");
```
**Cause**: Creating fund with empty token array
**Resolution**: Include at least one token in fund

#### "Registry: Exceeds max fund tokens"
```solidity
require(_fundAllowedTokens.length <= maxInitialAllowedTokensLength, "Registry: Exceeds max fund tokens");
```
**Cause**: Too many tokens in fund creation
**Resolution**: Reduce number of tokens or increase registry limit

#### "Registry: Vault symbol cannot be empty"
```solidity
require(bytes(_vaultSymbol).length > 0, "Registry: Vault symbol cannot be empty");
```
**Cause**: Empty string provided for fund symbol
**Resolution**: Provide non-empty symbol

#### "Registry: Vault symbol already taken"
```solidity
require(!isSymbolTaken[_vaultSymbol], "Registry: Vault symbol already taken");
```
**Cause**: Symbol already used by another fund
**Resolution**: Choose unique symbol

#### "Registry: Agent AUM fee wallet zero"
```solidity
require(_agentAumFeeWalletForFund != address(0), "Registry: Agent AUM fee wallet zero");
```
**Cause**: Zero address for agent fee recipient
**Resolution**: Provide valid fee wallet address

#### "Registry: Fund AUM fee exceeds protocol max/default"
```solidity
require(_agentSetTotalAumFeeBps <= totalAumFeeBpsForFunds, "Registry: Fund AUM fee exceeds protocol max/default");
```
**Cause**: Fund fee rate exceeds protocol maximum
**Resolution**: Reduce fee rate to within protocol limits

#### "Registry: Fund token not allowed"
```solidity
require(isTokenAllowedInRegistry[_fundAllowedTokens[i]], "Registry: Fund token not allowed");
```
**Cause**: Fund contains token not in registry allowlist
**Resolution**: Only use tokens from registry allowlist

#### "Registry: Insufficient USDC balance"
```solidity
require(USDC_TOKEN.balanceOf(msg.sender) >= protocolFundCreationFeeUsdcAmount, "Registry: Insufficient USDC balance");
```
**Cause**: Insufficient USDC for creation fee
**Resolution**: Ensure adequate USDC balance and approval

#### "Registry: Index out of bounds"
```solidity
require(_index < deployedFunds.length, "Registry: Index out of bounds");
```
**Cause**: Requesting fund at invalid index
**Resolution**: Use valid index within deployed funds array

#### "Registry: Max length must be > 0"
```solidity
require(_newMaxLength > 0, "Registry: Max length must be > 0");
```
**Cause**: Setting maximum token length to zero
**Resolution**: Use positive value for token limit

## Standard ERC20 Errors

### OpenZeppelin ERC20 Errors

#### ERC20InsufficientBalance
```solidity
error ERC20InsufficientBalance(address sender, uint256 balance, uint256 needed);
```
**Description**: Account doesn't have enough tokens for transfer
**Parameters**:
- `sender`: Account attempting transfer
- `balance`: Current balance
- `needed`: Amount attempted to transfer

#### ERC20InvalidSender
```solidity
error ERC20InvalidSender(address sender);
```
**Description**: Invalid sender address (usually zero address)

#### ERC20InvalidReceiver
```solidity
error ERC20InvalidReceiver(address receiver);
```
**Description**: Invalid receiver address (usually zero address)

#### ERC20InsufficientAllowance
```solidity
error ERC20InsufficientAllowance(address spender, uint256 allowance, uint256 needed);
```
**Description**: Spender doesn't have sufficient allowance
**Used In**: Fund withdrawals when caller != owner

#### ERC20InvalidApprover
```solidity
error ERC20InvalidApprover(address approver);
```
**Description**: Invalid approver address in approval operations

#### ERC20InvalidSpender
```solidity
error ERC20InvalidSpender(address spender);
```
**Description**: Invalid spender address in approval operations

## Error Handling Patterns

### JavaScript/TypeScript Error Handling

```javascript
class WhackRockErrorHandler {
    static parseError(error) {
        const errorMessage = error.message || error.toString();
        
        // Custom errors
        if (errorMessage.includes('E1')) {
            return {
                code: 'E1',
                type: 'ZERO_ADDRESS',
                message: 'Zero address provided where valid address required',
                userMessage: 'Invalid address. Please provide a valid Ethereum address.'
            };
        }
        
        if (errorMessage.includes('E2')) {
            return {
                code: 'E2',
                type: 'INVALID_AMOUNT',
                message: 'Invalid amount or array length',
                userMessage: 'Invalid input. Check amounts and ensure minimum requirements are met.'
            };
        }
        
        if (errorMessage.includes('E3')) {
            return {
                code: 'E3',
                type: 'INSUFFICIENT_BALANCE',
                message: 'Insufficient balance for operation',
                userMessage: 'Insufficient balance. You don\'t have enough shares for this withdrawal.'
            };
        }
        
        if (errorMessage.includes('E4')) {
            return {
                code: 'E4',
                type: 'UNAUTHORIZED',
                message: 'Not authorized as agent',
                userMessage: 'Unauthorized. Only the fund agent can perform this action.'
            };
        }
        
        if (errorMessage.includes('E5')) {
            return {
                code: 'E5',
                type: 'INVALID_STATE',
                message: 'Contract in invalid state',
                userMessage: 'Operation not available. Contract state doesn\'t allow this action.'
            };
        }
        
        if (errorMessage.includes('E6')) {
            return {
                code: 'E6',
                type: 'SWAP_FAILED',
                message: 'DEX swap operation failed',
                userMessage: 'Rebalancing failed. This may be due to low liquidity or high slippage.'
            };
        }
        
        // Registry errors
        if (errorMessage.includes('Registry: Vault symbol already taken')) {
            return {
                code: 'SYMBOL_TAKEN',
                type: 'VALIDATION_ERROR',
                message: 'Fund symbol already in use',
                userMessage: 'This symbol is already taken. Please choose a different symbol for your fund.'
            };
        }
        
        if (errorMessage.includes('Registry: Insufficient USDC balance')) {
            return {
                code: 'INSUFFICIENT_USDC',
                type: 'PAYMENT_ERROR',
                message: 'Insufficient USDC for creation fee',
                userMessage: 'Insufficient USDC balance. Please ensure you have enough USDC for the creation fee.'
            };
        }
        
        // ERC20 errors
        if (errorMessage.includes('ERC20InsufficientAllowance')) {
            return {
                code: 'INSUFFICIENT_ALLOWANCE',
                type: 'APPROVAL_ERROR',
                message: 'Insufficient token allowance',
                userMessage: 'Please approve the contract to spend your tokens before proceeding.'
            };
        }
        
        // Default
        return {
            code: 'UNKNOWN',
            type: 'UNKNOWN_ERROR',
            message: errorMessage,
            userMessage: 'An unexpected error occurred. Please try again or contact support.'
        };
    }
    
    static async handleTransaction(transactionPromise) {
        try {
            const tx = await transactionPromise;
            return await tx.wait();
        } catch (error) {
            const parsedError = this.parseError(error);
            throw new Error(parsedError.userMessage);
        }
    }
}

// Usage example
try {
    await fund.deposit(amount, receiver);
} catch (error) {
    const parsedError = WhackRockErrorHandler.parseError(error);
    console.error('Error:', parsedError.userMessage);
    
    // Handle specific error types
    switch (parsedError.type) {
        case 'INVALID_AMOUNT':
            // Show minimum deposit requirements
            break;
        case 'INSUFFICIENT_BALANCE':
            // Show current balance
            break;
        case 'UNAUTHORIZED':
            // Check agent permissions
            break;
        default:
            // Generic error handling
    }
}
```

### React Error Boundary

```jsx
import React from 'react';

class WhackRockErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }
    
    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }
    
    componentDidCatch(error, errorInfo) {
        const parsedError = WhackRockErrorHandler.parseError(error);
        console.error('WhackRock Error:', parsedError);
        
        // Log to error reporting service
        this.logErrorToService(parsedError, errorInfo);
    }
    
    render() {
        if (this.state.hasError) {
            const parsedError = WhackRockErrorHandler.parseError(this.state.error);
            
            return (
                <div className="error-boundary">
                    <h2>Something went wrong</h2>
                    <p>{parsedError.userMessage}</p>
                    <button onClick={() => this.setState({ hasError: false, error: null })}>
                        Try Again
                    </button>
                </div>
            );
        }
        
        return this.props.children;
    }
}
```

### Solidity Error Handling Best Practices

```solidity
// Good: Use custom errors for gas efficiency
error InvalidInput(uint256 provided, uint256 required);

function someFunction(uint256 amount) external {
    if (amount < MINIMUM_AMOUNT) {
        revert InvalidInput(amount, MINIMUM_AMOUNT);
    }
}

// Good: Descriptive require messages for complex conditions
require(
    tokens.length == weights.length && tokens.length > 0,
    "Arrays must have matching non-zero length"
);

// Good: Check effects interactions pattern prevents reentrancy
function deposit(uint256 amount) external {
    // Checks
    require(amount >= MINIMUM_DEPOSIT, "Amount too small");
    
    // Effects
    shares = calculateShares(amount);
    _mint(msg.sender, shares);
    
    // Interactions
    token.transferFrom(msg.sender, address(this), amount);
}
```

## Error Prevention

### Input Validation

```javascript
class InputValidator {
    static validateAddress(address) {
        if (!ethers.isAddress(address)) {
            throw new Error('Invalid Ethereum address format');
        }
        if (address === ethers.ZeroAddress) {
            throw new Error('Zero address not allowed');
        }
        return address.toLowerCase();
    }
    
    static validateWeights(weights) {
        if (!Array.isArray(weights) || weights.length === 0) {
            throw new Error('Weights must be non-empty array');
        }
        
        const total = weights.reduce((sum, weight) => {
            if (typeof weight !== 'number' || weight <= 0 || weight > 10000) {
                throw new Error('Each weight must be between 1 and 10000');
            }
            return sum + weight;
        }, 0);
        
        if (total !== 10000) {
            throw new Error(`Weights must sum to 10000, got ${total}`);
        }
        
        return weights;
    }
    
    static validateAmount(amount, minAmount = '0.01') {
        const parsed = ethers.parseEther(amount.toString());
        const min = ethers.parseEther(minAmount);
        
        if (parsed < min) {
            throw new Error(`Amount must be at least ${minAmount} ETH`);
        }
        
        return parsed;
    }
}
```

## Debugging Guide

### Common Error Scenarios

1. **Fund Creation Fails**
   - Check: Token allowlist inclusion
   - Check: Symbol uniqueness
   - Check: USDC balance and approval
   - Check: Weight sum equals 10000

2. **Deposit Fails**
   - Check: Minimum deposit amount (0.01 WETH)
   - Check: WETH balance and approval
   - Check: Valid receiver address

3. **Withdrawal Fails**
   - Check: Sufficient share balance
   - Check: Proper allowance if caller ≠ owner
   - Check: Valid addresses

4. **Agent Operations Fail**
   - Check: Caller is current agent
   - Check: Weight constraints
   - Check: Gas price for rebalancing

5. **Fee Collection Fails**
   - Check: Fee rate > 0
   - Check: Time elapsed since last collection
   - Check: Fund has positive NAV and shares

## Related Documentation

- [Integration Guide](../integration/quick-start.md) - Error handling in practice
- [Security Best Practices](../security/best-practices.md) - Preventing errors
- [Access Control](../security/access-control.md) - Permission-related errors