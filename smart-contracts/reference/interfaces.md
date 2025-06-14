# Contract Interfaces

## Overview

This page documents the complete interfaces for WHACKROCK smart contracts, providing a comprehensive reference for developers integrating with the protocol.

## IWhackRockFundRegistry

### Interface Definition

```solidity
interface IWhackRockFundRegistry {
    // Events
    event WhackRockFundCreated(
        uint256 indexed fundId,
        address indexed fundAddress,
        address indexed creator,
        address initialAgent,
        string vaultName,
        string vaultSymbol,
        string vaultURI,
        string description,
        address[] allowedTokens,
        uint256[] targetWeights,
        address agentAumFeeWallet,
        uint256 agentTotalAumFeeBps,
        uint256 timestamp
    );
    
    event RegistryAllowedTokenAdded(address indexed token);
    event RegistryAllowedTokenRemoved(address indexed token);
    event MaxInitialAllowedTokensLengthUpdated(uint256 newLength);
    event RegistryParamsUpdated(
        address usdcTokenAddress,
        address whackRockRewardsAddr,
        uint256 protocolCreationFeeUsdc,
        uint256 totalAumFeeBps,
        address protocolAumRecipient,
        uint256 maxAgentDepositFeeBpsAllowed
    );

    // Token Allowlist Management
    function addRegistryAllowedToken(address _token) external;
    function batchAddRegistryAllowedToken(address[] memory _tokens) external;
    function removeRegistryAllowedToken(address _token) external;
    function setMaxInitialAllowedTokensLength(uint256 _newMaxLength) external;
    
    // Fund Creation
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
    ) external returns (address fundAddress);

    // View Functions
    function getDeployedFundsCount() external view returns (uint256);
    function getFundAddressByIndex(uint256 _index) external view returns (address);
    function getRegistryAllowedTokens() external view returns (address[] memory);
}
```

### Function Details

#### addRegistryAllowedToken()
```solidity
function addRegistryAllowedToken(address _token) external;
```
- **Access**: Owner only
- **Purpose**: Add single token to global allowlist
- **Restrictions**: Token cannot be address(0) or WETH
- **Events**: `RegistryAllowedTokenAdded`

#### batchAddRegistryAllowedToken()
```solidity
function batchAddRegistryAllowedToken(address[] memory _tokens) external;
```
- **Access**: Owner only
- **Purpose**: Add multiple tokens to allowlist efficiently
- **Gas Optimization**: More efficient than multiple single calls
- **Events**: `RegistryAllowedTokenAdded` (for each token)

#### removeRegistryAllowedToken()
```solidity
function removeRegistryAllowedToken(address _token) external;
```
- **Access**: Owner only
- **Purpose**: Remove token from allowlist
- **Implementation**: Uses swap-and-pop for gas efficiency
- **Events**: `RegistryAllowedTokenRemoved`

#### createWhackRockFund()
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
) external returns (address fundAddress);
```
- **Access**: Public
- **Purpose**: Deploy new fund contract
- **Returns**: Address of deployed fund
- **Fee**: May require USDC payment
- **Events**: `WhackRockFundCreated`

### Integration Example

```javascript
// TypeScript interface
interface IWhackRockFundRegistry {
    // Events
    filters: {
        WhackRockFundCreated(
            fundId?: BigNumberish | null,
            fundAddress?: string | null,
            creator?: string | null
        ): TypedEventFilter<[BigNumber, string, string, string, string, string, string, string, string[], BigNumber[], string, BigNumber, BigNumber]>;
        
        RegistryAllowedTokenAdded(token?: string | null): TypedEventFilter<[string]>;
        RegistryAllowedTokenRemoved(token?: string | null): TypedEventFilter<[string]>;
    };
    
    // Functions
    addRegistryAllowedToken(token: string, overrides?: Overrides): Promise<ContractTransaction>;
    removeRegistryAllowedToken(token: string, overrides?: Overrides): Promise<ContractTransaction>;
    
    createWhackRockFund(
        initialAgent: string,
        fundAllowedTokens: string[],
        initialTargetWeights: BigNumberish[],
        vaultName: string,
        vaultSymbol: string,
        vaultURI: string,
        description: string,
        agentAumFeeWallet: string,
        agentSetTotalAumFeeBps: BigNumberish,
        overrides?: Overrides
    ): Promise<ContractTransaction>;
    
    // View functions
    getDeployedFundsCount(overrides?: CallOverrides): Promise<BigNumber>;
    getFundAddressByIndex(index: BigNumberish, overrides?: CallOverrides): Promise<string>;
    getRegistryAllowedTokens(overrides?: CallOverrides): Promise<string[]>;
}
```

## IWhackRockFund

### Interface Definition

```solidity
interface IWhackRockFund {
    // Events
    event AgentUpdated(address indexed oldAgent, address indexed newAgent);
    
    event TargetWeightsUpdated(
        address indexed agent,
        address[] tokens,
        uint256[] weights,
        uint256 timestamp
    );
    
    event RebalanceCheck(
        bool needsRebalance,
        uint256 maxDeviationBPS,
        uint256 currentNAV_AA
    );
    
    event RebalanceCycleExecuted(
        uint256 navBeforeRebalanceAA,
        uint256 navAfterRebalanceAA,
        uint256 blockTimestamp,
        uint256 wethValueInUSDC
    );
    
    event FundTokenSwapped(
        address indexed tokenFrom,
        uint256 amountFrom,
        address indexed tokenTo,
        uint256 amountTo
    );
    
    event WETHDepositedAndSharesMinted(
        address indexed depositor,
        address indexed receiver,
        uint256 wethDeposited,
        uint256 sharesMinted,
        uint256 navBeforeDepositWETH,
        uint256 totalSupplyBeforeDeposit,
        uint256 wethValueInUSDC
    );
    
    event BasketAssetsWithdrawn(
        address indexed owner,
        address indexed receiver,
        uint256 sharesBurned,
        address[] tokensWithdrawn,
        uint256[] amountsWithdrawn,
        uint256 navBeforeWithdrawalWETH,
        uint256 totalSupplyBeforeWithdrawal,
        uint256 totalWETHValueOfWithdrawal,
        uint256 wethValueInUSDC
    );
    
    event AgentAumFeeCollected(
        address indexed agentFeeWallet,
        uint256 agentSharesMinted,
        address indexed protocolFeeRecipient,
        uint256 protocolSharesMinted,
        uint256 totalFeeValueInAccountingAsset,
        uint256 navAtFeeCalculation,
        uint256 totalSharesAtFeeCalculation,
        uint256 timestamp
    );
    
    event EmergencyWithdrawal(address indexed token, uint256 amount);

    // State Variable Getters
    function agent() external view returns (address);
    function dexRouter() external view returns (IAerodromeRouter);
    function ACCOUNTING_ASSET() external view returns (address);
    function USDC_ADDRESS() external view returns (address);
    function allowedTokens(uint256 index) external view returns (address);
    function targetWeights(address token) external view returns (uint256);
    function isAllowedTokenInternal(address token) external view returns (bool);
    function agentAumFeeWallet() external view returns (address);
    function agentAumFeeBps() external view returns (uint256);
    function protocolAumFeeRecipient() external view returns (address);
    function lastAgentAumFeeCollectionTimestamp() external view returns (uint256);
    
    // Constants
    function TOTAL_WEIGHT_BASIS_POINTS() external view returns (uint256);
    function AGENT_AUM_FEE_SHARE_BPS() external view returns (uint256);
    function PROTOCOL_AUM_FEE_SHARE_BPS() external view returns (uint256);
    function DEFAULT_SLIPPAGE_BPS() external view returns (uint256);
    function SWAP_DEADLINE_OFFSET() external view returns (uint256);
    function DEFAULT_POOL_STABILITY() external view returns (bool);
    function REBALANCE_DEVIATION_THRESHOLD_BPS() external view returns (uint256);

    // Core Functions
    function totalNAVInAccountingAsset() external view returns (uint256);
    function totalNAVInUSDC() external view returns (uint256);
    
    function deposit(uint256 amountWETHToDeposit, address receiver) external returns (uint256 sharesMinted);
    function withdraw(uint256 sharesToBurn, address receiver, address owner) external;
    
    function collectAgentManagementFee() external;
    
    function setAgent(address _newAgent) external;
    function setTargetWeights(uint256[] calldata _weights) external;
    function setTargetWeightsAndRebalanceIfNeeded(uint256[] calldata _weights) external;
    function triggerRebalance() external;
    
    function getTargetCompositionBPS() external view returns (
        uint256[] memory targetComposition_,
        address[] memory tokenAddresses_,
        string[] memory tokenSymbols_
    );
    
    function getCurrentCompositionBPS() external view returns (
        uint256[] memory currentComposition_,
        address[] memory tokenAddresses_,
        string[] memory tokenSymbols_
    );
    
    function emergencyWithdrawERC20(address _tokenAddress, address _to, uint256 _amount) external;
    function emergencyWithdrawNative(address payable _to, uint256 _amount) external;
}
```

### Function Details

#### deposit()
```solidity
function deposit(uint256 amountWETHToDeposit, address receiver) external returns (uint256 sharesMinted);
```
- **Access**: Public
- **Purpose**: Invest WETH and receive fund shares
- **Minimum**: 0.01 WETH (0.01 WETH for first deposit)
- **Returns**: Number of shares minted
- **Side Effects**: May trigger rebalancing

#### withdraw()
```solidity
function withdraw(uint256 sharesToBurn, address receiver, address owner) external;
```
- **Access**: Public (with allowance checks)
- **Purpose**: Burn shares and receive proportional assets
- **Returns**: All fund assets proportionally
- **Side Effects**: May trigger rebalancing

#### setTargetWeights()
```solidity
function setTargetWeights(uint256[] calldata _weights) external;
```
- **Access**: Agent only
- **Purpose**: Update portfolio target allocations
- **Requirements**: Weights must sum to 10000, no zero weights
- **Side Effects**: Does not trigger immediate rebalancing

#### collectAgentManagementFee()
```solidity
function collectAgentManagementFee() external;
```
- **Access**: Public
- **Purpose**: Calculate and distribute AUM fees
- **Mechanism**: Mints shares to fee recipients
- **Distribution**: 60% agent, 40% protocol

### Integration Example

```javascript
// TypeScript interface for fund contract
interface IWhackRockFund extends IERC20 {
    // Investment operations
    deposit(
        amountWETHToDeposit: BigNumberish,
        receiver: string,
        overrides?: Overrides
    ): Promise<ContractTransaction>;
    
    withdraw(
        sharesToBurn: BigNumberish,
        receiver: string,
        owner: string,
        overrides?: Overrides
    ): Promise<ContractTransaction>;
    
    // Portfolio management
    setTargetWeights(
        weights: BigNumberish[],
        overrides?: Overrides
    ): Promise<ContractTransaction>;
    
    triggerRebalance(overrides?: Overrides): Promise<ContractTransaction>;
    
    // View functions
    totalNAVInAccountingAsset(overrides?: CallOverrides): Promise<BigNumber>;
    getCurrentCompositionBPS(overrides?: CallOverrides): Promise<{
        currentComposition_: BigNumber[];
        tokenAddresses_: string[];
        tokenSymbols_: string[];
    }>;
    
    // State variables
    agent(overrides?: CallOverrides): Promise<string>;
    agentAumFeeBps(overrides?: CallOverrides): Promise<BigNumber>;
}
```

## Supporting Interfaces

### IAerodromeRouter

```solidity
interface IAerodromeRouter {
    function weth() external view returns (address);
    
    function swapExactTokensForTokens(
        uint256 amountIn,
        uint256 amountOutMin,
        address[] calldata path,
        address to,
        uint256 deadline
    ) external returns (uint256[] memory amounts);
    
    function getAmountsOut(
        uint256 amountIn,
        address[] calldata path
    ) external view returns (uint256[] memory amounts);
}
```

### IERC20 (Standard)

```solidity
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

## Error Interfaces

### Fund Errors

```solidity
// Custom errors used in WhackRockFund
error E1(); // Zero address
error E2(); // Invalid amount/length
error E3(); // Insufficient balance
error E4(); // Unauthorized (not agent)
error E5(); // Invalid state
error E6(); // Swap failed
```

### Standard Errors

```solidity
// OpenZeppelin ERC20 errors
error ERC20InsufficientBalance(address sender, uint256 balance, uint256 needed);
error ERC20InvalidSender(address sender);
error ERC20InvalidReceiver(address receiver);
error ERC20InsufficientAllowance(address spender, uint256 allowance, uint256 needed);
error ERC20InvalidApprover(address approver);
error ERC20InvalidSpender(address spender);
```

## ABI Specifications

### Registry ABI (Simplified)

```json
[
  {
    "type": "function",
    "name": "createWhackRockFund",
    "inputs": [
      {"name": "_initialAgent", "type": "address"},
      {"name": "_fundAllowedTokens", "type": "address[]"},
      {"name": "_initialTargetWeights", "type": "uint256[]"},
      {"name": "_vaultName", "type": "string"},
      {"name": "_vaultSymbol", "type": "string"},
      {"name": "_vaultURI", "type": "string"},
      {"name": "_description", "type": "string"},
      {"name": "_agentAumFeeWalletForFund", "type": "address"},
      {"name": "_agentSetTotalAumFeeBps", "type": "uint256"}
    ],
    "outputs": [{"name": "fundAddress", "type": "address"}],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "getRegistryAllowedTokens",
    "inputs": [],
    "outputs": [{"name": "", "type": "address[]"}],
    "stateMutability": "view"
  },
  {
    "type": "event",
    "name": "WhackRockFundCreated",
    "inputs": [
      {"name": "fundId", "type": "uint256", "indexed": true},
      {"name": "fundAddress", "type": "address", "indexed": true},
      {"name": "creator", "type": "address", "indexed": true},
      {"name": "initialAgent", "type": "address", "indexed": false},
      {"name": "vaultName", "type": "string", "indexed": false},
      {"name": "vaultSymbol", "type": "string", "indexed": false}
    ]
  }
]
```

### Fund ABI (Key Functions)

```json
[
  {
    "type": "function",
    "name": "deposit",
    "inputs": [
      {"name": "amountWETHToDeposit", "type": "uint256"},
      {"name": "receiver", "type": "address"}
    ],
    "outputs": [{"name": "sharesMinted", "type": "uint256"}],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "withdraw",
    "inputs": [
      {"name": "sharesToBurn", "type": "uint256"},
      {"name": "receiver", "type": "address"},
      {"name": "owner", "type": "address"}
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "totalNAVInAccountingAsset",
    "inputs": [],
    "outputs": [{"name": "", "type": "uint256"}],
    "stateMutability": "view"
  }
]
```

## Integration Examples

### Complete Registry Integration

```javascript
import { ethers } from 'ethers';
import { IWhackRockFundRegistry } from './types/contracts';

class RegistryIntegration {
    private registry: IWhackRockFundRegistry;
    
    constructor(registryAddress: string, signer: ethers.Signer) {
        this.registry = new ethers.Contract(
            registryAddress,
            REGISTRY_ABI,
            signer
        ) as IWhackRockFundRegistry;
    }
    
    async createFund(params: FundCreationParams): Promise<string> {
        const tx = await this.registry.createWhackRockFund(
            params.initialAgent,
            params.fundAllowedTokens,
            params.initialTargetWeights,
            params.vaultName,
            params.vaultSymbol,
            params.vaultURI,
            params.description,
            params.agentAumFeeWallet,
            params.agentSetTotalAumFeeBps
        );
        
        const receipt = await tx.wait();
        const event = receipt.events?.find(e => e.event === 'WhackRockFundCreated');
        
        return event?.args?.fundAddress;
    }
    
    async getAllowedTokens(): Promise<string[]> {
        return await this.registry.getRegistryAllowedTokens();
    }
}
```

### Complete Fund Integration

```javascript
class FundIntegration {
    private fund: IWhackRockFund;
    
    constructor(fundAddress: string, signer: ethers.Signer) {
        this.fund = new ethers.Contract(
            fundAddress,
            FUND_ABI,
            signer
        ) as IWhackRockFund;
    }
    
    async invest(wethAmount: BigNumber): Promise<BigNumber> {
        const tx = await this.fund.deposit(wethAmount, await this.fund.signer.getAddress());
        const receipt = await tx.wait();
        
        const event = receipt.events?.find(e => e.event === 'WETHDepositedAndSharesMinted');
        return event?.args?.sharesMinted || BigNumber.from(0);
    }
    
    async getPortfolioComposition() {
        const [weights, addresses, symbols] = await this.fund.getCurrentCompositionBPS();
        
        return addresses.map((address, i) => ({
            token: address,
            symbol: symbols[i],
            weight: weights[i].toNumber() / 100 // Convert to percentage
        }));
    }
}
```

## Related Documentation

- [Registry Functions](../registry/functions.md) - Detailed function documentation
- [Fund Operations](../fund/investment-ops.md) - Investment operation details
- [Integration Guide](../integration/quick-start.md) - Practical integration examples