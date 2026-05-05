# 📁 AURORA HOME OPS - PROYECTO COMPLETO

---

## 🏠 simulador_home.py

```python
import paho.mqtt.client as mqtt
import json
import time
import random

FLESPI_TOKEN = "TU_TOKEN_FLESPI_AQUI"

client = mqtt.Client()
client.username_pw_set(FLESPI_TOKEN)
client.connect("mqtt.flespi.io", 1883, 60)
client.loop_start()

while True:
    data = {
        "temperatura": random.randint(20, 60),
        "gas": random.choice([False, False, False, True]),
        "puerta": random.choice([False, True]),
        "agua": random.choice([False, False, True])
    }

    client.publish("aurora/home/sala", json.dumps(data))
    print("📡 Enviado:", data)

    time.sleep(5)
🚨 alertas_home.py
import paho.mqtt.client as mqtt
import json
import requests

FLESPI_TOKEN = "TU_TOKEN_FLESPI_AQUI"

TOKEN = "TU_TOKEN_TELEGRAM"
CHAT_ID = "TU_CHAT_ID"

def enviar_alerta(msg):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    payload = {"chat_id": CHAT_ID, "text": msg}
    requests.post(url, data=payload)

def on_message(client, userdata, msg):
    data = json.loads(msg.payload.decode())

    temp = data.get("temperatura", 0)
    gas = data.get("gas", False)
    puerta = data.get("puerta", False)
    agua = data.get("agua", False)

    print("📥 Recibido:", data)

    if temp > 50:
        enviar_alerta("🔥 ALERTA: Alta temperatura en casa")

    if gas:
        enviar_alerta("💨 ALERTA: Fuga de gas detectada")

    if puerta:
        enviar_alerta("🚨 ALERTA: Puerta abierta")

    if agua:
        enviar_alerta("💧 ALERTA: Fuga de agua")

client = mqtt.Client()
client.username_pw_set(FLESPI_TOKEN)
client.connect("mqtt.flespi.io", 1883, 60)

client.subscribe("aurora/home/sala")
client.on_message = on_message

print("🚨 Sistema de alertas activo...")
client.loop_forever()
🖥️ app.py
from flask import Flask, render_template, jsonify
import paho.mqtt.client as mqtt
import json

app = Flask(__name__)

latest_data = {}

def on_message(client, userdata, msg):
    global latest_data
    latest_data = json.loads(msg.payload.decode())

mqttc = mqtt.Client()
mqttc.username_pw_set("TU_TOKEN_FLESPI_AQUI")
mqttc.connect("mqtt.flespi.io", 1883, 60)
mqttc.subscribe("aurora/home/sala")
mqttc.on_message = on_message
mqttc.loop_start()

@app.route('/')
def index():
    return render_template("index.html")

@app.route('/data')
def data():
    return jsonify(latest_data)

if __name__ == "__main__":
    app.run(debug=True)
🌐 templates/index.html
<!DOCTYPE html>
<html>
<head>
    <title>AURORA HOME OPS</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>

<body style="background:#111; color:white; text-align:center; font-family:sans-serif;">

<h1>🏠 AURORA HOME OPS</h1>

<h2 id="temp">🌡️ Temperatura: --</h2>
<h2 id="gas">💨 Gas: --</h2>
<h2 id="puerta">🚪 Puerta: --</h2>
<h2 id="agua">💧 Agua: --</h2>

<canvas id="chart" width="400" height="150"></canvas>

<script>
const ctx = document.getElementById('chart').getContext('2d');

const chart = new Chart(ctx, {
    type: 'line',
    data: {
        labels: [],
        datasets: [{
            label: 'Temperatura',
            data: [],
            borderColor: 'orange'
        }]
    }
});

async function update(){
    const res = await fetch('/data');
    const data = await res.json();

    document.getElementById("temp").innerText = "🌡️ Temperatura: " + data.temperatura;
    document.getElementById("gas").innerText = "💨 Gas: " + (data.gas ? "ALERTA" : "OK");
    document.getElementById("puerta").innerText = "🚪 Puerta: " + (data.puerta ? "ABIERTA" : "CERRADA");
    document.getElementById("agua").innerText = "💧 Agua: " + (data.agua ? "FUGA" : "OK");

    let time = new Date().toLocaleTimeString();

    chart.data.labels.push(time);
    chart.data.datasets[0].data.push(data.temperatura);

    if(chart.data.labels.length > 20){
        chart.data.labels.shift();
        chart.data.datasets[0].data.shift();
    }

    chart.update();
}

setInterval(update, 2000);
</script>

</body>
</html>
