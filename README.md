import os
import requests
import yfinance as yf
from datetime import datetime

# 깃허브 비밀 금고(Secrets)에서 꺼내 쓰기
BOT_TOKEN = os.environ['MY_TOKEN']
CHAT_ID = os.environ['MY_ID']

def send_telegram(msg):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    data = {'chat_id': CHAT_ID, 'text': msg, 'parse_mode': 'Markdown'}
    requests.post(url, data=data)

# 데이터 조회 (BTC, IONQ)
btc = yf.download('BTC-USD', period='2mo', progress=False)
ionq = yf.download('IONQ', period='1wk', progress=False)

# 지표 계산
btc['MA20'] = btc['Close'].rolling(window=20).mean()
btc_close = float(btc['Close'].iloc[-1])
btc_ma = float(btc['MA20'].iloc[-1])
prev_btc = float(btc['Close'].iloc[-2])
prev_ma = float(btc['MA20'].iloc[-2])
ionq_price = float(ionq['Close'].iloc[-1])

# 신호 판단
msg = ""
status = ""

if prev_btc <= prev_ma and btc_close > btc_ma:
    status = "🚀 *[강력 매수 신호]*"
    msg = "비트코인 상승장 진입! IONQ 매수 추천."
elif prev_btc >= prev_ma and btc_close < btc_ma:
    status = "📉 *[전량 매도 신호]*"
    msg = "비트코인 하락장 진입! 현금화 추천."
else:
    # (선택) 매일매일 알림 받기 싫으면 이 부분 주석 처리
    if btc_close > btc_ma:
        status = "🔥 *[상승장 지속 중]*"
        msg = "추세 양호. 계속 보유(Hold)하세요."
    else:
        status = "❄️ *[하락장 지속 중]*"
        msg = "아직 하락세입니다. 관망하세요."

final_msg = f"{status}\n\nBTC: ${btc_close:,.0f} (20일선 ${btc_ma:,.0f})\nIONQ: ${ionq_price:.2f}\n\n{msg}"
send_telegram(final_msg)
