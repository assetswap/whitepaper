---
icon: chart-bar
---

# Token Scoring Methodology

## Overview

AssetSwap employs a transparent, multi-dimensional scoring system to evaluate tokens on Solana. Each token receives scores across five key dimensions, which are combined to produce an overall quality score from 0-100. This document details the exact methodology used in production.

## Component Scores

### 1. Liquidity Score (0-100)

Evaluates the token's market depth and ability to handle trades without significant slippage.

#### Calculation Breakdown

**Liquidity Amount (40% weight)**
```javascript
if (liquidity >= $1,000,000)    → 40 points  // Excellent
else if (liquidity >= $500,000) → 35 points  // Very Good
else if (liquidity >= $250,000) → 30 points  // Good
else if (liquidity >= $100,000) → 25 points  // Moderate
else if (liquidity >= $50,000)  → 20 points  // Low-Moderate
else if (liquidity >= $25,000)  → 15 points  // Low
else if (liquidity >= $10,000)  → 10 points  // Very Low
else                            → 0 points   // Insufficient
```

**Trading Volume (30% weight)**
```javascript
if (volume24h >= $1,000,000)    → 30 points  // Very High
else if (volume24h >= $100,000) → 25 points  // High
else if (volume24h >= $10,000)  → 20 points  // Medium
else if (volume24h >= $1,000)   → 10 points  // Low
else                            → 5 points   // Minimal
```

**Volume Trend Stability (30% weight)**
```javascript
if (volumeChange24h between -10% and +50%)     → 30 points  // Stable/Healthy
else if (volumeChange24h between +50% and +200%) → 20 points  // High Growth
else if (volumeChange24h between -25% and -10%)  → 15 points  // Moderate Decline
else if (volumeChange24h <= -50%)                → 5 points   // Severe Decline
```

### 2. Activity Score (0-100)

Measures trading engagement and market participation.

#### Calculation Breakdown

**Multi-Timeframe Trading Activity (40% weight)**

The system analyzes trading across three timeframes with weighted importance:
- 24-hour trades: 50% of activity score
- 1-hour trades: 30% of activity score  
- 4-hour trades: 20% of activity score

```javascript
// For each timeframe:
if (trades >= 100)      → Full weight allocation
else if (trades >= 50)  → 75% weight allocation
else if (trades >= 20)  → 50% weight allocation
else if (trades >= 5)   → 25% weight allocation
else                    → 12.5% weight allocation
```

**Unique Wallet Engagement (35% weight)**
```javascript
if (uniqueWallets24h >= 1000) → 35 points  // Excellent
else if (uniqueWallets24h >= 500)  → 28 points  // Very Good
else if (uniqueWallets24h >= 100)  → 20 points  // Good
else if (uniqueWallets24h >= 50)   → 15 points  // Moderate
else if (uniqueWallets24h >= 10)   → 10 points  // Low
else                               → 5 points   // Minimal
```

**Buy/Sell Balance (25% weight)**
```javascript
buyRatio = buys24h / (buys24h + sells24h)

if (buyRatio between 0.4 and 0.6)     → 25 points  // Optimal Balance
else if (buyRatio between 0.3 and 0.7) → 20 points  // Good Balance
else if (buyRatio between 0.2 and 0.8) → 15 points  // Acceptable
else                                   → 10 points  // Imbalanced
```

### 3. Community Score (0-100)

Analyzes the health and distribution of the token holder base.

#### Calculation Breakdown

**Total Holders (35% weight)**
```javascript
if (holders >= 10,000) → 35 points  // Large Community
else if (holders >= 5,000)  → 30 points  // Strong Community
else if (holders >= 1,000)  → 25 points  // Growing Community
else if (holders >= 500)    → 20 points  // Moderate Community
else if (holders >= 250)    → 15 points  // Small Community
else if (holders >= 100)    → 10 points  // Minimal Community
else                        → 0 points   // Insufficient
```

**Holder Distribution (30% weight)**

The system evaluates concentration risk:
- Whale holders (>$100K): Should be <5% of total holders
- Shark holders ($10K-$100K): Healthy range 10-20%
- Dolphin/Fish holders (<$10K): Should be >40% for good distribution

```javascript
// Whale concentration penalty
if (whaleRatio <= 0.05)      → 15 points  // Excellent
else if (whaleRatio <= 0.10)  → 12 points  // Good
else if (whaleRatio <= 0.15)  → 8 points   // Acceptable
else                          → 3 points   // Concerning

// Mid-tier holder bonus
if (midTierRatio >= 0.40)     → 15 points  // Excellent
else if (midTierRatio >= 0.30) → 12 points  // Good
else if (midTierRatio >= 0.20) → 8 points   // Acceptable
else                           → 3 points   // Low
```

**Holder Growth Trends (25% weight)**

Analyzes changes across multiple timeframes:
- 24-hour change: 40% weight
- 7-day change: 30% weight
- 30-day change: 30% weight

```javascript
if (changePercent >= 5%)       → Full weight  // Strong Growth
else if (changePercent >= 1%)  → 80% weight   // Good Growth
else if (changePercent >= 0%)  → 60% weight   // Stable
else if (changePercent >= -2%) → 40% weight   // Minor Decline
else if (changePercent >= -5%) → 20% weight   // Moderate Decline
else                           → 0% weight    // Severe Decline
```

**Acquisition Method Diversity (10% weight)**
```javascript
swapRatio = swapHolders / totalHolders

if (swapRatio between 0.80 and 0.95) → 10 points  // Organic Growth
else if (swapRatio >= 0.70)          → 8 points   // Mostly Organic
else if (swapRatio >= 0.60)          → 6 points   // Mixed
else                                 → 3 points   // Artificial (airdrops/transfers)
```

### 4. Security Score (0-100)

Evaluates contract security and risk factors based on SolSniffer audits.

#### Calculation Breakdown

**Base Security Score**
- Starts with SolSniffer audit score (0-100)
- Note: SolSniffer provides risk scores where lower is better, so we invert: `100 - riskScore`

**Security Features (Bonuses)**
```javascript
if (mintDisabled)    → +10 points  // Cannot mint new tokens
if (freezeDisabled)  → +10 points  // Cannot freeze accounts
if (lpBurned)       → +8 points   // Liquidity permanently locked
```

**Risk Penalties**
```javascript
if (top10HoldersControl > 50%)  → -15 points  // Concentration risk
highRiskCount * 8                → Deduction   // Each high risk -8 points
moderateRiskCount * 3             → Deduction   // Each moderate risk -3 points
// Low risks don't penalize
```

**Final Score**
```javascript
securityScore = Math.max(0, Math.min(100, 
    baseScore + bonuses - penalties
))
```

### 5. Momentum Score (0-100)

Evaluates price trends and market momentum.

#### Calculation Breakdown

**Price Performance (50% weight)**
```javascript
// Recent performance weighted more heavily
priceChange1h  * 0.15  // 15% weight
priceChange4h  * 0.20  // 20% weight
priceChange24h * 0.35  // 35% weight
priceChange7d  * 0.30  // 30% weight

// Scoring based on performance ranges
if (change > 20%)        → High positive score
else if (change > 5%)    → Moderate positive score
else if (change > -5%)   → Neutral score
else if (change > -20%)  → Moderate negative score
else                     → Low score
```

**Volume Momentum (30% weight)**
```javascript
if (volumeChange24h > 100%)      → 30 points  // Explosive growth
else if (volumeChange24h > 50%)  → 25 points  // Strong growth
else if (volumeChange24h > 0%)   → 20 points  // Growing
else if (volumeChange24h > -25%) → 10 points  // Declining
else                             → 5 points   // Collapsing
```

**Trader Momentum (20% weight)**
```javascript
uniqueWalletsChange24h scoring:
if (change > 50%)       → 20 points  // Viral growth
else if (change > 20%)  → 15 points  // Strong interest
else if (change > 0%)   → 10 points  // Growing interest
else                    → 5 points   // Declining interest
```

## Overall Score Calculation

### Weighted Average Method

```javascript
function calculateOverallScore(scores) {
    const weights = {
        liquidityScore: 0.25,   // 25% weight
        activityScore: 0.20,    // 20% weight
        communityScore: 0.20,   // 20% weight
        securityScore: 0.25,    // 25% weight
        momentumScore: 0.10     // 10% weight
    };
    
    let totalScore = 0;
    let totalWeight = 0;
    
    for (const [metric, weight] of Object.entries(weights)) {
        if (scores[metric] !== undefined) {
            totalScore += scores[metric] * weight;
            totalWeight += weight;
        }
    }
    
    // Normalize if not all scores available
    return totalWeight > 0 ? 
        Math.round(totalScore / totalWeight) : 0;
}
```

### Score Interpretation

| Overall Score | Rating | Interpretation |
|--------------|---------|----------------|
| 80-100 | Excellent | High-quality token with strong fundamentals |
| 60-79 | Good | Solid token with positive indicators |
| 40-59 | Moderate | Mixed signals, requires careful evaluation |
| 20-39 | Poor | Significant risks or weaknesses |
| 0-19 | Very Poor | High risk, multiple red flags |

### Action Recommendations

Based on the overall score and component analysis:

```javascript
function getRecommendation(score, components) {
    if (score >= 80 && components.securityScore >= 70) {
        return 'BUY';  // Strong candidate
    } else if (score >= 60 && components.securityScore >= 50) {
        return 'HOLD';  // Monitor for improvements
    } else {
        return 'AVOID';  // Too risky
    }
}
```

## Data Sources

### Real-time Data Providers
- **BirdEye**: Price, volume, liquidity, trading metrics
- **Moralis**: Holder analytics, distribution data
- **SolSniffer**: Security audits, contract analysis
- **LunarCrush**: Social sentiment (when available)

### Update Frequency
- Price/Volume data: Every 30 seconds
- Holder data: Every 5 minutes
- Security audits: On contract deployment, then cached
- Score recalculation: On data update

## Important Considerations

### Limitations
1. **New Tokens**: May have artificially low scores due to limited history
2. **Manipulation**: Short-term metrics can be gamed
3. **Data Availability**: Not all tokens have complete data
4. **Market Conditions**: Scores don't account for broader market trends

### Best Practices
1. **Use Multiple Metrics**: Don't rely on overall score alone
2. **Check Security First**: Low security score is a red flag regardless of other metrics
3. **Monitor Trends**: Score changes over time are often more informative than absolute values
4. **Verify Data**: Cross-reference with other sources for important decisions

## API Access

### Get Token Score
```http
GET /api/tokens/{tokenAddress}/score

Response:
{
    "address": "...",
    "symbol": "TOKEN",
    "scores": {
        "liquidityScore": 75,
        "activityScore": 82,
        "communityScore": 68,
        "securityScore": 90,
        "momentumScore": 71,
        "score": 77
    },
    "action": "HOLD",
    "lastUpdated": "2024-09-03T10:30:00Z"
}
```

### Bulk Score Request
```http
POST /api/tokens/scores
Body: {
    "addresses": ["address1", "address2", ...]
}
```

## Conclusion

AssetSwap's scoring methodology provides a comprehensive, transparent framework for evaluating tokens. By combining multiple data sources and applying consistent scoring logic, we help users make informed trading decisions based on quantifiable metrics rather than speculation.

The system is designed to be:
- **Transparent**: All calculations are documented and verifiable
- **Comprehensive**: Multiple dimensions capture different risk/opportunity aspects
- **Adaptive**: Scores update in real-time as market conditions change
- **Actionable**: Clear recommendations based on score thresholds

---

*This methodology is subject to updates and improvements based on market feedback and data availability.*