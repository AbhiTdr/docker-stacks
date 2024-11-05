
```
import pandas as pd
import yfinance as yf
import time

# Define parameters
bank_nifty_symbol = "^NSEBANK"
atm_ce_pe_symbol_suffix = "CE"  # and "PE"
entry_time = "09:30:00"
exit_time = "15:15:00"
target_percentage = 1.00
stop_loss_percentage = 0.25
profit_percentage = 0.15

# Function to get current market price (CMP)
def get_cmp(symbol):
    return yf.Ticker(symbol).history(period="1d", interval="1m")['Close'].iloc[-1]

# Function to check trade conditions
def check_trade_conditions(cmp, atm_ce_pe):
    if atm_ce_pe == "CE":
        return cmp > (get_cmp(bank_nifty_symbol) * (1 + profit_percentage))
    elif atm_ce_pe == "PE":
        return cmp > (get_cmp(bank_nifty_symbol) * (1 - profit_percentage))

# Function to execute trade
def execute_trade(atm_ce_pe, cmp):
    # Buy option
    entry_price = cmp
    stop_loss = entry_price * (1 - stop_loss_percentage)
    target_price = entry_price * (1 + target_percentage)
    print(f"Buying {atm_ce_pe} at {entry_price}")
    
    # Monitor and exit
    while True:
        current_price = get_cmp(atm_ce_pe)
        if current_price >= target_price or current_price <= stop_loss or time.strftime("%H:%M:%S") >= exit_time:
            if current_price >= target_price:
                print(f"Target hit! Selling {atm_ce_pe} at {current_price}")
            elif current_price <= stop_loss:
                print(f"Stop loss hit! Selling {atm_ce_pe} at {current_price}")
            else:
                print(f"Exiting {atm_ce_pe} at {current_price} due to time")
            break
        time.sleep(60)  # Check every minute

# Main loop
while True:
    current_time = time.strftime("%H:%M:%S")
    if current_time == entry_time:
        # Get ATM CE and PE symbols
        atm_ce_symbol = f"{bank_nifty_symbol}{atm_ce_pe_symbol_suffix}{time.strftime('%d%m%y')}"
        atm_pe_symbol = f"{bank_nifty_symbol}{atm_ce_pe_symbol_suffix.replace('CE', 'PE')}{time.strftime('%d%m%y')}"
        
        # Get CMP of ATM CE and PE
        atm_ce_cmp = get_cmp(atm_ce_symbol)
        atm_pe_cmp = get_cmp(atm_pe_symbol)
        
        # Check trade conditions
        if check_trade_conditions(atm_ce_cmp, "CE"):
            execute_trade(atm_ce_symbol, atm_ce_cmp)
        if check_trade_conditions(atm_pe_cmp, "PE"):
            execute_trade(atm_pe_symbol, atm_pe_cmp)
    
    time.sleep(60)  # Check every minute
```
