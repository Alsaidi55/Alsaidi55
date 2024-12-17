# الاستيراد
import openai
import dash
from dash import dcc, html
import plotly.graph_objs as go
import pandas as pd
from web3 import Web3
from tensorflow.keras.layers import Input, Dense
from tensorflow.keras.models import Model
import speech_recognition as sr
import websocket
import json
import requests
import numpy as np
import time
import streamlit as st

# إعداد API الخاصة بـ GPT-4
openai.api_key = "AIzaSyAY51NlGy4O21cR4WeJKgiIgipPdPgA-FA"  # دمج المفتاح هنا

# 1. تحليل المشاعر باستخدام GPT-4
def analyze_sentiment_with_gpt(text):
    response = openai.Completion.create(
        model="gpt-4",  # استخدام GPT-4 لتحليل النص
        prompt=f"Analyze the sentiment of the following news article and classify it as Positive, Negative, or Neutral:\n\n{text}",
        max_tokens=50
    )
    sentiment = response.choices[0].text.strip()
    return sentiment

# 2. الرسوم البيانية التفاعلية باستخدام Plotly Dash مع تأثيرات متقدمة
def create_dash_app():
    df = pd.read_csv('forex_data.csv')

    # إنشاء التطبيق Dash
    app = dash.Dash(__name__)

    # واجهة المستخدم مع تأثيرات Parallax للتمرير
    app.layout = html.Div([
        html.H1("منصة تحليل الفوركس 2024", style={'text-align': 'center', 'color': '#1e3d58', 'font-size': '40px'}),
        
        # تأثير Parallax مع خلفية متحركة
        html.Div(id='parallax', style={
            'background-image': 'url("https://example.com/parallax-image.jpg")',  # صورة خلفية متحركة
            'height': '500px',
            'background-attachment': 'fixed',
            'background-size': 'cover',
            'position': 'relative'
        }),

        # رسم بياني للأسعار التفاعلي مع تأثيرات تغيير اللون والتكبير
        dcc.Graph(
            id='forex-graph',
            figure={
                'data': [
                    go.Candlestick(
                        x=pd.to_datetime(df['timestamp']),
                        open=df['open'],
                        high=df['high'],
                        low=df['low'],
                        close=df['close'],
                        name="EURUSD",
                        increasing_line_color='green',
                        decreasing_line_color='red'
                    ),
                ],
                'layout': go.Layout(
                    title="بيانات الفوركس 2024",
                    xaxis={'rangeslider': {'visible': False}},
                    yaxis={'title': 'سعر العملة'},
                    dragmode='zoom',  # إمكانية تكبير الرسم البياني
                    showlegend=True,
                    font=dict(family="Arial, sans-serif", size=14, color="darkgray"),
                    plot_bgcolor="rgb(235, 240, 245)",
                    hovermode='closest',
                ),
            }
        ),
    ])

    # تشغيل الخادم
    app.run_server(debug=True)

# 3. تحليل البيانات الحية باستخدام Streamlit
def streamlit_dashboard():
    st.title("منصة تحليل الفوركس 2024")

    # تأثيرات على الخلفية عند التمرير (Parallax)
    st.markdown(
        """
        <style>
            .stApp {
                background-image: url('https://example.com/parallax-image.jpg');
                background-attachment: fixed;
                background-size: cover;
            }
        </style>
        """, unsafe_allow_html=True)

    # عرض الرسومات البيانية
    st.subheader("الرسم البياني للأسعار")
    df = pd.read_csv('forex_data.csv')
    st.line_chart(df['close'])

    # إدخال صوتي للمستخدم
    st.subheader("تحدث لتقديم أمر تداول")
    command = listen_for_command()
    st.write(f"أنت قلت: {command}")

    # عرض التوصية بناءً على البيانات
    recommendation = analyze_live_data(fetch_flex_data())
    st.write(f"التوصية: {recommendation}")

# 4. الإدخال الصوتي باستخدام SpeechRecognition
def listen_for_command():
    recognizer = sr.Recognizer()
    mic = sr.Microphone()

    with mic as source:
        print("الرجاء التحدث...")
        audio = recognizer.listen(source)

    try:
        command = recognizer.recognize_google(audio, language="ar-SA")
        print(f"أنت قلت: {command}")
        return command
    except sr.UnknownValueError:
        print("لم أتمكن من التعرف على الصوت")
    except sr.RequestError:
        print("حدث خطأ أثناء الاتصال بالخدمة")

# 5. توصيات حية باستخدام WebSockets
def on_message(ws, message):
    data = json.loads(message)
    print("توصية حية:", data['recommendation'])

def on_error(ws, error):
    print("خطأ:", error)

def on_close(ws, close_status_code, close_msg):
    print("تم الإغلاق")

def on_open(ws):
    print("تم الاتصال!")

def run_websocket():
    ws = websocket.WebSocketApp("wss://your_websocket_server.com",
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever()

# 6. تحليل البيانات الحية ودمجها مع الذكاء الاصطناعي
def analyze_live_data(data):
    # استخدام الذكاء الاصطناعي لتحليل البيانات
    if data['USD'] > 1.2:
        return "شراء"
    else:
        return "بيع"

# 7. استخدام Web3 لتخزين المعاملات في Blockchain
def store_transaction_in_blockchain(transaction_data):
    w3 = Web3(Web3.HTTPProvider('https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID'))

    if w3.isConnected():
        print("متصل بشبكة Ethereum!")

    # إعداد العقد الذكي
    contract_address = '0xYourContractAddress'
    abi = [...]  # ABI الخاص بالعقد الذكي
    contract = w3.eth.contract(address=contract_address, abi=abi)

    # إرسال المعاملة
    tx = contract.functions.storeData(transaction_data).transact({
        'from': w3.eth.accounts[0],
        'gas': 2000000,
        'gasPrice': w3.toWei('20', 'gwei'),
    })
    return tx

# تشغيل التطبيقات
if __name__ == '__main__':
    # تشغيل تطبيق Dash
    create_dash_app()

    # تشغيل توصيات حية
    run_websocket()

    # تشغيل واجهة Streamlit
    streamlit_dashboard()
