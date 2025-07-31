import time
from binance.client import Client
from binance.enums import *
from decimal import Decimal

# API credentials (replace with your real keys)
API_KEY = 'a358632822a28ea0cc73c8e933ba8079fd44a4cf8eb2027c843f10fccbb8edd5'
API_SECRET = '835aaed0525a94b4f760aa8c94ce4ca127d9eab607514c504657ca902650d48b'

client = Client(API_KEY, API_SECRET)

# === CONFIGURATION ===
SYMBOL = 'BTCUSDT'
QUOTE_ASSET = 'USDT'
CLOSE_PROFIT_TARGET =1000
HEDGE_DISTANCE = 500
LOT_SIZES = [0.001, 0.001, 0.001, 0.001, 0.001, 0.001, 0.001, 0.001]
SLEEP_TIME = 10  # in seconds

# === VARIABLES ===
last_price = None
position_count = 0
base_order_side = 'BUY'  # or 'SELL'

# === FUNCTIONS ===

def get_price():
    ticker = client.get_symbol_ticker(symbol=SYMBOL)
    return float(ticker['price'])

def get_open_orders():
    return client.get_open_orders(symbol=SYMBOL)

def get_account_balance(asset):
    balance = client.get_asset_balance(asset=asset)
    return float(balance['free'])

def place_market_order(side, quantity):
    try:
        order = client.create_order(
            symbol=SYMBOL,
            side=SIDE_BUY if side == 'BUY' else SIDE_SELL,
            type=ORDER_TYPE_MARKET,
            quantity=quantity
        )
        print(f"Placed {side} order for {quantity} {SYMBOL}")
        return order
    except Exception as e:
        print(f"Order failed: {e}")
        return None

def get_total_unrealized_profit():
    # Binance SPOT API does not support PnL tracking easily.
    # You would need to simulate based on open orders + historical price
    return 0  # Placeholder

def close_all_positions():
    # On spot trading, you "sell" to close a long, or "buy" to close a short
    print("Closing all positions (manual logic required)")
    # Example: you can manually place the opposite trade

def calculate_next_lot(index):
    if index < len(LOT_SIZES):
        return LOT_SIZES[index]
    else:
        return LOT_SIZES[-1]

# === MAIN LOOP ===

def main():
    global last_price, position_count, base_order_side

    while True:
        try:
            current_price = get_price()
            print(f"Current Price: {current_price}")

            if last_price is None:
                last_price = current_price
                time.sleep(SLEEP_TIME)
                continue

            price_diff = abs(current_price - last_price)

            # Hedge logic
            if price_diff >= HEDGE_DISTANCE:
                lot = calculate_next_lot(position_count)
                place_market_order(base_order_side, lot)
                position_count += 1
                last_price = current_price

            # Close all logic
            total_profit = get_total_unrealized_profit()
            if total_profit >= CLOSE_PROFIT_TARGET:
                close_all_positions()
                position_count = 0
                last_price = current_price

            time.sleep(SLEEP_TIME)

        except KeyboardInterrupt:
            print("Bot stopped manually.")
            break
        except Exception as e:
            print(f"Error: {e}")
            time.sleep(SLEEP_TIME)

if __name__ == '__main__':
    main()
