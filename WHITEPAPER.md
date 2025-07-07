# WhackRock Protocol: A Trust Layer for AI-Managed Investment Funds

**Version 1.0**  
**December 2024**

---

## Abstract

WhackRock Protocol establishes a decentralized infrastructure for AI-managed investment funds, providing transparency, accountability, and standardization to autonomous financial management. Built on Base, the protocol enables AI agents to manage tokenized investment vehicles while ensuring investor protection through immutable smart contracts and on-chain transparency.

The protocol integrates with the GAME SDK (Virtuals Protocol) to provide sophisticated AI agent capabilities, allowing developers to deploy autonomous fund managers that can make investment decisions, rebalance portfolios, and collect performance-based fees in a trustless environment.

**Key Security Features:**
- AI agents can ONLY trade pre-specified, highly liquid tokens defined at fund creation
- Token lists are IMMUTABLE - agents cannot add new tokens or change approved assets
- Agents have NO withdrawal capabilities - only investors can redeem their shares
- All agent actions are limited to portfolio weight adjustments and rebalancing
- Smart contracts enforce all constraints, preventing any unauthorized actions

---

## 1. Introduction

### 1.1 The Problem

The emergence of AI-driven investment strategies has created significant opportunities, but current implementations suffer from critical limitations:

- **Lack of Accountability**: AI investment decisions operate in black boxes without verifiable performance tracking
- **Centralized Risk**: Funds managed by traditional entities create single points of failure
- **Opaque Operations**: Investors cannot verify AI decision-making processes or fund management activities
- **Inconsistent Standards**: No standardized framework for comparing AI fund performance

### 1.2 The Solution

WhackRock Protocol addresses these challenges by providing:

- **Transparent Operations**: All fund activities recorded immutably on-chain
- **Standardized Infrastructure**: Uniform smart contract interfaces for fund creation and management
- **Decentralized Architecture**: No single point of control or failure
- **Performance Accountability**: Verifiable track records for AI agents and fund managers

---

## 2. Protocol Architecture

### 2.1 Core Components

#### Fund Registry
The central registry contract maintains a comprehensive directory of all WhackRock funds, providing:
- Fund discovery and enumeration
- Metadata management
- Performance tracking
- Agent assignment verification

#### Fund Contracts (WhackRockFundV6_UniSwap_TWAP)
Individual fund smart contracts implementing:
- ERC20 tokenized shares
- Portfolio weight management
- Automatic rebalancing mechanisms
- Fee collection and distribution
- TWAP-based price oracles for accurate valuation

#### AI Agent Integration
Integration layer connecting fund contracts with AI agents through:
- Standardized function interfaces
- Permission-based access controls
- Performance-based fee structures
- Decision audit trails

### 2.2 GAME SDK Integration with Security Constraints

WhackRock Protocol leverages the GAME SDK (Virtuals Protocol) to provide sophisticated AI agent capabilities while maintaining strict security constraints:

#### Agent Deployment with Limited Permissions
```solidity
// Example agent configuration with restricted permissions
Worker gameAgent = Worker({
    api_key: GAME_API_KEY,
    description: "Investment Signal Analyzer",
    instruction: "Analyze market data and generate portfolio weights...",
    model_name: "Llama-3.1-405B-Instruct",
    allowed_actions: ["SET_WEIGHTS", "TRIGGER_REBALANCE"]  // Cannot withdraw or transfer funds
});
```

#### Predefined Action Constraints
AI agents are strictly limited to the following actions:
- **Set Portfolio Weights**: Only adjust target allocation percentages
- **Trigger Rebalancing**: Execute trades within predefined token lists
- **Collect Fees**: Only through protocol-defined mechanisms

**Agents CANNOT**:
- Withdraw or transfer funds to external addresses
- Add new tokens to the approved list
- Change fund parameters or security settings
- Access private keys or execute arbitrary transactions
- Bypass smart contract restrictions

#### Token Restrictions
Agents can only trade:
- **Prespecified Token Lists**: Defined at fund creation and immutable
- **Highly Liquid Assets**: Only tokens meeting minimum liquidity thresholds
- **Verified Contracts**: Tokens must pass security validation
- **No Dynamic Additions**: Cannot add new tokens post-deployment

---

## 3. Fund Creation and Management

### 3.1 Creating a Fund with GAME Agents

Fund creators can deploy new investment vehicles through a streamlined process:

#### Prerequisites
- GAME API key and agent configuration
- Initial token selection and target weights
- Fund parameters (fees, rebalancing thresholds)
- Agent strategy description

#### Deployment Process with Security Safeguards
1. **Token Whitelist Definition**: Select only highly liquid, verified tokens
2. **Agent Setup**: Configure GAME agent with limited permissions
3. **Fund Deployment**: Deploy immutable fund contract with security constraints
4. **Registry Registration**: Register fund with transparency requirements
5. **Initial Configuration**: Set portfolio weights within allowed tokens only

#### Example Fund Creation with Security Parameters
```javascript
const fundParams = {
  name: "AI Momentum Strategy",
  symbol: "AIMS",
  description: "AI-driven momentum trading strategy",
  allowedTokens: ["WETH", "USDC", "WBTC"],  // IMMUTABLE - Cannot be changed post-deployment
  targetWeights: [4000, 3000, 3000], // Basis points
  securityParams: {
    maxSlippage: 100,  // 1% max slippage protection
    minLiquidity: 1000000,  // $1M minimum liquidity requirement
    rebalanceThreshold: 500,  // 5% deviation trigger
    emergencyPause: true  // Allow emergency pause by owner only
  },
  gameAgentConfig: {
    strategy: "momentum_trading",
    riskTolerance: "moderate",
    rebalanceFrequency: "daily",
    permissions: ["SET_WEIGHTS", "TRIGGER_REBALANCE"]  // Restricted action set
  }
};
```

### 3.2 AI Fund Management with Security Constraints

#### Autonomous Operations Within Strict Boundaries
GAME agents manage funds through predefined, secure mechanisms:
- **Continuous Monitoring**: Real-time surveillance of ONLY whitelisted tokens
- **Signal Analysis**: Processing data within approved parameters
- **Portfolio Optimization**: Weight adjustments ONLY within allowed tokens
- **Risk Management**: Enforced by smart contract, not agent discretion

#### Agent Responsibilities and Limitations
**What Agents CAN Do:**
- Monitor market conditions for whitelisted tokens only
- Calculate optimal weights within the predefined token set
- Trigger rebalancing through secure smart contract functions
- Collect fees ONLY through protocol-defined mechanisms

**What Agents CANNOT Do:**
- Add or remove tokens from the approved list
- Withdraw funds to any address
- Execute trades outside the whitelisted token pairs
- Override smart contract security parameters
- Access user funds directly
- Execute arbitrary contract calls

#### Security-First Performance Tracking
All agent actions are transparent and auditable:
- Every weight change requires smart contract validation
- All trades execute through secure DEX integrations
- Fund assets remain in the smart contract at all times
- Users can withdraw their proportional share anytime
- Emergency pause functionality for extreme scenarios

---

## 4. Investment and Portfolio Management

### 4.1 Investor Participation

#### Investment Process
1. **Fund Discovery**: Browse available funds in registry
2. **Due Diligence**: Review agent performance and strategy
3. **Investment**: Deposit WETH to receive tokenized shares
4. **Monitoring**: Track performance through transparent metrics

#### Share Tokenization
WhackRock funds issue ERC20 tokens representing proportional ownership:
- Dynamic pricing based on NAV per share
- Automatic share minting/burning on deposits/withdrawals
- Proportional claim to underlying assets

#### Withdrawal Mechanism
Investors can exit positions through:
- **Basket Withdrawals**: Receive proportional share of all fund assets
- **On-Demand Liquidity**: No lock-up periods or withdrawal restrictions
- **Transparent Pricing**: NAV calculated using TWAP oracles

### 4.2 Portfolio Rebalancing

#### Automated Rebalancing
Fund contracts automatically rebalance when:
- Portfolio weights deviate beyond threshold (default: 5%)
- Agent signals trigger rebalancing
- Major market events require position adjustments

#### Rebalancing Process
1. **Deviation Detection**: Monitor current vs. target weights
2. **Agent Consultation**: Verify rebalancing necessity
3. **Trade Execution**: Swap tokens through DEX integrations
4. **Weight Restoration**: Achieve target portfolio allocation

#### DEX Integration
Rebalancing utilizes:
- Uniswap V3 for primary liquidity
- TWAP oracles for fair pricing
- Slippage protection mechanisms
- MEV-resistant execution strategies

---

## 5. Economic Model

### 5.1 Fee Structure

#### Management Fees
- **Agent Fees**: Performance-based compensation for AI agents
- **Protocol Fees**: Revenue for protocol development and maintenance
- **Fee Distribution**: Automatic allocation through smart contracts

#### Fee Collection Mechanism
```solidity
function collectAgentManagementFee() external {
    uint256 timeElapsed = block.timestamp - lastFeeCollection;
    uint256 annualizedFeeValue = (nav * agentAumFeeBps * timeElapsed) / 
                                (TOTAL_BASIS_POINTS * 365 days);
    
    uint256 agentShares = (annualizedFeeValue * AGENT_FEE_SHARE) / nav;
    uint256 protocolShares = (annualizedFeeValue * PROTOCOL_FEE_SHARE) / nav;
    
    _mint(agentFeeWallet, agentShares);
    _mint(protocolFeeRecipient, protocolShares);
}
```

### 5.2 Staking and Protocol Rewards

#### Staking Mechanism
WhackRock implements a comprehensive staking system to align long-term incentives and distribute protocol rewards:

**Staking Features:**
- **Single-Asset Staking**: Stake WROCK tokens to earn protocol rewards
- **Flexible Lock Periods**: Choose staking duration for multiplied rewards
- **Auto-Compounding**: Automatic reinvestment of earned rewards
- **Liquid Staking**: Receive stWROCK tokens representing staked position

#### Reward Distribution
```solidity
// Staking rewards calculation
function calculateRewards(address staker) public view returns (uint256) {
    StakeInfo memory stake = stakes[staker];
    uint256 timeStaked = block.timestamp - stake.timestamp;
    uint256 baseReward = (stake.amount * rewardRate * timeStaked) / SECONDS_PER_YEAR;
    uint256 multiplier = getLockMultiplier(stake.lockPeriod);
    return baseReward * multiplier / 100;
}
```

**Reward Sources:**
1. **Protocol Fees**: Percentage of all fund management fees
2. **Performance Fees**: Share of successful fund performance
3. **Ecosystem Growth**: Treasury allocations for growth incentives
4. **Partner Rewards**: Integration and partnership revenues

### 5.3 Points System and Airdrop Mechanism

#### Points Accumulation
Users earn points through various protocol interactions:

**Point-Earning Activities:**
- **Staking Duration**: 100 points per WROCK per day staked
- **Fund Investment**: 50 points per $100 invested per day
- **Liquidity Provision**: 150 points per $100 in LP per day
- **Protocol Usage**: 10 points per transaction
- **Referrals**: 500 points per successful referral

#### Points Redeemer Contract
The Points Redeemer Contract enables users to claim airdropped rewards based on accumulated points:

```solidity
contract PointsRedeemer {
    mapping(address => uint256) public userPoints;
    mapping(uint256 => AirdropRound) public airdropRounds;
    
    struct AirdropRound {
        uint256 totalRewards;
        uint256 totalPoints;
        uint256 rewardPerPoint;
        mapping(address => bool) claimed;
    }
    
    function claimAirdrop(uint256 roundId) external {
        AirdropRound storage round = airdropRounds[roundId];
        require(!round.claimed[msg.sender], "Already claimed");
        
        uint256 userReward = userPoints[msg.sender] * round.rewardPerPoint;
        round.claimed[msg.sender] = true;
        
        IERC20(rewardToken).transfer(msg.sender, userReward);
        emit AirdropClaimed(msg.sender, roundId, userReward);
    }
}
```

#### Airdrop Schedule
**Regular Airdrops:**
- **Monthly Rewards**: Distribution of protocol fees to point holders
- **Quarterly Bonuses**: Additional rewards for consistent participants
- **Special Events**: Partnership tokens and new feature launches
- **Milestone Rewards**: Achievements-based token distributions

**Reward Calculation:**
```
User Reward = (User Points / Total Points) × Total Airdrop Amount
```

### 5.4 Agent Incentives

#### Performance Alignment
- Fees tied to fund performance metrics
- Long-term value creation incentives
- Reputation-based agent selection
- Staking requirements for fund managers

#### Economic Security
- Agent stake requirements for fund management
- Slashing mechanisms for poor performance
- Insurance funds for investor protection
- Points multipliers for successful agents

---

## 6. Security and Risk Management

### 6.1 Smart Contract Security

#### Immutable Token Lists
- **Fixed at Deployment**: Token whitelists cannot be modified post-deployment
- **Liquidity Requirements**: Only tokens with proven liquidity profiles
- **Verified Contracts**: All tokens undergo security verification
- **No Proxy Contracts**: Preventing malicious upgrades

#### Access Control Hierarchy
```solidity
// Agent permissions are strictly limited
modifier onlyAgent() {
    require(msg.sender == agent, "Unauthorized");
    require(allowedActions[msg.sender]["SET_WEIGHTS"], "Action not permitted");
    _;
}

// Agents cannot access withdrawal functions
modifier onlyShareOwner() {
    require(balanceOf(msg.sender) > 0, "Not a shareholder");
    require(msg.sender != agent, "Agents cannot withdraw");
    _;
}
```

#### Fund Drainage Prevention
- **No Agent Withdrawals**: Agents have zero access to withdrawal functions
- **User-Only Redemptions**: Only token holders can withdraw their proportional share
- **Basket Withdrawals**: Users receive all tokens proportionally, not just one
- **Time-Locked Changes**: Any parameter changes require time delays

### 6.2 AI Agent Security

#### Restricted Action Set
Agents operate within a sandbox of allowed functions:
1. `setTargetWeights()` - Only adjust percentages, not token lists
2. `triggerRebalance()` - Execute trades within constraints
3. `collectManagementFee()` - Protocol-defined fee collection only

#### Hard-Coded Constraints
```solidity
// Example: Agents cannot add new tokens
function addToken(address token) external {
    revert("Token list is immutable");
}

// Example: Agents cannot withdraw funds
function emergencyWithdraw() external onlyOwner {
    require(msg.sender != agent, "Agents cannot withdraw");
    // Emergency withdrawal logic
}
```

#### Real-Time Monitoring
- On-chain activity tracking for anomaly detection
- Automatic circuit breakers for unusual patterns
- Community-driven security alerts
- Transparent decision logs for all agent actions

---

## 7. Governance and Decentralization

### 7.1 Protocol Governance

#### Decentralized Decision Making
- Community-driven protocol upgrades
- Parameter adjustment proposals
- Agent certification processes
- Staking-weighted voting power

#### Governance Token (WROCK)
- **Voting Rights**: Proportional to staked WROCK amount
- **Fee Distribution**: Stakers receive share of protocol fees
- **Lock Multipliers**: Longer stakes receive more voting power
- **Delegation**: Ability to delegate voting power while staking

#### Voting Power Calculation
```solidity
function getVotingPower(address user) public view returns (uint256) {
    StakeInfo memory stake = stakes[user];
    uint256 basePower = stake.amount;
    uint256 lockMultiplier = getLockMultiplier(stake.lockPeriod);
    uint256 loyaltyBonus = calculateLoyaltyBonus(user);
    return (basePower * lockMultiplier * loyaltyBonus) / 10000;
}
```

### 7.2 Agent Ecosystem

#### Agent Standards
- Standardized interfaces for fund management
- Performance benchmarking frameworks
- Community-driven agent development

#### Marketplace Dynamics
- Competition among AI agents
- Performance-based fund allocation
- Investor choice and fund migration

---

## 8. Technical Implementation

### 8.1 Smart Contract Architecture

#### Contract Hierarchy
```
FundRegistry
   WhackRockFund (ERC20)
      UniswapV3TWAPOracle
      PortfolioManager
      FeeCollector
   GameAgentInterface
```

#### Key Functions with Access Controls
```solidity
// Public Functions - Available to All Users
function deposit(uint256 amount, address receiver) external returns (uint256 shares)
function withdraw(uint256 shares, address receiver, address owner) external  // AGENT CANNOT CALL

// Agent-Only Functions - Restricted Permissions
function setTargetWeights(uint256[] weights) external onlyAgent  // Must sum to 10000 bps
function triggerRebalance() external onlyAgent  // Only trades whitelisted tokens

// Forbidden Functions - What Agents CANNOT Do
function addAllowedToken(address token) external  // DOES NOT EXIST - Tokens are immutable
function transferFunds(address to, uint256 amount) external  // DOES NOT EXIST - No direct transfers
function updateAllowedTokens(address[] tokens) external  // DOES NOT EXIST - Cannot change token list
function emergencyWithdraw() external onlyOwner  // OWNER ONLY - Agent cannot call
```

### 8.2 GAME Agent Integration

#### Agent Communication
```python
# GAME agent signal generation
def generate_investment_signal(market_data):
    analysis = worker.analyze_market_conditions(market_data)
    weights = worker.optimize_portfolio(analysis)
    return {
        'target_weights': weights,
        'confidence': analysis.confidence,
        'reasoning': analysis.explanation
    }
```

#### On-Chain Execution
Agents interact with fund contracts through:
- Authenticated API calls
- Cryptographic signature verification
- Rate limiting and access controls

---

## 9. Use Cases and Applications

### 9.1 For AI Developers

#### Agent Development
- Deploy sophisticated investment strategies
- Build verifiable performance track records
- Monetize AI capabilities through fund management

#### Revenue Opportunities
- Performance-based fee collection
- Agent licensing to multiple funds
- Consulting services for fund creation

### 9.2 For Investors

#### Diversified Exposure
- Access to multiple AI strategies
- Diversified portfolio management
- Professional-grade investment tools
- Staking rewards for long-term holders

#### Risk Management
- Transparent performance tracking
- On-demand liquidity
- Regulated smart contract environment
- Insurance through protocol reserves

#### Staking Benefits
- **Protocol Rewards**: Earn fees from all fund operations
- **Governance Power**: Influence protocol development
- **Airdrop Eligibility**: Access to partner token distributions
- **Points Accumulation**: Build position for future rewards

### 9.3 For Fund Managers

#### Operational Efficiency
- Automated portfolio management
- Reduced operational overhead
- Scalable fund administration

#### Innovation Platform
- Experiment with AI strategies
- Access to cutting-edge technology
- Community-driven development

---

## 10. Future Developments

### 10.1 Protocol Enhancements

#### Advanced Features
- Cross-chain fund management
- Institutional custody integration
- Regulatory compliance modules

#### Performance Improvements
- Gas optimization strategies
- Layer 2 scaling solutions
- Enhanced oracle systems

### 10.2 Ecosystem Expansion

#### Integration Partnerships
- Traditional finance bridges
- DeFi protocol integrations
- Institutional adoption pathways

#### Community Growth
- Developer incentive programs
- Educational initiatives
- Research partnerships

---

## 11. Conclusion

WhackRock Protocol represents a paradigm shift in investment management, combining the sophistication of AI-driven strategies with the transparency and security of blockchain technology. By providing standardized infrastructure for AI-managed funds, the protocol enables unprecedented innovation in autonomous finance while maintaining investor protection and market integrity.

The integration with GAME SDK ensures that fund managers have access to state-of-the-art AI capabilities, while the decentralized architecture guarantees transparency and eliminates single points of failure. As the protocol evolves, it will continue to push the boundaries of what's possible in autonomous financial management, creating new opportunities for developers, investors, and the broader DeFi ecosystem.

Through WhackRock Protocol, the future of finance is not just automatedit's transparent, accountable, and accessible to all.

---

## References

1. Virtuals Protocol GAME SDK Documentation
2. Ethereum Smart Contract Security Best Practices
3. Uniswap V3 Core and Periphery Contracts
4. ERC20 Token Standard Specification
5. Time-Weighted Average Price (TWAP) Oracle Design
6. Decentralized Autonomous Organization (DAO) Governance Models

---

**Repository**: [https://github.com/WhackRock/whackrock-treasury-template](https://github.com/WhackRock/whackrock-treasury-template)  
**Documentation**: [https://github.com/WhackRock/docs](https://github.com/WhackRock/docs)  
**Website**: [https://whackrock.ai](https://whackrock.ai)

*This whitepaper is subject to updates as the protocol evolves. Please refer to the official documentation for the most current information.*