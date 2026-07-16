# IB MCP Server

MCP server exposing Interactive Brokers data **and trading operations** via [`ib_async`](https://ib-api-reloaded.github.io/ib_async/) and [`FastMCP`](https://github.com/modelcontextprotocol/fastmcp).

Supports read-only mode (default) for safe data access, and a full trading mode (place/cancel/modify orders, bracket orders with OCA groups and trailing stops) when explicitly enabled.

> Forked from [Hellek1/ib-mcp](https://github.com/Hellek1/ib-mcp) v0.2.12 (BSD-3-Clause), extended with trading capabilities.

## Features

### Read Tools (always available)

#### Contract Lookup & Conversion
- **lookup_contract**: Look up contract details by ticker symbol and optional exchange/currency
- **ticker_to_conid**: Convert ticker symbol to contract ID (conid)
- **search_contracts**: Search for contracts by partial symbol or company name

#### Market Data
- **get_historical_data**: Retrieve historical market data with configurable duration, bar size, and data type

#### Options
- **get_option_chain**: List option-chain parameters (expirations and strikes) for an underlying, with strike-range filtering and pagination
- **get_option_quotes**: Batch bid/ask/last/close and model IV/delta for a list of option strikes; defaults to delayed-frozen data (no OPRA subscription required)
- **get_index_quote**: Spot quote (last/close/bid/ask) for an index, useful for strike-from-spot calculations

#### News
- **get_historical_news**: Retrieve historical news articles within a date range
- **get_article**: Retrieve a full news article by ID and provider code

#### Fundamental Data
- **get_fundamental_data**: Retrieve fundamental data including financial summaries, ownership, financial statements, and more

#### Portfolio & Account
- **get_account_summary**: Retrieve account summary information
- **get_positions**: Retrieve current positions with contract metadata, including option expiry/strike/right/multiplier fields
- **get_contract_details**: Get detailed contract information including dividends and corporate actions

### Trading Tools (require `readonly=false`)

#### Order Management
- **place_order**: Place a buy/sell order (MKT, LMT, STOP, STP_LMT) with configurable TIF (DAY, GTC, IOC, GTD) and account routing
- **place_bracket_order**: Place a 3-leg bracket order (entry LMT + take-profit LMT + stop-loss STOP) with parent/child linking, OCA group, and optional trailing stop
- **modify_order**: Modify an open order's quantity, price, or order type
- **cancel_order**: Cancel an open order by order ID

#### Order & Execution Queries
- **get_open_orders**: List open orders (this client or all clients)
- **get_trades**: Get recent execution history (fills) with optional symbol filter

## Prerequisites

1. **Interactive Brokers Account**: You need an active IB account
2. **IB Gateway or TWS**: Download and install either:
   - [IB Gateway (Stable)](https://www.interactivebrokers.com/en/trading/ibgateway-stable.php) - Recommended for API-only use
   - [IB Gateway (Latest)](https://www.interactivebrokers.com/en/trading/ibgateway-latest.php) - Latest features
   - [Trader Workstation (TWS)](https://www.interactivebrokers.com/en/trading/tws.php) - Full trading platform
3. **API Configuration**:
   - Enable API access in TWS/Gateway: `Configure → API → Settings` and check "Enable ActiveX and Socket Clients"
   - Set appropriate port (default: 7497 for TWS, 4001 for Gateway)
   - Add `127.0.0.1` to trusted IPs if connecting locally

## Installation

### From source (development)
```bash
git clone https://github.com/stavg91/ib-mcp.git
cd ib-mcp
pip install poetry
poetry install
```

## Usage

### Read-Only Mode (Default)

Safe for data retrieval — trading tools return an error.

```bash
# Default: read-only
poetry run ib-mcp-server --host 127.0.0.1 --port 4001 --client-id 100

# Or via env vars
IB_PORT=4001 IB_CLIENT_ID=100 poetry run ib-mcp-server
```

### Trading Mode

Enable order placement, modification, and cancellation:

```bash
# Via CLI flag
poetry run ib-mcp-server --port 4001 --client-id 100 --no-readonly

# Via env var
IB_MCP_READONLY=false IB_PORT=4001 IB_CLIENT_ID=100 poetry run ib-mcp-server
```

### STDIO Mode (Default)

The default mode runs as a spawnable MCP server communicating via standard input/output. This is ideal for integration with MCP clients like Claude Desktop or Hermes Agent.

### HTTP Mode

HTTP mode runs a persistent server that listens on a host and port, enabling multi-client access and network connectivity.

```bash
poetry run ib-mcp-server --transport http --http-host 127.0.0.1 --http-port 8000
```

### Command Line Options

#### IB Connection
- `--host`: IB Gateway/TWS host (default: 127.0.0.1, env: `IB_HOST`)
- `--port`: IB Gateway/TWS port (default: 7497, env: `IB_PORT`)
- `--client-id`: Unique client ID for the connection (default: 1, env: `IB_CLIENT_ID`)
- `--readonly` / `--no-readonly`: Read-only mode (default: true, env: `IB_MCP_READONLY`). Use `--no-readonly` to enable trading.

#### Transport
- `--transport`: `stdio` (default) or `http` (env: `IB_MCP_TRANSPORT`)
- `--http-host`: HTTP server host (default: 127.0.0.1, env: `IB_MCP_HTTP_HOST`)
- `--http-port`: HTTP server port (default: 8000, env: `IB_MCP_HTTP_PORT`)

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `IB_HOST` | `127.0.0.1` | IB Gateway/TWS host |
| `IB_PORT` | `7497` | IB Gateway/TWS port |
| `IB_CLIENT_ID` | `1` | Client ID |
| `IB_MCP_READONLY` | `true` | Read-only mode (`false` enables trading) |
| `IB_MCP_TRANSPORT` | `stdio` | Transport protocol |
| `IB_MCP_HTTP_HOST` | `127.0.0.1` | HTTP server host |
| `IB_MCP_HTTP_PORT` | `8000` | HTTP server port |

Flags override environment variables if both are provided.

## Available Tools

### Contract Lookup
```
lookup_contract(symbol, sec_type="STK", exchange="SMART", currency="USD")
ticker_to_conid(symbol, sec_type="STK", exchange="SMART", currency="USD")
search_contracts(pattern)
```

### Market Data
```
get_historical_data(symbol, duration="1 M", bar_size="1 day", data_type="TRADES", max_bars=20, exchange="SMART", currency="USD")
```

### Options
```
get_option_chain(symbol, sec_type="IND", exchange="CBOE", currency="USD", trading_class="", min_strike=0.0, max_strike=0.0, max_strikes=20, strike_offset=0)
get_option_quotes(symbol, expiry, right, strikes, exchange="SMART", currency="USD", trading_class="", use_delayed=True)
get_index_quote(symbol, exchange="CBOE", currency="USD", use_delayed=True)
```

### News
```
get_historical_news(symbol, start_date, end_date, max_count=10, exchange="SMART", currency="USD")
get_article(articleId, providerCode, as_markdown=True, truncate=0)
```

### Fundamentals
```
get_fundamental_data(symbol, report_type="ReportsFinSummary", exchange="SMART", currency="USD")
get_contract_details(symbol, sec_type="STK", exchange="SMART", currency="USD")
```

### Portfolio & Account
```bash
get_account_summary(account="")
get_positions(account="")
get_portfolio(account="")
```

### Trading *(requires readonly=false)*
```
place_order(symbol, action, quantity, order_type="MKT", limit_price=0, stop_price=0, tif="DAY", sec_type="STK", exchange="SMART", currency="USD", account="")
place_bracket_order(symbol, action, quantity, entry_price, take_profit_price, stop_loss_price=0, trail_amount=0, trail_percent=0, tif="GTC", sec_type="STK", exchange="SMART", currency="USD", account="")
modify_order(order_id, quantity=0, order_type="", limit_price=0, stop_price=0)
cancel_order(order_id)
get_open_orders(all_clients=False)
get_trades(symbol="", max_count=20)
```

#### Bracket Order Details

`place_bracket_order` creates a 3-leg parent/child group identical to IBKR's native bracket:

- **Entry** (parent): LMT order, `transmit=False`
- **Take-Profit** (child): LMT order, `parentId=entry`, `transmit=False`
- **Stop-Loss** (child): STOP or TRAIL order, `parentId=entry`, `transmit=True`

TP and SL are OCA-linked (`ocaType=1`) — when one fills, the other cancels.

If `trail_amount` or `trail_percent` is > 0, the stop-loss leg becomes a TRAIL order instead of a fixed STOP.

## Example Usage

Once connected to an LLM through MCP, you can ask questions like:

**Read-only:**
- "Look up the contract details for AAPL"
- "Get the last month of daily historical data for TSLA"
- "Show the XSP option chain strikes between 400 and 550"
- "Get put quotes for XSP expiry 20261218 at strikes 440, 450 and 460"
- "What are the recent news articles for Microsoft?"
- "Show me the financial summary for Google"
- "What positions do I currently have in my portfolio?"

**Trading (readonly=false):**
- "Buy 100 shares of AAPL at market"
- "Buy 100 shares of DAL at $25.50, take-profit at $27, stop-loss at $24"
- "Buy 50 shares of TSLA at $180 with a 5% trailing stop, take-profit at $200"
- "Cancel order 42"
- "Show my open orders"
- "Show recent fills for AAPL"

## Security Considerations

- **Read-only by default**: The server starts in `readonly=true` mode. Trading tools check the flag and return an error if true.
- **Explicit opt-in for trading**: Must set `IB_MCP_READONLY=false` or pass `--no-readonly`.
- Credentials are handled by the IB Gateway/TWS application.
- The server only accesses data you have permission to view in your IB account.
- Always test trading in a paper trading account before using with real money.

## Contributing

1. Fork & branch: `feat/xyz`
2. Install dev deps: `poetry install`
3. Run linter: `ruff check ib_mcp/ && ruff format ib_mcp/`
4. Run tests: `poetry run pytest -q`
5. Open a PR with a concise description.

## Support & References

- IB API functionality: [ib_async docs](https://ib-api-reloaded.github.io/ib_async/)
- MCP protocol: [MCP spec](https://spec.modelcontextprotocol.io/)
- Interactive Brokers: [IB API docs](https://ibkrcampus.com/ibkr-api-page/twsapi-doc/)

---

Licensed under the BSD 3-Clause License. Contributions welcome.
