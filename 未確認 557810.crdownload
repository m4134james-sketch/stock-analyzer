import streamlit as st
import yfinance as yf
import pandas as pd
import numpy as np
from datetime import datetime

st.set_page_config(page_title="股票走勢分析", page_icon="📈", layout="wide")

st.title("📈 股票走勢分析工具")
st.caption("輸入股票代碼即可分析（美股直接打代碼，港股加 .HK，例如 0700.HK）")

# 輸入區
col1, col2 = st.columns([3, 1])
with col1:
    symbols_input = st.text_input(
        "股票代碼（可多個，用逗號分隔）",
        value="AAPL, TSLA, NVDA, 0700.HK",
        placeholder="例如：AAPL, 0700.HK, 9988.HK"
    )
with col2:
    period = st.selectbox("數據期間", ["1mo", "3mo", "6mo", "1y"], index=1)

if st.button("開始分析", type="primary"):
    symbols = [s.strip() for s in symbols_input.split(",") if s.strip()]
    
    if not symbols:
        st.warning("請輸入至少一個股票代碼")
    else:
        for symbol in symbols:
            with st.spinner(f"正在分析 {symbol}..."):
                try:
                    ticker = yf.Ticker(symbol)
                    df = ticker.history(period=period, interval="1d")
                    
                    if df.empty:
                        st.error(f"{symbol} 無法取得數據，請檢查代碼")
                        continue
                    
                    info = ticker.info
                    name = info.get("shortName") or info.get("longName") or symbol
                    price = df["Close"].iloc[-1]
                    prev = df["Close"].iloc[-2]
                    change = price - prev
                    pct = (change / prev) * 100
                    
                    # 均線
                    df["MA20"] = df["Close"].rolling(20).mean()
                    df["MA50"] = df["Close"].rolling(50).mean()
                    ma20 = df["MA20"].iloc[-1]
                    ma50 = df["MA50"].iloc[-1]
                    
                    # RSI
                    delta = df["Close"].diff()
                    gain = delta.where(delta > 0, 0).rolling(14).mean()
                    loss = (-delta.where(delta < 0, 0)).rolling(14).mean()
                    rs = gain / loss.replace(0, np.nan)
                    rsi = 100 - (100 / (1 + rs))
                    rsi_val = rsi.iloc[-1]
                    
                    # 顯示結果
                    st.subheader(f"{name} ({symbol})")
                    
                    c1, c2, c3, c4 = st.columns(4)
                    c1.metric("現價", f"{price:.2f}", f"{change:+.2f} ({pct:+.2f}%)")
                    c2.metric("MA20", f"{ma20:.2f}", "上方" if price > ma20 else "下方")
                    c3.metric("MA50", f"{ma50:.2f}", "上方" if price > ma50 else "下方")
                    
                    if rsi_val > 70:
                        rsi_status = "超買"
                    elif rsi_val < 30:
                        rsi_status = "超賣"
                    else:
                        rsi_status = "中性"
                    c4.metric("RSI(14)", f"{rsi_val:.1f}", rsi_status)
                    
                    # 綜合判斷
                    signals = []
                    if price > ma20 > ma50:
                        signals.append("多頭排列（價 > MA20 > MA50）")
                    elif price < ma20 < ma50:
                        signals.append("空頭排列（價 < MA20 < MA50）")
                    else:
                        signals.append("均線糾纏，方向不明")
                    
                    if rsi_val > 70:
                        signals.append("RSI 偏高，注意回調風險")
                    elif rsi_val < 30:
                        signals.append("RSI 偏低，或有反彈機會")
                    
                    st.write("**綜合判斷：**")
                    for s in signals:
                        st.write(f"- {s}")
                    
                    # 近期高低
                    recent = df.tail(20)
                    st.caption(f"近20日高/低：{recent['High'].max():.2f} / {recent['Low'].min():.2f}")
                    
                    # 簡單走勢圖
                    chart_df = df[["Close", "MA20", "MA50"]].tail(60)
                    st.line_chart(chart_df)
                    
                    st.divider()
                    
                except Exception as e:
                    st.error(f"{symbol} 分析失敗：{e}")

st.markdown("---")
st.caption("數據來源：Yahoo Finance（通常有 15 分鐘延遲）｜僅供參考，非投資建議")
