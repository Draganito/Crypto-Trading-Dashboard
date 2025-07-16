# Crypto Trading Dashboard
https://draganito.github.io/Crypto-Trading-Dashboard/

A web-based application for cryptocurrency trading on Binance Futures, featuring interactive charts, technical indicators (RSI, OBV-MACD, MACD), risk management tools, custom scripting for automated trades, and direct order execution.

## Features
- **Interactive Charts**: Candlestick charts with live price updates, volume bars, and a 55-period SMA overlay.
- **Indicators**: Toggle between RSI (14-period), OBV-MACD (custom OBV smoothed with DEMA and MACD), and standard MACD (12/26/9).
- **Trading Controls**: Open long/short positions with configurable risk percentage, leverage, position size calculation, and liquidation price estimates. Supports SL (stop-loss), TP (take-profit), and RR (risk-reward) ratios.
- **Positions Table**: Real-time display of open positions with details like unrealized profit, liquidation price, SL/TP, and a "Close" button.
- **Scripting**: Simple if-then-else scripting language for conditional trades based on indicators and price data.
- **API Integration**: Connects to Binance Futures API for fetching data, placing orders, and managing positions.
- **Settings**: API key management, indicator toggles, and option to enforce Isolated Margin Mode (default: enabled).
- **Other**: Symbol selection sorted by volume, interval choices (1m to 1M), daily change percentage, and error/success messaging.

## Installation & Setup
1. Download the `index.html` file and open it in a modern web browser (e.g., Chrome, Firefox).
2. In the "Settings" tab, enter your Binance API Key and Secret for real trading (test mode uses a fake 1000 USDT balance if no keys are provided).
3. Select a symbol (e.g., BTCUSDT) and interval (e.g., 1h) from the top bar.
4. Ensure your Binance account is set to **Hedge Mode** (required for long/short position sides). You can enable it in the Binance app or via API (the app assumes it's active but does not enforce it automatically).
5. For trading: Use the "Trade" tab to configure risk/leverage and open positions.

**Note**: The app requires an internet connection for API calls. No additional installations needed (uses CDN for CryptoJS).

## Important Warnings and Disclaimer
**This software is provided as a proof of concept and is not intended for production use without additional security measures and thorough testing.** Trading cryptocurrencies involves significant risks, including the potential for complete loss of capital. Market volatility, technical issues, or API errors can lead to financial losses.

- **No Liability**: The author assumes no responsibility or liability for any errors, omissions, or losses incurred from using this app. You are solely responsible for your trading decisions and must verify all calculations and API responses independently.
- **Risks of Crypto Trading**: Cryptocurrencies are highly volatile. Do not invest money you cannot afford to lose. Always conduct your own research and consider consulting a financial advisor.
- **API Keys Security**: API keys are stored unencrypted in browser memory (JavaScript variables). This poses a security risk, as they could be exposed via browser dev tools, XSS attacks, or malware. Do not use production keys on unsecured devices. Consider using a server-side proxy for keys in a real-world setup.
- **Hedge Mode Requirement**: Binance must be in Hedge Mode for the app to function correctly (allows separate long/short positions). Check and enable it in your Binance account settings before trading.
- **Proof of Concept**: This app is experimental. It may contain bugs, incomplete features, or fail under certain conditions (e.g., rate limits, network issues). Use with caution, start with small test trades on Binance Testnet if possible, and implement your own safeguards (e.g., manual confirmations, position limits).
- **Usage Guidelines**: Only use this app if you understand the code and risks. Monitor trades closely, and never leave it running unattended. Commercial use is allowed under the MIT License, but ensure compliance with Binance's Terms of Service.

By using this software, you agree to these terms and accept all associated risks.

## Technical Implementation
This app is built as a single HTML file with vanilla JavaScript (no frameworks), using Canvas for charting and CryptoJS for API signing. It's modularized with Immediately Invoked Function Expressions (IIFE) for encapsulation. Key components:

### Architecture
- **Modules**:
  - **UtilsModule**: Handles fetch retries, time formatting, and candle expiration checks.
  - **EventEmitter**: Pub/sub system for loose coupling (e.g., events like "chart-data-updated", "order-placed").
  - **ErrorHandlerModule**: Centralized logging and UI messages for errors/successes.
  - **VolumeSortModule**: Fetches and sorts symbols by 24h volume from Binance exchangeInfo and ticker/24hr.
  - **ApiModule**: Manages Binance API calls (klines, prices, account, positions, orders). Includes signing with HMAC-SHA256, caching for prices (3s timeout), and methods like setLeverage, changeMarginType (for Isolated mode).
  - **TradeModule**: Core trading logic – calculates quantities, liquidation prices (using MMR tiers), places orders (market, SL/TP), closes positions. Supports risk-based sizing, leverage brackets, and Isolated mode toggle.
  - **UiModule**: DOM manipulation – renders charts, tabs, inputs, positions table, messages. Handles inputs throttling (100ms).
  - **ChartDataModule**: Fetches/processes klines, computes indicators (SMA, RSI, OBV-MACD, MACD, avg volume), manages live candle updates (every 500ms).
  - **ChartModule**: Canvas rendering – candlesticks, volume, indicators, crosshairs, zoom/pan. Uses caching for static elements to optimize redraws.
- **Data Flow**: Init fetches symbols/constraints, sets up events. Chart updates poll prices, recompute indicators. Trades use ApiModule for signed requests.
- **Configs**: Centralized in `CONFIG` object (e.g., periods, limits).
- **Security Notes**: No server-side; all client-side. API keys in memory only (cleared on invalid auth).
- **Performance**: Throttling, requestAnimationFrame, caching to handle live updates without lag.
- **Dependencies**: Only CryptoJS (CDN-loaded for HMAC).

For developers: The code is self-contained in the `<script>` tag. Extend by adding modules or events. Run locally by opening `index.html`.

## Scripting Language
The app includes a simple scripting system in the "Script" tab for conditional trades. Scripts run once or in a loop (every 1s, stops on successful trade). Format: `if(condition;then_action;else_action)` (or `wenn` for German).

### Condition
- Syntax: Variable-operator-value, chained with `&` (AND only, no OR).
- Operators: `>`, `<`, `>=`, `<=`, `==`, `!=`.
- Variables (from latest candle, including live if available):
  - `rsi`: RSI (14-period).
  - `lastclose`: Close price.
  - `sma55`: 55-period SMA.
  - `open`: Open price.
  - `high`: High price.
  - `low`: Low price.
  - `volume`: Volume.
  - `previousclose`: Close of previous candle.
  - `macd`: OBV-MACD value.
  - `signal`: OBV-MACD signal line.
  - `stdmacd`: Standard MACD value.
  - `stdsignal`: Standard MACD signal line.
  - `averagevolume`: 14-period average volume.
- Right side: Number or another variable (e.g., `rsi>50` or `lastclose>sma55`).
- Example: `rsi>50&lastclose>sma55&volume>averagevolume`

### Actions
- `long(risk%risk@leveragex[,sl=value% or abs][,tp=value% or abs][,rr=ratio])` or `short(...)`: Opens position.
  - `risk`: Risk percentage (0-100).
  - `leverage`: Leverage (1-max from API).
  - `sl`: Stop-loss (% from entry or absolute price).
  - `tp`: Take-profit (% from entry or absolute price).
  - `rr`: Risk-reward ratio (adjusts TP for net profit after fees).
- `donothing`: No action.
- Example: `long(1%risk@50x,sl=2%,rr=3)`

### Full Examples
- `if(rsi>50&lastclose>sma55;long(1%risk@50x,rr=5);donothing)`: Long if RSI>50 and close>SMA55, else nothing.
- `if(macd>signal&stdmacd>0;short(2%risk@20x,sl=1.5%,tp=3%);donothing)`: Short if OBV-MACD crossover and standard MACD positive.
- `if(volume>averagevolume&high>previousclose;long(5%risk@10x,sl=10000);donothing)`: Long with absolute SL at 10000.

Scripts evaluate on current data; errors show in messages. Extend by modifying `executeCustomScript` function.

## Development Note
This software was developed over weeks of work in collaboration with Grok, an AI built by xAI.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details. Commercial use is allowed, as long as the copyright notice is retained.

## Support / Donate
If you find this program cool and use it, I'd appreciate a small donation! It helps continue development.

- ETH address: 0x334e8A7d38a369C8Df4182ad4C44434582D70613.

Thank you! ❤️
Dragan Bojovic
bojovicd@proton.me
