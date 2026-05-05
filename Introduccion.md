# 🏠 AURORA HOME OPS v1.0  
### 📡 Sistema IoT de Monitoreo Inteligente del Hogar con Alertas en Tiempo Real
Pozos Flores Norberto 22210336

---

## 🎯 Resumen Ejecutivo

**AURORA HOME OPS v1.0** es una solución IoT diseñada para el monitoreo inteligente del hogar mediante sensores simulados o físicos.

El sistema utiliza Flespi como broker MQTT para transmitir datos en tiempo real y permite:

- 📊 Visualización en dashboard web  
- 🚨 Detección de eventos críticos  
- 📱 Notificaciones automáticas al móvil mediante Telegram  
- 🧠 Reglas básicas de automatización  

---

## 📋 Tabla de Contenidos

1. Arquitectura del Sistema  
2. Tecnologías Utilizadas  
3. Estructura del Proyecto  
4. Configuración Inicial  
5. Simulación de Sensores  
6. Sistema de Alertas  
7. Dashboard Web  
8. Instalación y Ejecución  
9. Ejemplos de Datos  
10. Futuras Mejoras  

---

## 🏗️ Arquitectura del Sistema

```mermaid
graph TB
    subgraph "Sensores del Hogar"
        TEMP[🌡️ Temperatura]
        GAS[💨 Gas]
        DOOR[🚪 Puerta]
        WATER[💧 Agua]
    end
    
    subgraph "Simulador Python"
        SIM[simulador_home.py]
    end
    
    subgraph "Broker MQTT"
        MQTT[mqtt.flespi.io]
    end
    
    subgraph "Procesamiento"
        ALERTAS[alertas_home.py]
    end
    
    subgraph "Visualización"
        DASH[Dashboard Flask]
    end
    
    subgraph "Notificaciones"
        TEL[Telegram Bot]
    end
    
    TEMP & GAS & DOOR & WATER --> SIM
    SIM --> MQTT
    MQTT --> ALERTAS
    MQTT --> DASH
    ALERTAS --> TEL


🧰 Tecnologías Utilizadas
Tecnología	Uso
Python	Backend principal
paho-mqtt	Comunicación MQTT
Flask	Dashboard web
Telegram API	Notificaciones móviles
Flespi MQTT	Broker IoT


📂 Estructura del Proyecto
aurora_home/
│
├── simulador_home.py      # Generador de datos IoT
├── alertas_home.py        # Motor de alertas + Telegram
├── app.py                 # Backend Flask
│
└── templates/
    └── index.html         # Dashboard web

⚙️ Configuración Inicial
🔹 Requisitos
Python 3.8+
Conexión a internet
🔹 Instalación de dependencias
pip install paho-mqtt flask requests


🔐 Configuración de MQTT (Flespi)
Crear cuenta en Flespi
Generar token
Reemplazar en los scripts:
FLESPI_TOKEN = "TU_TOKEN_AQUI"


📱 Configuración de Telegram
Abrir Telegram
Buscar @BotFather
Crear bot → obtener TOKEN
Obtener CHAT_ID
FLESPI_TOKEN = "TU_TOKEN_AQUI"


📡 Simulación de Sensores

El sistema simula datos del hogar:
{
  "temperatura": 28,
  "gas": false,
  "puerta": true,
  "agua": false
}


🚨 Sistema de Alertas

Se generan alertas automáticas cuando:

Evento	    Condición     Acción
🔥 Incendio	temperatura > 50°C	Mensaje Telegram
💨 Gas	gas = true	Alerta inmediata
🚪 Intrusión	puerta abierta	Notificación
💧 Fuga	agua = true	Advertencia


🖥️ Dashboard Web
Accede en: http://localhost:5000

Visualiza:
🌡️ Temperatura en tiempo real
💨 Estado de gas
🚪 Estado de puerta
💧 Fugas de agua
📊 Historial de temperatura


▶️ Instalación y Ejecución
1. Ejecutar simulador
python simulador_home.py
2. Ejecutar sistema de alertas
python alertas_home.py
3. Ejecutar dashboard
python app.py


📊 Ejemplo de flujo de datos
Sensor → MQTT → Procesamiento → Dashboard + Telegram
🔥 Resultado

El sistema permite:

✔ Monitoreo en tiempo real
✔ Alertas inmediatas al móvil
✔ Visualización web
✔ Arquitectura escalable


🚀 Futuras Mejoras:
📱 Aplicación móvil nativa
🤖 Integración con IA
🏠 Soporte para dispositivos reales (ESP32)
☁️ Deploy en la nube (AWS / Azure)
🎤 Integración con asistentes (Alexa / Google Home)
