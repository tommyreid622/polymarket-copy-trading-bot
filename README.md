# Polymarket Copy Trading Bot

A sophisticated, production-ready copy trading bot for Polymarket that automatically mirrors trades from target wallets in real-time. Built with TypeScript, leveraging WebSocket connections for low-latency trade execution and integrated with Polymarket's CLOB (Central Limit Order Book) API.

## 🎯 Overview

This bot monitors specified wallet addresses on Polymarket and automatically replicates their trading activity with configurable parameters. It provides real-time trade copying, automatic position redemption, risk management, and comprehensive logging for production deployment.

### Key Capabilities

- **Real-time Trade Mirroring**: Monitors target wallets via WebSocket and executes trades within milliseconds
- **Automated Redemption**: Automatically redeems winning positions from resolved markets
- **Risk Management**: Configurable size multipliers, maximum order amounts, and negative risk protection
- **Order Type Flexibility**: Supports FAK (Fill-or-Kill) and FOK (Fill-or-Kill) order types
- **Holdings Tracking**: Maintains local database of token holdings for efficient redemption
- **Multi-market Support**: Handles binary and multi-outcome markets seamlessly

## 🏗️ Architecture

### Technology Stack

- **Runtime**: Bun (TypeScript-first runtime)
- **Language**: TypeScript 5.9+
- **Blockchain**: Polygon (Ethereum-compatible L2)
- **APIs**: 
  - Polymarket CLOB Client (`@polymarket/clob-client`)
  - Polymarket Real-Time Data Client (`@polymarket/real-time-data-client`)
- **Web3**: Ethers.js v6 for blockchain interactions
- **Logging**: Custom logger with structured output

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Real-Time Data Client                     │
│              (WebSocket Connection to Polymarket)            │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                      Trade Monitor                           │
│  - Filters trades by target wallet address                  │
│  - Validates trade payloads                                  │
│  - Triggers copy trade execution                            │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   Trade Order Builder                       │
│  - Converts trade payloads to market orders                  │
│  - Applies size multipliers and risk limits                  │
│  - Handles order type conversion (FAK/FOK)                   │
│  - Manages tick size precision                               │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                      CLOB Client                            │
│  - Executes orders on Polymarket                            │
│  - Manages allowances and approvals                         │
│  - Tracks wallet balances                                   │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   Holdings Manager                           │
│  - Tracks token positions                                   │
│  - Maintains local JSON database                            │
│  - Enables efficient redemption                              │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  Redemption Engine                          │
│  - Monitors market resolution status                         │
│  - Automatically redeems winning positions                   │
│  - Supports scheduled and on-demand redemption              │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Installation

### Prerequisites

- **Bun** runtime (v1.0+): [Install Bun](https://bun.sh)
- **Node.js** 18+ (for npm package management)
- **Polygon wallet** with USDC for trading
- **Polymarket API credentials** (API key and secret)

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd polymarket-copytrading
   ```

2. **Install dependencies**
   ```bash
   bun install
   ```

3. **Configure environment variables**
   ```bash
   cp .env.example .env
   ```
   
   Edit `.env` with your configuration:
   ```env
   # Wallet Configuration
   PRIVATE_KEY=your_private_key_here
   TARGET_WALLET=0x... # Wallet address to copy trades from
   
   # Trading Configuration
   SIZE_MULTIPLIER=1.0
   MAX_ORDER_AMOUNT=100
   ORDER_TYPE=FAK
   TICK_SIZE=0.01
   NEG_RISK=false
   ENABLE_COPY_TRADING=true
   
   # Redemption Configuration
   REDEEM_DURATION=60 # Minutes between auto-redemptions
   
   # API Configuration
   CHAIN_ID=137 # Polygon mainnet
   CLOB_API_URL=https://clob.polymarket.com
   ```

4. **Initialize credentials**
   ```bash
   bun src/index.ts
   ```
   The bot will automatically create API credentials on first run.

## ⚙️ Configuration

### Environment Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `PRIVATE_KEY` | string | **required** | Private key of trading wallet |
| `TARGET_WALLET` | string | **required** | Wallet address to copy trades from |
| `SIZE_MULTIPLIER` | number | `1.0` | Multiplier for trade sizes (e.g., `2.0` = 2x size) |
| `MAX_ORDER_AMOUNT` | number | `undefined` | Maximum USDC amount per order |
| `ORDER_TYPE` | string | `FAK` | Order type: `FAK` or `FOK` |
| `TICK_SIZE` | string | `0.01` | Price precision: `0.1`, `0.01`, `0.001`, `0.0001` |
| `NEG_RISK` | boolean | `false` | Enable negative risk (allow negative balances) |
| `ENABLE_COPY_TRADING` | boolean | `true` | Enable/disable copy trading |
| `REDEEM_DURATION` | number | `null` | Minutes between auto-redemptions (null = disabled) |
| `CHAIN_ID` | number | `137` | Blockchain chain ID (137 = Polygon) |
| `CLOB_API_URL` | string | `https://clob.polymarket.com` | CLOB API endpoint |

### Trading Parameters

- **Size Multiplier**: Scales the copied trade size. `1.0` = exact copy, `2.0` = double size, `0.5` = half size
- **Max Order Amount**: Safety limit to prevent oversized positions. Orders exceeding this amount are rejected
- **Order Type**:
  - `FAK` (Fill-and-Kill): Partial fills allowed, remaining unfilled portion cancelled
  - `FOK` (Fill-or-Kill): Entire order must fill immediately or cancelled
- **Tick Size**: Price precision for order placement. Must match market's tick size
- **Negative Risk**: When enabled, allows orders that may result in negative USDC balance

## 🚀 Usage

### Starting the Bot

```bash
# Start copy trading bot
bun src/index.ts

# Or using npm script
npm start
```

The bot will:
1. Initialize WebSocket connection to Polymarket
2. Subscribe to trade activity feed
3. Monitor target wallet for trades
4. Automatically copy trades when detected
5. Run scheduled redemptions (if enabled)

### Manual Redemption

#### Redeem from Holdings File
```bash
# Redeem all resolved markets from token-holding.json
bun src/auto-redeem.ts

# Dry run (preview only)
bun src/auto-redeem.ts --dry-run

# Clear holdings after redemption
bun src/auto-redeem.ts --clear-holdings
```

#### Redeem from API
```bash
# Fetch all markets from API and redeem winning positions
bun src/auto-redeem.ts --api

# Limit number of markets checked
bun src/auto-redeem.ts --api --max 500
```

#### Redeem Specific Market
```bash
# Check market status
bun src/redeem.ts --check <conditionId>

# Redeem specific market
bun src/redeem.ts <conditionId>

# Redeem with specific index sets
bun src/redeem.ts <conditionId> 1 2
```

## 🔧 Technical Details

### Trade Execution Flow

1. **Trade Detection**: WebSocket receives trade activity message
2. **Wallet Filtering**: Validates trade originates from target wallet
3. **Order Construction**: Converts trade payload to market order:
   - Applies size multiplier
   - Validates against max order amount
   - Adjusts price to tick size
   - Sets order type (FAK/FOK)
4. **Balance Validation**: Checks sufficient USDC/token balance
5. **Allowance Management**: Ensures proper token approvals
6. **Order Execution**: Submits order to CLOB API
7. **Holdings Update**: Records token positions locally
8. **Logging**: Logs all operations with structured output

### Redemption Mechanism

The bot maintains a local JSON database (`src/data/token-holding.json`) tracking all token positions. When markets resolve:

1. **Resolution Check**: Queries Polymarket API for market status
2. **Winning Detection**: Identifies winning outcome tokens
3. **Balance Verification**: Confirms user holds winning tokens
4. **Redemption Execution**: Calls Polymarket redemption contract
5. **Holdings Cleanup**: Removes redeemed positions from database

### Security Features

- **Credential Management**: Secure API key storage in `src/data/credential.json`
- **Allowance Control**: Automatic USDC approval management
- **Balance Validation**: Pre-order balance checks prevent over-trading
- **Error Handling**: Comprehensive error handling with graceful degradation
- **Private Key Security**: Uses environment variables (never hardcoded)

### Order Builder Logic

The `TradeOrderBuilder` class handles complex order construction:

```typescript
class TradeOrderBuilder {
  async copyTrade(options: CopyTradeOptions): Promise<CopyTradeResult> {
    // 1. Extract trade parameters
    // 2. Apply size multiplier
    // 3. Validate against max amount
    // 4. Convert to market order format
    // 5. Handle buy vs sell logic
    // 6. Execute order
    // 7. Update holdings
  }
}
```

**Buy Orders**:
- Validates USDC balance
- Checks allowance and approves if needed
- Places market order
- Records token holdings

**Sell Orders**:
- Validates token holdings
- Checks available balance (accounting for open orders)
- Places market order
- Updates holdings after execution

## 📁 Project Structure

```
polymarket-copytrading/
├── src/
│   ├── index.ts                 # Main bot entry point
│   ├── auto-redeem.ts           # Automated redemption script
│   ├── redeem.ts                # Manual redemption script
│   ├── data/                    # Data storage
│   │   ├── credential.json      # API credentials (auto-generated)
│   │   └── token-holding.json  # Token holdings database
│   ├── order-builder/           # Order construction logic
│   │   ├── builder.ts           # TradeOrderBuilder class
│   │   ├── helpers.ts           # Order conversion utilities
│   │   └── types.ts             # Type definitions
│   ├── providers/               # API clients
│   │   ├── clobclient.ts        # CLOB API client
│   │   ├── wssProvider.ts       # WebSocket provider
│   │   └── rpcProvider.ts       # RPC provider
│   ├── security/                # Security utilities
│   │   ├── allowance.ts         # Token approval management
│   │   └── createCredential.ts # Credential generation
│   └── utils/                   # Utility functions
│       ├── balance.ts           # Balance checking
│       ├── holdings.ts          # Holdings management
│       ├── logger.ts            # Logging utility
│       ├── redeem.ts            # Redemption logic
│       └── types.ts             # TypeScript types
├── package.json
├── tsconfig.json
└── README.md
```

## 🔌 API Integration

### Polymarket CLOB Client

The bot uses the official `@polymarket/clob-client` for order execution:

```typescript
import { ClobClient, OrderType, Side } from "@polymarket/clob-client";

const client = await getClobClient();
const order = await client.createOrder({
  token_id: tokenId,
  side: Side.BUY,
  price: price,
  size: size,
  order_type: OrderType.FAK,
});
```

### Real-Time Data Client

WebSocket connection for live trade monitoring:

```typescript
import { RealTimeDataClient } from "@polymarket/real-time-data-client";

client.subscribe({
  subscriptions: [{
    topic: "activity",
    type: "trades"
  }]
});
```

## 📊 Monitoring & Logging

The bot provides comprehensive logging:

- **Trade Detection**: Logs all detected trades from target wallet
- **Order Execution**: Records order placement and results
- **Redemption Activity**: Tracks redemption operations
- **Error Handling**: Detailed error messages with stack traces
- **Balance Updates**: Displays wallet balances after operations

Log levels:
- `info`: General operational messages
- `success`: Successful operations
- `warning`: Non-critical issues
- `error`: Errors requiring attention

## ⚠️ Risk Considerations

1. **Market Risk**: Copy trading amplifies both gains and losses
2. **Liquidity Risk**: Large orders may not fill completely
3. **Slippage**: Market orders execute at current market price
4. **Gas Costs**: Each transaction incurs Polygon gas fees
5. **API Limits**: Rate limiting may affect order execution
6. **Network Latency**: WebSocket delays may cause missed trades

**Recommendations**:
- Start with small size multipliers
- Set conservative max order amounts
- Monitor wallet balance regularly
- Use dry-run mode for testing
- Test with small amounts before scaling

## 🛠️ Development

### Building

```bash
# Type checking
bun run tsc --noEmit

# Run in development
bun --watch src/index.ts
```

### Testing

```bash
# Test redemption (dry run)
bun src/auto-redeem.ts --dry-run

# Test specific market
bun src/redeem.ts --check <conditionId>
```

## 📝 License

ISC

## 🤝 Contributing

Contributions welcome! Please ensure:
- Code follows TypeScript best practices
- All functions are properly typed
- Error handling is comprehensive
- Logging is informative
- Documentation is updated

## 📞 Support

For issues, questions, or contributions:
- Open an issue on GitHub
- Review existing documentation
- Check Polymarket API documentation

---

**Disclaimer**: This software is provided as-is. Trading cryptocurrencies and prediction markets carries significant risk. Use at your own discretion and never trade more than you can afford to lose.

