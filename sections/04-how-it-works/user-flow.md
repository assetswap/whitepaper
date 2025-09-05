# User Flow Examples

## Example 1: New User - First Trade

### User Story
Sarah, a marketing manager with no trading experience, heard about AssetSwap from a friend. She wants to invest $1,000 in crypto but doesn't know where to start.

### Flow

1. **Onboarding (2 minutes)**
   - Signs up with email
   - Connects wallet (or creates new one)
   - Completes risk assessment quiz

2. **First Command**
   - Types: "Invest $1000 safely in crypto"
   - AGI responds: "I'll create a diversified portfolio focusing on established tokens with good safety scores"

3. **AGI Actions**
   - Analyzes top 50 cryptocurrencies
   - Filters for security score >80
   - Creates balanced allocation:
     - 40% BTC (security: 95)
     - 30% ETH (security: 92)
     - 20% SOL (security: 85)
     - 10% Stablecoins (safety reserve)

4. **Execution**
   - Shows proposed allocation to user
   - User approves with one click
   - AGI executes all trades
   - Sends confirmation

5. **Ongoing Management**
   - AGI monitors 24/7
   - Rebalances monthly
   - Alerts on significant changes
   - Provides weekly summaries

## Example 2: Active Trader - Complex Strategy

### User Story
Mike is an experienced trader who wants to automate his meme coin strategy while he focuses on his day job.

### Flow

1. **Strategy Setup**
   - Command: "Buy new meme coins with strong communities, $100 each, max 10 positions, sell when 3x or score drops below 50"

2. **AGI Configuration**
   - Parsing Parameters:
     - Asset type: Meme coins
     - Entry criteria: New + strong community
     - Position size: $100
     - Max positions: 10
     - Exit: 3x profit OR score <50

3. **Continuous Operation**
   - **Hour 1**: Identifies new token launch, community score 75, buys $100
   - **Hour 4**: Second token found, score 82, buys $100
   - **Day 2**: First token hits 3x, automatically sells
   - **Day 3**: Third token score drops to 48, sells at small loss
   - **Week 1**: Portfolio has 7 positions, 3 exits (2 profitable, 1 loss)

4. **Performance Tracking**
   - Total invested: $700
   - Total returned: $950
   - ROI: 35.7%
   - Win rate: 66%

## Example 3: DCA Investor - Long-term Accumulation

### User Story
Jennifer wants to build a crypto position over time without watching markets.

### Flow

1. **Setup Recurring Strategy**
   - Command: "Buy $500 of Bitcoin every Monday, but wait for a red day if the week starts green"

2. **AGI Interpretation**
   - Amount: $500
   - Asset: Bitcoin
   - Frequency: Weekly (Monday)
   - Condition: Price optimization on green opens

3. **Execution Pattern**
   - **Week 1**: Monday opens -2%, executes immediately
   - **Week 2**: Monday opens +5%, waits until Wednesday (-1.5%), executes
   - **Week 3**: Monday opens -0.5%, executes immediately
   - **Week 4**: Monday opens +8%, waits until Thursday (-2%), executes

4. **Results After 3 Months**
   - Average entry: 3.2% better than fixed Monday buying
   - Total accumulated: 0.75 BTC
   - Cost basis: $45,200
   - Current value: $52,500

## Example 4: Risk-Averse Investor - Protected Growth

### User Story
Robert wants crypto exposure but can't afford to lose more than 20% of his investment.

### Flow

1. **Risk-Protected Setup**
   - Command: "Invest $10,000 in crypto but never let me lose more than $2,000"

2. **AGI Strategy**
   - Allocates 80% to lower-risk assets
   - Sets portfolio-wide stop loss at -20%
   - Implements graduated risk reduction:
     - At -10%: Reduce risky positions by 50%
     - At -15%: Move 30% to stables
     - At -18%: Full defensive mode

3. **Market Downturn Scenario**
   - Portfolio drops to -10%
   - AGI automatically reduces meme coin exposure
   - Shifts allocation to BTC/ETH
   - Market recovers, portfolio protected
   - Final outcome: -5% instead of -30% market drop

## Example 5: Opportunity Hunter - Alert-Based Trading

### User Story
Alex wants to catch rapid moves but can't monitor markets constantly.

### Flow

1. **Opportunity Criteria**
   - Command: "Alert me to any token that gains 50% in an hour with over $1M volume, auto-buy $200 if security score is above 70"

2. **AGI Monitoring**
   - Scans 1000+ tokens continuously
   - Triggers on qualifying events
   - Verifies security before executing

3. **Real Execution**
   - **Day 1**: 2 opportunities, 1 passes security, AGI buys
   - **Day 3**: 5 opportunities, 2 pass security, AGI buys both
   - **Day 5**: First position +120%, takes partial profits
   - **Week 1 Results**: 3 positions, average gain +45%

## Common Patterns

### Most Used Commands
1. "Buy [amount] of [token] when [condition]"
2. "Sell [percentage] when profit reaches [target]"
3. "Rebalance my portfolio weekly"
4. "Find the next trending token"
5. "Protect my portfolio if market crashes"

### Success Metrics
- **Average Time to First Trade**: 5 minutes
- **User Retention**: 65% monthly active
- **Strategy Success Rate**: 73%
- **User Satisfaction**: 4.6/5 stars