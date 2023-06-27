import ccxt
import talib

# Connect to Binance exchange using CCXT library
exchange = ccxt.binance({
    'apiKey': 'YOUR_API_KEY',
    'secret': 'YOUR_SECRET_KEY',
    'enableRateLimit': True
})

# Define MACD parameters
fast_period = 12
slow_period =26
signal_period = 9

# Define trading variables
btc_balance = 1.0
usdt_balance = 1000.0
position = None

# Initialize trading loop
while True:
    # Get BTCUSDT 1-hour candlestick data
    ohlcv = exchange.fetch_ohlcv('BTC/USDT', '1h')

    # Extract close prices
    closes = [ohlcv[i][4] for i in range(len(ohlcv))]

    # Calculate MACD indicator
    macd, signal, hist = talib.MACD(closes, fastperiod=fast_period, slowperiod=slow_period, signalperiod=signal_period)

    # Check MACD crossover
    if macd[-1] > signal[-1] and macd[-2] <= signal[-2]:
        # Buy BTC with USDT
        if position == 'sell':
            amount = usdt_balance / closes[-1]
            exchange.create_market_buy_order('BTC/USDT', amount)
            btc_balance += amount
            usdt_balance = 0.0
            position = 'buy'
            print('Bought', amount, 'BTC at', closes[-1], 'USDT')

    elif macd[-1] < signal[-1] and macd[-2] >= signal[-2]:
        # Sell BTC for USDT
        if position == 'buy':
            amount = btc_balance
            exchange.create_market_sell_order('BTC/USDT', amount)
            usdt_balance += amount * closes[-1]
            btc_balance = 0.0
            position = 'sell'
            print('Sold', amount, 'BTC at', closes[-1], 'USDT')

    # Wait for next candlestick
    exchange.sleep(3600)
