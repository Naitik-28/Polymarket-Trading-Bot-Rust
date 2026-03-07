# Polymarket Trading Bot V2

A high-performance Rust trading bot for [Polymarket](https://polymarket.com) prediction markets. Trades 15-minute and 5-minute price markets using limit orders, trailing stops, and hedge strategies.

---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [CLOB SDK Setup](#clob-sdk-setup)
- [Bot Strategies](#bot-strategies)
- [Configuration](#configuration)
- [Test Utilities](#test-utilities)
- [Security](#security)
- [Support](#support)

---


**Demo:** [Watch the bot in action] https://github.com/user-attachments/assets/4d697fc1-ca8d-4ff4-8e45-4bf5b8e4a71f



## Features

| Feature | Description |
|---------|-------------|
| **Dual Limit Same-Size** | Place Up/Down limit buys at $0.45 at market start; hedge with market buy if only one fills |
| **2-min / 4-min / Early / Standard Hedge** | Time-based and price-based hedge triggers with trailing stop |
| **5-Minute BTC Bot** | Specialized for BTC 5-minute markets with dual time windows |
| **Trailing Bot** | Wait for price &lt; 0.45, then trail with stop loss and trailing stop |
| **Backtest** | Replay strategy on historical price data |
| **Simulation Mode** | Test with live prices without placing real orders |

---

## Architecture

```
poly-bot-for-alche-rust-3/
├── src/
│   ├── api.rs          # Polymarket CLOB API client
│   ├── monitor.rs      # Market price monitoring
│   ├── trader.rs       # Order execution & hedge logic
│   ├── simulation.rs    # Simulation engine
│   └── bin/            # Executable entry points
│       ├── main_dual_limit_045_same_size.rs   # Default bot
│       ├── main_dual_limit_045_5m_btc.rs      # BTC 5-min bot
│       ├── main_trailing.rs                    # Trailing bot
│       └── backtest.rs                         # Backtest runner
├── config.json         # API keys & trading params (create from config.example.json)
└── history/            # Price history for backtesting
```

---

## Prerequisites

- **Rust** 1.70+ — [Install via rustup](https://rustup.rs)
- **Polymarket API** — API key, secret, passphrase from Polymarket
- **Polygon wallet** — Private key with USDC for trading
- **CLOB SDK** — Shared library for authentication (see below)

---

## Quick Start

```bash
# 1. Clone and build
git clone <repo-url>
cd poly-bot-for-alche-rust-3
cargo build --release

# 2. Configure
cp config.example.json config.json
# Edit config.json with your API keys and trading params

# 3. Run in simulation (no real orders)
cargo run -- --simulation

# 4. Run in production
cargo run -- --no-simulation
```

---

## CLOB SDK Setup

The bot uses a shared CLOB SDK library for Polymarket authentication. **You must build it with the `clob` feature:**

```bash
# Build the CLOB SDK (in polymarket-clob-sdk directory)
cd ../polymarket-clob-sdk
cargo build --release --features clob

# Copy the library to the bot's lib folder
cp target/release/libclob_sdk.so ../poly-bot-for-alche-rust-3/lib/

# Or set the path via environment variable
export LIBCOB_SDK_SO=/path/to/polymarket-clob-sdk/target/release/libclob_sdk.so
```

Without this, you'll see: `symbol clob_sdk_client_create not found`.

---

## Bot Strategies

### 1. Dual Limit Same-Size (Default)

**Binary:** `main_dual_limit_045_same_size`

At market start, places limit buys for BTC, ETH, SOL, and XRP Up/Down at $0.45. If **both** fill → done. If **only one** fills, hedges by buying the unfilled side at market (2-min trailing, 4-min, early, or standard hedge).

**Low-price exit:** After 10 minutes, if one side bid &lt; 0.10, places limit sells at $0.05/$0.99 (or $0.02/$0.99 when hedge price &lt; 0.60).

```bash
cargo run -- --simulation      # Simulation
cargo run -- --no-simulation  # Production
```

### 2. Dual Limit 5-Minute BTC

**Binary:** `main_dual_limit_045_5m_btc`

Same dual-limit logic for **BTC 5-minute markets only**. Uses 2-min (2–3 min) and 3-min (≥3 min) windows with trailing stop.

```bash
cargo run --bin main_dual_limit_045_5m_btc -- --simulation
```

### 3. Trailing Bot

**Binary:** `main_trailing`

Waits until one token's price is under $0.45, then trails with stop loss and trailing stop on the opposite side.

```bash
cargo run --bin main_trailing -- --simulation
```

### 4. Backtest

**Binary:** `backtest`

Replays strategy on `history/market_*_prices.toml` files.

```bash
cargo run --bin backtest -- --backtest
```

---

## Configuration

| Option | Description |
|--------|-------------|
| `--config <path>` | Config file (default: `config.json`) |
| `--simulation` | No real orders, live prices only |
| `--no-simulation` | Production mode, real orders |
| `--history-file <name>` | Replay from specific history file |

**Config structure:**

```json
{
  "polymarket": {
    "gamma_api_url": "https://gamma-api.polymarket.com",
    "clob_api_url": "https://clob.polymarket.com",
    "api_key": "...",
    "api_secret": "...",
    "api_passphrase": "...",
    "private_key": "0x...",
    "proxy_wallet_address": "0x...",
    "signature_type": 2
  },
  "trading": {
    "check_interval_ms": 500,
    "dual_limit_price": 0.45,
    "dual_limit_shares": 10,
    "enable_btc_trading": true,
    "enable_eth_trading": false,
    "dual_limit_hedge_after_minutes": 10,
    "dual_limit_hedge_price": 0.85,
    "trailing_stop_point": 0.02
  }
}
```

---

## Test Utilities

| Binary | Purpose |
|--------|---------|
| `test_allowance` | Check balance, set approval (`--approve-only`, `--list`) |
| `test_limit_order` | Place limit order (`--price-cents 60 --shares 10`) |
| `test_redeem` | List/redeem winning tokens (`--list`, `--redeem-all`) |
| `test_merge` | Merge complete sets to USDC |
| `test_sell` | Test market sell |
| `test_predict_fun` | Test prediction logic |

**One-time approval (required before selling):**

```bash
cargo run --bin test_allowance -- --approve-only
```

---

## Security

- **Never commit** `config.json` with real keys
- Use **simulation** and small sizes when testing
- Store keys in environment variables or secure vaults
- Monitor logs and balances in production


