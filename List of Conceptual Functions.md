# List of Conceptual Functions

1. **`initialize_environment()`** *Complete*
   - **Description**: Sets up the runtime environment by loading configuration from a `.env` file and configuring logging.
   - **Inputs**: None (relies on environment variables).
   - **Outputs**: Tuple (bool, dict) - Success flag and dictionary of environment variables (app_key, app_secret, callback_url).
   - **Behavior**: Prepares foundational settings, loads `.env`, and validates variables.

2. **`start_client(app_key, app_secret, callback_url)`** *Complete*
   - **Description**: Creates and configures a Schwab API client instance using provided credentials.
   - **Inputs**: App key (str), app secret (str), callback URL (str).
   - **Outputs**: Tuple (bool, schwabdev.Client or None) - Success flag and client instance if successful.
   - **Behavior**: Initializes the Schwab API client, validates inputs, and handles exceptions.

3. **`initialize_portfolio(initial_cash)`** *Complete*
   - **Description**: Creates a simulated portfolio with an initial cash balance.
   - **Inputs**: Initial cash amount (float, default 100000.0).
   - **Outputs**: `Portfolio` object with cash and empty positions.
   - **Behavior**: Sets up a portfolio for simulation mode tracking.

4. **`fetch_account_hash(client, simulate)`** *Complete*
   - **Description**: Retrieves the account hash for real trading if simulation is disabled.
   - **Inputs**: Schwab client object, simulation flag (boolean).
   - **Outputs**: Account hash (string) or None if simulating.
   - **Behavior**: Queries linked accounts and extracts the hash for real trades.

5. **`request_account_details(client, account_hash)`** *Complete*
   - **Description**: Fetches detailed account information (e.g., balances, status) from Schwab.
   - **Inputs**: Schwab client object, account hash (string).
   - **Outputs**: Dictionary containing account details (e.g., cash balance, margin).
   - **Behavior**: Retrieves real account data via the Schwab API.

6. **`request_account_positions(client, account_hash)`** *Complete*
   - **Description**: Retrieves current positions (e.g., stock holdings) for the Schwab account.
   - **Inputs**: Schwab client object, account hash (string).
   - **Outputs**: List of position dictionaries (e.g., symbol, quantity, price).
   - **Behavior**: Queries real-time position data from Schwab.

7. **`load_initial_historical_data(client, symbols, indicators)`** *Complete*
   - **Description**: Fetches initial historical minute-level data for a list of symbols and passes it to the indicators object.
   - **Inputs**: Schwab client object, list of initial symbols, `Indicators` object.
   - **Outputs**: Boolean indicating success, `Indicators` object updated with historical data.
   - **Behavior**: Requests minute-level data and feeds it to `Indicators`, which handles aggregation internally.

8. **`start_streamer(client, response_handler, start_time, stop_time, days, timezone)`**
   - **Description**: Starts the Schwab streamer in a background thread with a schedule and callback.
   - **Inputs**: Schwab client, response handler function, start/stop times, active days, timezone.
   - **Outputs**: Running streamer thread.
   - **Behavior**: Initiates real-time data streaming during market hours.

9. **`subscribe_to_streams(streamer, initial_symbols)`**
   - **Description**: Subscribes to real-time chart and screener data streams for initial symbols.
   - **Inputs**: Streamer object, list of initial symbols.
   - **Outputs**: Updated streamer subscriptions.
   - **Behavior**: Sends subscription commands for CHART_EQUITY and SCREENER_EQUITY.

10. **`process_stream_messages(shared_list, client, simulate, portfolio, indicators)`**
    - **Description**: Processes incoming streamer messages and dispatches them to handlers.
    - **Inputs**: Shared message list, Schwab client, simulation flag, `Portfolio` object, `Indicators` object.
    - **Outputs**: Updated portfolio and indicators.
    - **Behavior**: Routes messages to handlers; `Indicators` aggregates data internally.

11. **`handle_chart_equity(service, client, simulate, portfolio, indicators)`**
    - **Description**: Processes CHART_EQUITY data, updates indicators, and executes trades.
    - **Inputs**: Service data, Schwab client, simulation flag, `Portfolio` object, `Indicators` object.
    - **Outputs**: Updated indicators and portfolio.
    - **Behavior**: Feeds minute-level data to `Indicators`, which aggregates and computes indicators; triggers trades based on results.

12. **`handle_screener_equity(service, streamer, indicators, portfolio, client)`**
    - **Description**: Processes SCREENER_EQUITY data, manages subscriptions, and feeds data to indicators.
    - **Inputs**: Service data, streamer object, `Indicators` object, `Portfolio` object, Schwab client.
    - **Outputs**: Updated subscriptions and indicators.
    - **Behavior**: Subscribes to promising symbols, unsubscribes from closed positions, and passes data to `Indicators` for aggregation.

13. **`update_indicators(symbol, close, volume, timestamp, indicators)`**
    - **Description**: Updates the `Indicators` object with new minute-level data, aggregating and recalculating indicators based on its internal time frame configuration.
    - **Inputs**: Stock symbol, close price, volume, timestamp, `Indicators` object.
    - **Outputs**: Updated `Indicators` object, dict of booleans indicating new aggregation periods (e.g., {'minute': True, 'hour': False}).
    - **Behavior**: Aggregates minute data into configured time frames (e.g., minute, hour) and updates indicators like RSI or SMA.

14. **`calculate_rsi(market_inputs, period)`**
    - **Description**: Calculates the Relative Strength Index (RSI) for given market data over a specified period.
    - **Inputs**: Market inputs (list of close prices or similar), period (int, e.g., 14).
    - **Outputs**: RSI value (float) or None if insufficient data.
    - **Behavior**: Computes RSI based on price changes over the specified period.

15. **`calculate_ma_crossover(market_inputs, short_period, long_period)`**
    - **Description**: Detects moving average crossovers (short over long) from market data.
    - **Inputs**: Market inputs (list of close prices), short period (int, e.g., 10), long period (int, e.g., 20).
    - **Outputs**: Tuple (bool for crossover up, bool for crossover down) or None if insufficient data.
    - **Behavior**: Calculates short and long MAs and checks for crossovers.

16. **`calculate_trading_action(symbol, indicators, *indicator_values)`**
    - **Description**: Determines buy, sell, or hold action using indicator values from the `Indicators` object.
    - **Inputs**: Stock symbol, `Indicators` object, variable args of indicator values (e.g., minutely RSI float, hourly SMA tuple).
    - **Outputs**: Action string ("buy", "sell", "hold").
    - **Behavior**: Evaluates trading decisions based on indicators computed across configured time frames.

17. **`execute_trade(action, symbol, price, quantity, client, simulate, portfolio, account_hash)`**
    - **Description**: Executes a buy or sell trade, either simulated or real.
    - **Inputs**: Action, symbol, price, quantity, Schwab client, simulation flag, `Portfolio` object, account hash.
    - **Outputs**: Updated portfolio or placed order.
    - **Behavior**: Updates portfolio in simulation or sends real market order.

18. **`report_portfolio_status(portfolio, interval, last_report_time)`**
    - **Description**: Periodically logs simulated portfolio gains/losses and cash balance.
    - **Inputs**: `Portfolio` object, report interval (float), last report time (float).
    - **Outputs**: Updated last report time, logged report.
    - **Behavior**: Logs portfolio status at specified intervals.

19. **`report_account_status(client, account_hash, interval, last_report_time)`**
    - **Description**: Periodically logs real account status (e.g., cash, positions) from Schwab.
    - **Inputs**: Schwab client, account hash, report interval (float), last report time (float).
    - **Outputs**: Updated last report time, logged account status.
    - **Behavior**: Fetches and logs actual account details and positions.

20. **`main_loop(shared_list, client, simulate, portfolio, indicators, report_interval)`**
    - **Description**: Runs the main event loop, processing messages and reporting statuses.
    - **Inputs**: Shared message list, Schwab client, simulation flag, `Portfolio` object, `Indicators` object, report interval.
    - **Outputs**: Continuous execution.
    - **Behavior**: Processes messages and generates periodic reports; `Indicators` manages aggregation internally.