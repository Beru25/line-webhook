# line-webhook
from flask import Flask, request
import requests
import json

app = Flask(__name__)

LINE_TOKEN = 'YOUR_LINE_CHANNEL_ACCESS_TOKEN'
NODEMCU_URL = 'http://YOUR_NODEMCU_IP/control'

def reply_message(reply_token, text):
    headers = {
        "Authorization": f"Bearer {LINE_TOKEN}",
        "Content-Type": "application/json"
    }
    body = {
        "replyToken": reply_token,
        "messages": [{"type": "text", "text": text}]
    }
    requests.post("https://api.line.me/v2/bot/message/reply", headers=headers, json=body)

@app.route("/webhook", methods=['POST'])
def webhook():
    payload = request.json
    event = payload["events"][0]
    user_text = event["message"]["text"]
    reply_token = event["replyToken"]

    if user_text == "รดน้ำ":
        try:
            requests.get(NODEMCU_URL + "?action=water")
            reply_message(reply_token, "💧 รดน้ำต้นไม้เรียบร้อยแล้วครับ")
        except:
            reply_message(reply_token, "❌ ไม่สามารถเชื่อมต่ออุปกรณ์ได้")
    elif user_text == "สถานะ":
        try:
            res = requests.get(NODEMCU_URL + "?action=status")
            reply_message(reply_token, f"📊 ความชื้นในดิน: {res.text}")
        except:
            reply_message(reply_token, "❌ ไม่สามารถดึงสถานะได้")
    else:
        reply_message(reply_token, "❓ คำสั่งไม่ถูกต้อง\nพิมพ์: \"รดน้ำ\" หรือ \"สถานะ\"")

    return "OK"
