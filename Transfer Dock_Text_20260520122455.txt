import yfinance as yf
import pandas as pd
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes

TOKEN = "حط_التوكن_بتاعك_هنا"

MARKETS = {
    "EUR/USD": "EURUSD=X",
    ‏    "GBP/USD": "GBPUSD=X", 
    ‏    "USD/JPY": "USDJPY=X",
    ‏    "BTC/USD": "BTC-USD"
    ‏}
def analyze_market(symbol):
    df = yf.download(symbol, period="1d", interval="1m", progress=False)
    ‏    if df.empty:
    ‏        return None, None
    ‏    
    ‏    df['MA20'] = df['Close'].rolling(window=20).mean()
    ‏    delta = df['Close'].diff()
    ‏    gain = (delta.where(delta > 0, 0)).rolling(window=14).mean()
    ‏    loss = (-delta.where(delta < 0, 0)).rolling(window=14).mean()
    ‏    rs = gain / loss
    ‏    df['RSI'] = 100 - (100 / (1 + rs))
    ‏    
    ‏    last_close = df['Close'].iloc[-1]
    ‏    last_ma = df['MA20'].iloc[-1]
    ‏    last_rsi = df['RSI'].iloc[-1]
    ‏    
    ‏    if last_rsi < 30 and last_close > last_ma:
    ‏        return "BUY", "1 دقيقة"
    ‏    elif last_rsi > 70 and last_close < last_ma:
    ‏        return "SELL", "1 دقيقة"
    ‏    else:
    ‏        return "WAIT", "انتظر"
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [[InlineKeyboardButton(k, callback_data=v)] for k, v in MARKETS.items()]
    ‏    reply_markup = InlineKeyboardMarkup(keyboard)
    ‏    await update.message.reply_text("اختر السوق:", reply_markup=reply_markup)
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    ‏    await query.answer()
    ‏    symbol = query.data
    ‏    
    ‏    await query.edit_message_text(f"جاري تحليل {symbol}...")
    ‏    signal, duration = analyze_market(symbol)
    ‏    
    ‏    if signal == "BUY":
    ‏        msg = f"🟢 إشارة شراء\nالسوق: {symbol}\nالمدة: {duration}"
    ‏    elif signal == "SELL":
    ‏        msg = f"🔴 إشارة بيع\nالسوق: {symbol}\nالمدة: {duration}"
    ‏    else:
    ‏        msg = f"⏸️ لا توجد إشارة واضحة\nالسوق: {symbol}"
    ‏    
    ‏    await query.edit_message_text(msg)
def main():
    app = Application.builder().token(TOKEN).build()
    ‏    app.add_handler(CommandHandler("start", start))
    ‏    app.add_handler(CallbackQueryHandler(button))
    ‏    app.run_polling()
if __name__ == "__main__":
    main()
