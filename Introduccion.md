# 🏠 AURORA HOME OPS v1.0  
### 📡 Sistema IoT de Monitoreo Inteligente del Hogar con Alertas en Tiempo Real

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
