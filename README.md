pip install requests qrcode

import requests
import qrcode
from flask import Flask, request, jsonify

app = Flask(__name__)

# Payment API is yet to be patnered and used the best one
API_KEY = 'your_api_key'
API_SECRET = 'your_api_secret'
PAYMENT_GATEWAY_URL = 'https://api.paymentgateway.com/create_order'
VERIFY_URL = 'https://api.paymentgateway.com/verify_payment'

def create_order(amount, currency='INR'):
    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    data = {
        'amount': amount,
        'currency': currency,
        'payment_capture': 1  # 1 for automatic capture, 0 for manual
    }
    response = requests.post(PAYMENT_GATEWAY_URL, json=data, headers=headers)
    return response.json()

def generate_qr_code(payment_url):
    qr = qrcode.make(payment_url)
    qr.save('payment_qr.png')

@app.route('/create_payment', methods=['POST'])
def create_payment():
    data = request.json
    amount = data['amount']
    order = create_order(amount)
    payment_url = order['payment_url']
    generate_qr_code(payment_url)
    return jsonify(order)

@app.route('/verify_payment', methods=['POST'])
def verify_payment():
    data = request.json
    payment_id = data['payment_id']
    order_id = data['order_id']

    headers = {
        'Authorization': f'Bearer {API_KEY}',
        'Content-Type': 'application/json'
    }
    response = requests.get(f'{VERIFY_URL}/{order_id}/{payment_id}', headers=headers)
    return jsonify(response.json())

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

