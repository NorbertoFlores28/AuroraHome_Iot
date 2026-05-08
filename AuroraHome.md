# 📁 AURORA HOME OPS — Código Fuente Completo

---

## 🏠 `simulador_home.py`

```python
import paho.mqtt.client as mqtt
import json
import time
import random

FLESPI_TOKEN = "EjMNmJ3R0RYmjOFn6uiw6vdNa5UqiNJK7d3elpFrSlgsJJtnR2x61EBDpy8TG3j0"

client = mqtt.Client()
client.username_pw_set(FLESPI_TOKEN)
client.connect("mqtt.flespi.io", 1883, 60)
client.loop_start()

while True:
    data = {
        "temperatura": random.randint(20, 60),
        "gas":    random.choice([False, False, False, True]),
        "puerta": random.choice([False, True]),
        "agua":   random.choice([False, False, True])
    }

    client.publish("aurora/home/sala", json.dumps(data))
    print("📡 Enviado:", data)

    time.sleep(5)
```

---

## 🚨 `alertas_home.py`

```python
import paho.mqtt.client as mqtt
import json
import requests
import time

FLESPI_TOKEN = "EjMNmJ3R0RYmjOFn6uiw6vdNa5UqiNJK7d3elpFrSlgsJJtnR2x61EBDpy8TG3j0"

TOKEN   = "8446002292:AAFb7lwXjF1UuOYCqNIJTEGEpdr7E4SNAjc"
CHAT_ID = "8385382206"

# Tiempo entre alertas (2 minutos)
TIEMPO_ALERTA = 120

# Guardar última alerta enviada
ultima_alerta = {
    "temperatura": 0,
    "gas": 0,
    "puerta": 0,
    "agua": 0
}

def enviar_alerta(msg):
    url     = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    payload = {"chat_id": CHAT_ID, "text": msg}
    requests.post(url, data=payload)

def puede_enviar(tipo):
    ahora = time.time()

    if ahora - ultima_alerta[tipo] >= TIEMPO_ALERTA:
        ultima_alerta[tipo] = ahora
        return True

    return False

def on_message(client, userdata, msg):
    data = json.loads(msg.payload.decode())

    temp   = data.get("temperatura", 0)
    gas    = data.get("gas",    False)
    puerta = data.get("puerta", False)
    agua   = data.get("agua",   False)

    print("📥 Recibido:", data)

    if temp > 50:
        if puede_enviar("temperatura"):
            enviar_alerta("🔥 ALERTA: Alta temperatura en casa")

    if gas:
        if puede_enviar("gas"):
            enviar_alerta("💨 ALERTA: Fuga de gas detectada")

    if puerta:
        if puede_enviar("puerta"):
            enviar_alerta("🚨 ALERTA: Puerta abierta")

    if agua:
        if puede_enviar("agua"):
            enviar_alerta("💧 ALERTA: Fuga de agua")

client = mqtt.Client()
client.username_pw_set(FLESPI_TOKEN)
client.connect("mqtt.flespi.io", 1883, 60)

client.subscribe("aurora/home/sala")
client.on_message = on_message

print("🚨 Sistema de alertas activo...")
client.loop_forever()
```

---

## 🖥️ `app.py`

```python
from flask import Flask, render_template, jsonify
import paho.mqtt.client as mqtt
import json

app = Flask(__name__)

latest_data = {}

def on_message(client, userdata, msg):
    global latest_data
    latest_data = json.loads(msg.payload.decode())

mqttc = mqtt.Client()
mqttc.username_pw_set("EjMNmJ3R0RYmjOFn6uiw6vdNa5UqiNJK7d3elpFrSlgsJJtnR2x61EBDpy8TG3j0")
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
  app.run(host="0.0.0.0", port=5000, debug=False)
```

---

## 🌐 `templates/index.html`

```
<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>AURORA HOME OPS</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>

*{
    margin:0;
    padding:0;
    box-sizing:border-box;
}

body{
    font-family:Arial, Helvetica, sans-serif;
    background:
        radial-gradient(circle at top,#1e293b,#0f172a,#020617);
    color:white;
    min-height:100vh;
    overflow-x:hidden;
}

.container{
    width:95%;
    margin:auto;
    padding-bottom:40px;
}

/* HEADER */

.header{
    text-align:center;
    padding:30px 0 10px;
}

.header h1{
    font-size:52px;

    text-shadow:
        0 0 10px rgba(255,255,255,0.3),
        0 0 30px rgba(255,255,255,0.15);
}

.time{
    color:#94a3b8;
    margin-top:10px;
    font-size:18px;
}

/* STATUS */

.status-bar{
    display:flex;
    justify-content:center;
    gap:20px;
    margin:25px 0;
    flex-wrap:wrap;
}

.status{
    background:rgba(255,255,255,0.05);
    border:1px solid rgba(255,255,255,0.08);
    padding:12px 22px;
    border-radius:15px;
    backdrop-filter:blur(10px);

    box-shadow:
        0 0 15px rgba(0,0,0,0.3);
}

.online{
    color:#00ff88;
    font-weight:bold;
}

/* INFO */

.info-grid{

    display:grid;
    grid-template-columns:repeat(4,1fr);
    gap:20px;
    margin-bottom:25px;
}

.info-card{

    background:rgba(255,255,255,0.05);

    border:1px solid rgba(255,255,255,0.08);

    border-radius:20px;

    padding:20px;

    text-align:center;

    backdrop-filter:blur(10px);
}

.info-title{
    color:#94a3b8;
    margin-bottom:10px;
}

.info-value{
    font-size:30px;
    font-weight:bold;
}

/* CARDS */

.cards{
    display:grid;
    grid-template-columns:repeat(4,1fr);
    gap:25px;
}

.card{

    background:rgba(255,255,255,0.05);

    border:1px solid rgba(255,255,255,0.08);

    border-radius:25px;

    padding:25px;

    text-align:center;

    backdrop-filter:blur(15px);

    transition:0.3s;

    box-shadow:
        0 0 25px rgba(0,0,0,0.35);
}

.card:hover{

    transform:translateY(-8px);

    box-shadow:
        0 0 35px rgba(255,255,255,0.1);
}

.title{
    font-size:22px;
    margin-bottom:20px;
    color:#e2e8f0;
}

/* GAUGE */

.gauge-container{
    position:relative;
    width:220px;
    height:220px;
    margin:auto;
}

.gauge{
    width:220px;
    height:220px;
}

.gauge-text{

    position:absolute;

    top:50%;
    left:50%;

    transform:translate(-50%,-50%);

    font-size:34px;
    font-weight:bold;
}

/* CHART */

.chart-box{

    margin-top:35px;

    background:rgba(255,255,255,0.05);

    border:1px solid rgba(255,255,255,0.08);

    border-radius:25px;

    padding:25px;

    height:520px;

    backdrop-filter:blur(15px);

    box-shadow:
        0 0 25px rgba(0,0,0,0.35);
}

canvas{
    width:100% !important;
    height:100% !important;
}

/* RESPONSIVE */

@media(max-width:1200px){

    .cards,
    .info-grid{
        grid-template-columns:1fr 1fr;
    }
}

@media(max-width:700px){

    .cards,
    .info-grid{
        grid-template-columns:1fr;
    }

    .header h1{
        font-size:38px;
    }
}

</style>
</head>

<body>

<div class="container">

    <!-- HEADER -->

    <div class="header">

        <h1>🏠 AURORA HOME OPS</h1>

        <div class="time" id="clock">
            --:--:--
        </div>

    </div>

    <!-- STATUS -->

    <div class="status-bar">

        <div class="status">
            🟢 MQTT <span class="online">ONLINE</span>
        </div>

        <div class="status">
            🟢 FLESPI <span class="online">CONNECTED</span>
        </div>

        <div class="status">
            🟢 TELEGRAM <span class="online">ACTIVE</span>
        </div>

    </div>

    <!-- EXTRA INFO -->

    <div class="info-grid">

        <div class="info-card">
            <div class="info-title">🌡️ Temp Máxima</div>
            <div class="info-value" id="maxTemp">0°C</div>
        </div>

        <div class="info-card">
            <div class="info-title">❄️ Temp Mínima</div>
            <div class="info-value" id="minTemp">0°C</div>
        </div>

        <div class="info-card">
            <div class="info-title">⏱️ Uptime</div>
            <div class="info-value" id="uptime">0s</div>
        </div>

        <div class="info-card">
            <div class="info-title">📡 Última actualización</div>
            <div class="info-value" id="lastUpdate">0s</div>
        </div>

    </div>

    <!-- GAUGES -->

    <div class="cards">

        <!-- TEMPERATURA -->

        <div class="card">

            <div class="title">
                🌡️ Temperatura
            </div>

            <div class="gauge-container">

                <svg class="gauge" viewBox="0 0 200 200">

                    <circle
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#222"
                        stroke-width="15"
                        fill="none"
                    />

                    <circle
                        id="temp-gauge"
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#00ff88"
                        stroke-width="15"
                        fill="none"
                        stroke-linecap="round"
                        stroke-dasharray="440"
                        stroke-dashoffset="440"
                        transform="rotate(-90 100 100)"
                    />

                </svg>

                <div class="gauge-text" id="temp">
                    --°
                </div>

            </div>

        </div>

        <!-- GAS -->

        <div class="card">

            <div class="title">
                💨 Gas
            </div>

            <div class="gauge-container">

                <svg class="gauge" viewBox="0 0 200 200">

                    <circle
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#222"
                        stroke-width="15"
                        fill="none"
                    />

                    <circle
                        id="gas-gauge"
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#00ff88"
                        stroke-width="15"
                        fill="none"
                        stroke-linecap="round"
                        stroke-dasharray="440"
                        stroke-dashoffset="220"
                        transform="rotate(-90 100 100)"
                    />

                </svg>

                <div class="gauge-text" id="gas">
                    OK
                </div>

            </div>

        </div>

        <!-- PUERTA -->

        <div class="card">

            <div class="title">
                🚪 Puerta
            </div>

            <div class="gauge-container">

                <svg class="gauge" viewBox="0 0 200 200">

                    <circle
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#222"
                        stroke-width="15"
                        fill="none"
                    />

                    <circle
                        id="door-gauge"
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#00ff88"
                        stroke-width="15"
                        fill="none"
                        stroke-linecap="round"
                        stroke-dasharray="440"
                        stroke-dashoffset="220"
                        transform="rotate(-90 100 100)"
                    />

                </svg>

                <div class="gauge-text" id="puerta">
                    SAFE
                </div>

            </div>

        </div>

        <!-- AGUA -->

        <div class="card">

            <div class="title">
                💧 Agua
            </div>

            <div class="gauge-container">

                <svg class="gauge" viewBox="0 0 200 200">

                    <circle
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#222"
                        stroke-width="15"
                        fill="none"
                    />

                    <circle
                        id="water-gauge"
                        cx="100"
                        cy="100"
                        r="70"
                        stroke="#00ff88"
                        stroke-width="15"
                        fill="none"
                        stroke-linecap="round"
                        stroke-dasharray="440"
                        stroke-dashoffset="220"
                        transform="rotate(-90 100 100)"
                    />

                </svg>

                <div class="gauge-text" id="agua">
                    OK
                </div>

            </div>

        </div>

    </div>

    <!-- CHART -->

    <div class="chart-box">
        <canvas id="chart"></canvas>
    </div>

</div>

<script>

/* VARIABLES */

let startTime = Date.now();
let lastMessage = Date.now();

let maxTemp = -999;
let minTemp = 999;

let gasAlerts = 0;
let doorAlerts = 0;
let waterAlerts = 0;

/* CLOCK */

function updateClock(){

    const now = new Date();

    document.getElementById("clock").innerHTML =
        now.toLocaleDateString() +
        " — " +
        now.toLocaleTimeString();
}

setInterval(updateClock,1000);

updateClock();

/* CHART */

const ctx =
document.getElementById('chart').getContext('2d');

const chart = new Chart(ctx, {

    type:'line',

    data:{
        labels:[],

        datasets:[

            {
                label:'Temperatura',
                data:[],
                borderColor:'#ffaa00',
                borderWidth:4,
                tension:0.4
            },

            {
                label:'Gas',
                data:[],
                borderColor:'#ff3b3b',
                borderWidth:2
            },

            {
                label:'Puerta',
                data:[],
                borderColor:'#00bfff',
                borderWidth:2
            },

            {
                label:'Agua',
                data:[],
                borderColor:'#00ff88',
                borderWidth:2
            }

        ]
    },

    options:{

        responsive:true,

        maintainAspectRatio:false,

        plugins:{
            legend:{
                labels:{
                    color:'white'
                }
            }
        },

        scales:{

            x:{
                ticks:{
                    color:'white'
                },

                grid:{
                    color:'#333'
                }
            },

            y:{
                ticks:{
                    color:'white'
                },

                grid:{
                    color:'#333'
                }
            }
        }
    }
});

/* UPDATE */

async function update(){

    const res =
    await fetch('/data');

    const data =
    await res.json();

    lastMessage = Date.now();

    /* TEMPERATURA */

    const temp =
    data.temperatura || 0;

    document.getElementById("temp").innerHTML =
    temp + "°";

    const gauge =
    document.getElementById("temp-gauge");

    const offset =
    440 - (440 * temp / 100);

    gauge.style.strokeDashoffset =
    offset;

    /* COLOR DINÁMICO */

    let color = "#00ff88";

    if(temp >= 40 && temp < 70){

        color = "#ffaa00";
    }

    if(temp >= 70){

        color = "#ff3b3b";
    }

    gauge.style.stroke = color;

    document.getElementById("temp").style.color =
    color;

    /* MAX Y MIN */

    if(temp > maxTemp){
        maxTemp = temp;
    }

    if(temp < minTemp){
        minTemp = temp;
    }

    document.getElementById("maxTemp").innerHTML =
    maxTemp + "°C";

    document.getElementById("minTemp").innerHTML =
    minTemp + "°C";

    /* GAS */

    const gasGauge =
    document.getElementById("gas-gauge");

    if(data.gas){

        gasAlerts++;

        document.getElementById("gas").innerHTML =
        "ALERT";

        document.getElementById("gas").style.color =
        "#ff3b3b";

        gasGauge.style.stroke =
        "#ff3b3b";

        gasGauge.style.strokeDashoffset =
        "0";

    }else{

        document.getElementById("gas").innerHTML =
        "OK";

        document.getElementById("gas").style.color =
        "#00ff88";

        gasGauge.style.stroke =
        "#00ff88";

        gasGauge.style.strokeDashoffset =
        "220";
    }

    /* PUERTA */

    const doorGauge =
    document.getElementById("door-gauge");

    if(data.puerta){

        doorAlerts++;

        document.getElementById("puerta").innerHTML =
        "OPEN";

        document.getElementById("puerta").style.color =
        "#ffaa00";

        doorGauge.style.stroke =
        "#ffaa00";

        doorGauge.style.strokeDashoffset =
        "0";

    }else{

        document.getElementById("puerta").innerHTML =
        "SAFE";

        document.getElementById("puerta").style.color =
        "#00ff88";

        doorGauge.style.stroke =
        "#00ff88";

        doorGauge.style.strokeDashoffset =
        "220";
    }

    /* AGUA */

    const waterGauge =
    document.getElementById("water-gauge");

    if(data.agua){

        waterAlerts++;

        document.getElementById("agua").innerHTML =
        "LEAK";

        document.getElementById("agua").style.color =
        "#00bfff";

        waterGauge.style.stroke =
        "#00bfff";

        waterGauge.style.strokeDashoffset =
        "0";

    }else{

        document.getElementById("agua").innerHTML =
        "OK";

        document.getElementById("agua").style.color =
        "#00ff88";

        waterGauge.style.stroke =
        "#00ff88";

        waterGauge.style.strokeDashoffset =
        "220";
    }

    /* CHART */

    const time =
    new Date().toLocaleTimeString();

    chart.data.labels.push(time);

    chart.data.datasets[0].data.push(temp);
    chart.data.datasets[1].data.push(data.gas ? 100 : 0);
    chart.data.datasets[2].data.push(data.puerta ? 100 : 0);
    chart.data.datasets[3].data.push(data.agua ? 100 : 0);

    if(chart.data.labels.length > 20){

        chart.data.labels.shift();

        chart.data.datasets.forEach(ds => {
            ds.data.shift();
        });
    }

    chart.update();
}

setInterval(update,2000);

/* UPTIME */

setInterval(()=>{

    const seconds =
    Math.floor((Date.now() - startTime)/1000);

    document.getElementById("uptime").innerHTML =
    seconds + "s";

    const last =
    Math.floor((Date.now() - lastMessage)/1000);

    document.getElementById("lastUpdate").innerHTML =
    "Hace " + last + "s";

},1000);

</script>

</body>
</html>

```
