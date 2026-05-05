# 🏠 AURORA HOME OPS v1.0
### 📡 Sistema IoT de Monitoreo Inteligente del Hogar con Alertas en Tiempo Real

**Pozos Flores Norberto** · Matrícula: 22210336

---

## 🎯 Resumen Ejecutivo

**AURORA HOME OPS v1.0** es una solución IoT diseñada para el monitoreo inteligente del hogar mediante sensores simulados o físicos. El sistema utiliza Flespi como broker MQTT para transmitir datos en tiempo real y permite:

- 📊 Visualización en dashboard web
- 🚨 Detección de eventos críticos
- 📱 Notificaciones automáticas al móvil mediante Telegram
- 🧠 Reglas básicas de automatización

---

## 📋 Tabla de Contenidos

1. [Arquitectura del Sistema](#-arquitectura-del-sistema)
2. [Tecnologías Utilizadas](#-tecnologías-utilizadas)
3. [Estructura del Proyecto](#-estructura-del-proyecto)
4. [Configuración Inicial](#️-configuración-inicial)
5. [Simulación de Sensores](#-simulación-de-sensores)
6. [Sistema de Alertas](#-sistema-de-alertas)
7. [Dashboard Web](#️-dashboard-web)
8. [Instalación y Ejecución](#️-instalación-y-ejecución)
9. [Ejemplos de Datos](#-flujo-de-datos)
10. [Futuras Mejoras](#-futuras-mejoras)

---

## 🏗️ Arquitectura del Sistema

```
Sensores del Hogar
┌──────────────────────────────────────────────┐
│  🌡️ Temperatura  💨 Gas  🚪 Puerta  💧 Agua  │
└────────────────────┬─────────────────────────┘
                     │
              simulador_home.py
                     │
              mqtt.flespi.io  (Broker MQTT)
                    / \
                   /   \
       alertas_home.py   Dashboard Flask
              │                │
         Telegram Bot    http://localhost:5000
```

---

## 🧰 Tecnologías Utilizadas

| Tecnología    | Uso                      |
|---------------|--------------------------|
| Python        | Backend principal        |
| paho-mqtt     | Comunicación MQTT        |
| Flask         | Dashboard web            |
| Telegram API  | Notificaciones móviles   |
| Flespi MQTT   | Broker IoT               |

---

## 📂 Estructura del Proyecto

```
aurora_home/
│
├── simulador_home.py      # Generador de datos IoT
├── alertas_home.py        # Motor de alertas + Telegram
├── app.py                 # Backend Flask
│
└── templates/
    └── index.html         # Dashboard web
```

---

## ⚙️ Configuración Inicial

### 🔹 Requisitos

- Python 3.8+
- Conexión a internet

### 🔹 Instalación de dependencias

```bash
pip install paho-mqtt flask requests
```

### 🔐 Configuración de MQTT (Flespi)

1. Crear cuenta en [Flespi](https://flespi.io)
2. Generar token
3. Reemplazar en los scripts:

```python
FLESPI_TOKEN = "TU_TOKEN_AQUI"
```

### 📱 Configuración de Telegram

1. Abrir Telegram
2. Buscar `@BotFather`
3. Crear bot → obtener TOKEN
4. Obtener CHAT_ID

```python
TELEGRAM_TOKEN = "TU_TOKEN_BOT"
CHAT_ID = "TU_CHAT_ID"
```

---

## 📡 Simulación de Sensores

El sistema simula datos del hogar en formato JSON:

```json
{
  "temperatura": 28,
  "gas": false,
  "puerta": true,
  "agua": false
}
```

---

## 🚨 Sistema de Alertas

Se generan alertas automáticas cuando se detectan las siguientes condiciones:

| Evento        | Condición           | Acción            |
|---------------|---------------------|-------------------|
| 🔥 Incendio   | temperatura > 50°C  | Mensaje Telegram  |
| 💨 Gas        | gas = true          | Alerta inmediata  |
| 🚪 Intrusión  | puerta abierta      | Notificación      |
| 💧 Fuga       | agua = true         | Advertencia       |

---

## 🖥️ Dashboard Web

Accede en: `http://localhost:5000`

Visualiza en tiempo real:

- 🌡️ Temperatura
- 💨 Estado de gas
- 🚪 Estado de puerta
- 💧 Fugas de agua
- 📊 Historial de temperatura

---

## ▶️ Instalación y Ejecución

### 1. Ejecutar simulador

```bash
python simulador_home.py
```

### 2. Ejecutar sistema de alertas

```bash
python alertas_home.py
```

### 3. Ejecutar dashboard

```bash
python app.py
```

---

## 📊 Flujo de Datos

```
Sensor → MQTT → Procesamiento → Dashboard + Telegram
```

**El sistema permite:**

- ✔ Monitoreo en tiempo real
- ✔ Alertas inmediatas al móvil
- ✔ Visualización web
- ✔ Arquitectura escalable

---

## 🚀 Futuras Mejoras

- 📱 Aplicación móvil nativa
- 🤖 Integración con IA para predicción de eventos
- 🏠 Soporte para dispositivos reales (ESP32, Arduino)
- ☁️ Deploy en la nube (AWS / Azure)
- 🎤 Integración con asistentes de voz (Alexa, Google Home)lexa / Google Home)
