---
icon: building
---

# Technical Architecture

## System Overview

AssetSwap's architecture is built as a modular Node.js application that integrates AI agents, blockchain interactions, and real-time data processing. The system uses a microservices approach with clear separation of concerns between AI processing, order management, and blockchain execution.

## Core Architecture Components

### 1. AI Agent System

#### LangChain Router
The central routing system that processes user requests:

```javascript
// Actual implementation from router.js
export async function handleUserRequest(userInput, {
    agentName,
    threadId,
    user,
    sendEvent,
    metadata = {},
    controllers,
    attachments,
}) {
    agentName = agentName || 'assetswap';
    const agent = agents[agentName];
    
    // Intent classification and context enrichment
    // Agent selection based on user input
    // Tool execution and response generation
}
```

#### Specialized Agents

**Available Agents:**
- `assetswap.agent.js` - General trading and portfolio management
- `meme.agent.js` - Meme token analysis and trading
- `stock.agent.js` - Tokenized stock operations

Each agent uses LangGraph for workflow management:

```javascript
const workflow = new StateGraph(StateAnnotation)
    .addEdge('__start__', 'supervisor')
    .addEdge('supervisor', 'agent')
    .addEdge('tools', 'agent')
    .addNode('agent', agentNode)
    .addNode('tools', toolNode)
    .compile({ checkpointer, recursionLimit: 50 });
```

### 2. Order Management System

#### Order Model Structure

```javascript
// From order.model.js
const OrderSchema = {
    user: ObjectId,
    walletAddress: String,
    orderType: ['take_profit', 'stop_loss'],
    status: ['active', 'executing', 'executed', 'failed', 'expired'],
    
    // Token details
    tokenMint: String,
    tokenSymbol: String,
    amount: Number,
    
    // Trigger system
    triggerType: String, // 30+ types available
    triggerValue: Number,
    triggerField: String,
    currentValue: Number,
    
    // Execution details
    slippage: Number,
    expiresAt: Date,
    txHash: String,
    retryCount: Number
}
```

#### Trigger Types

The system supports extensive trigger conditions:

```javascript
// From trigger.service.js
const TRIGGER_FIELD_MAP = {
    // Price triggers
    'price': 'market.price',
    
    // Score triggers (AI-calculated)
    'security_score': 'scores.securityScore',
    'liquidity_score': 'scores.liquidityScore',
    'activity_score': 'scores.activityScore',
    'community_score': 'scores.communityScore',
    'momentum_score': 'scores.momentumScore',
    
    // Market data triggers
    'market_cap': 'market.marketCap',
    'volume': 'market.volume24h',
    'holders': 'holders.total',
    
    // Holder metrics
    'whale_holders': 'holders.whales',
    'top10_supply_percent': 'holders.top10SupplyPercent'
}
```

### 3. Trading Execution

#### Jupiter Integration

```javascript
// From trading.service.js
async function tradeTokens({
    inputMint,
    outputMint,
    inputAmount,
    slippageBps = 300,
    wallet,
    walletPrivateKey,
    rpcUrl,
    user = null
}) {
    // Step 1: Validate parameters
    const { connection, wallet: finalWallet } = await validateTradeParams();
    
    // Step 2: Check wallet balance
    const balanceInfo = await checkWalletBalance();
    
    // Step 3: Check for gas sponsorship
    const solBalanceCheck = await checkSponsorGasNeeded();
    
    // Step 4: Execute swap via Jupiter
    const result = await executeSwapTransaction();
    
    return result;
}
```

### 4. Token Scoring System

#### Scoring Components

```javascript
// From scoring.js - Transparent scoring implementation

// Liquidity Score (40% liquidity amount, 30% volume, 30% trend)
export function calculateLiquidityScore(data) {
    let score = 0;
    
    if (data.market?.liquidity) {
        const liquidity = data.market.liquidity;
        if (liquidity >= 1000000) score += 40;      // $1M+
        else if (liquidity >= 500000) score += 35;  // $500K+
        else if (liquidity >= 100000) score += 25;  // $100K+
        // ... scaled scoring
    }
    
    // Volume and trend analysis
    // Returns 0-100 score
}

// Activity Score (40% trades, 35% wallets, 25% balance)
export function calculateActivityScore(data) {
    // Multi-timeframe analysis (1h, 4h, 24h)
    // Unique wallet engagement scoring
    // Buy/sell ratio optimization
}

// Community Score (35% holders, 30% distribution, 25% growth)
export function calculateCommunityScore(data) {
    // Total holder count scaling
    // Whale vs retail distribution
    // Growth trend analysis
}

// Security Score (base + features - risks)
export function calculateSecurityScore(data) {
    // SolSniffer audit integration
    // Risk level penalties
    // Security feature bonuses
}
```

### 5. Data Layer

#### MongoDB Collections

```javascript
// Primary data models
- users           // User accounts and preferences
- orders          // All order types and states
- transactions    // Historical trades
- conversations   // AI chat history
- memes          // Token data with scores
- task-logs      // Background task execution
- automations    // User automation rules
```

#### Token Data Model

```javascript
// From meme.model.js
const MemeSchema = {
    // Core identification
    address: String,
    symbol: String,
    name: String,
    
    // Market data (from BirdEye)
    market: {
        price: Number,
        marketCap: Number,
        volume24h: Number,
        liquidity: Number,
        trades24h: Number,
        uniqueWallets24h: Number,
        // ... 50+ market metrics
    },
    
    // Holder data (from Moralis)
    holders: {
        total: Number,
        whales: Number,
        sharks: Number,
        dolphins: Number,
        top10SupplyPercent: Number,
        // ... distribution metrics
    },
    
    // Calculated scores
    scores: {
        liquidityScore: Number,  // 0-100
        activityScore: Number,   // 0-100
        communityScore: Number,  // 0-100
        securityScore: Number,   // 0-100
        momentumScore: Number,   // 0-100
        score: Number           // Overall 0-100
    }
}
```

### 6. Service Architecture

#### Core Services

```
src/services/
├── order/
│   ├── polling.service.js      # Order monitoring
│   └── trigger.service.js      # Trigger evaluation
├── jupiter/
│   ├── trading.service.js      # Trade execution
│   └── sponsor.service.js      # Gas sponsorship
├── token/
│   ├── scoring.js              # Score calculation
│   └── formatter.js            # Data formatting
├── task-executor.service.js    # Background tasks
└── credit.service.js           # User credits/rewards
```

#### Background Jobs

```javascript
// From polling.service.js
class OrderPollingService {
    async pollOrders() {
        // Runs every second
        const activeOrders = await Order.find({ 
            status: 'active' 
        }).limit(BATCH_SIZE);
        
        for (const order of activeOrders) {
            await this.evaluateAndExecute(order);
        }
    }
}
```

### 7. API Layer

#### REST Endpoints

```javascript
// From routes/
router.post('/api/trade', tradeController.executeTrade);
router.post('/api/orders', orderController.createOrder);
router.get('/api/orders', orderController.getUserOrders);
router.delete('/api/orders/:id', orderController.cancelOrder);

router.post('/api/ai/chat', aiController.chat);
router.get('/api/tokens/:address/score', tokenController.getScore);
```

#### WebSocket Implementation

```javascript
// From ws.handler.js
io.on('connection', (socket) => {
    socket.on('subscribe', ({ channel, params }) => {
        // Real-time price updates
        // Order status changes
        // Portfolio updates
    });
});
```

## Data Flow

### Trade Execution Flow

```
User Request → AI Agent → Intent Analysis → Tool Selection
    ↓
Jupiter Router ← Trade Parameters ← Wallet Validation
    ↓
Transaction Signing → Blockchain Submission → Confirmation
    ↓
Database Update → WebSocket Notification → User Response
```

### Order Monitoring Flow

```
Create Order → Database Storage → Active Order Pool
    ↓
Polling Service (1s intervals) → Trigger Evaluation
    ↓
Condition Met → Execute Trade → Update Status
    ↓
User Notification → Order Completion
```

## Performance Characteristics

### Current Metrics
- **Order Polling**: Every 1 second for active orders
- **Batch Processing**: 100 orders per batch
- **API Response**: <100ms average latency
- **Database Queries**: Indexed for performance
- **AI Response**: 1-3 seconds depending on complexity

### Scalability Design
- Horizontal scaling via PM2 cluster mode
- MongoDB replica sets for data redundancy
- Redis caching for frequently accessed data
- Queue-based task processing for background jobs

## Security Implementation

### Wallet Security
- Private keys encrypted before storage
- Google Cloud KMS integration available
- Non-custodial architecture
- Session-based key decryption

### API Security
- JWT authentication
- Rate limiting per endpoint
- IP-based access control
- Request validation middleware

## Monitoring & Logging

### Logging Infrastructure
```javascript
// Winston logger configuration
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.json(),
    transports: [
        new winston.transports.File({ filename: 'error.log' }),
        new winston.transports.File({ filename: 'combined.log' })
    ]
});
```

### Health Checks
- Database connectivity monitoring
- RPC endpoint health checks
- Service uptime tracking
- Error rate monitoring

## Deployment

### Environment Configuration
```bash
# Required environment variables
NODE_ENV=production
MONGODB_URI=mongodb://...
SOLANA_RPC_URL=https://...
OPENAI_API_KEY=...
JUPITER_API_URL=https://quote-api.jup.ag
```

### Production Setup
- Node.js 20.11.1 runtime
- PM2 for process management
- Nginx reverse proxy
- SSL/TLS termination
- CloudFlare DDoS protection

---

*This document reflects the actual implementation of AssetSwap's architecture as of September 2024.*