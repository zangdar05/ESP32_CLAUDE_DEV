# 🏗️ ARCHITECTURE - Vue C4 du système ESP32_CLAUDE_DEV

> Architecture complète du système IoT: ESP32 + Server + Frontend

---

## 📋 Table des matières
1. [Level 1: System Context](#level-1-system-context)
2. [Level 2: Containers](#level-2-containers)
3. [Level 3: Components](#level-3-components)
4. [Level 4: Code (Data models)](#level-4-code)
5. [Data Flow](#data-flow)
6. [Technology Stack](#technology-stack)

---

## LEVEL 1: System Context

```
┌────────────────────────────────────────────────────────────────────┐
│                      ESP32_CLAUDE_DEV System                       │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────────┐                  ┌──────────────────┐      │
│  │                  │                  │                  │      │
│  │  ESP32 Device    │◄──MQTT/HTTP──►   │  Server Backend  │      │
│  │  (Greenhouse)    │    +SSL/TLS      │  (NAS Synology)  │      │
│  │                  │                  │                  │      │
│  │  - DHT22         │                  │  - REST API      │      │
│  │  - pH sensor     │                  │  - WebSocket     │      │
│  │  - Power meter   │                  │  - PostgreSQL    │      │
│  └──────────────────┘                  └──────────────────┘      │
│         │                                       │                 │
│         │                                       │                 │
│         │                                  ┌────▼─────────┐      │
│         │                                  │              │      │
│         │                                  │  Frontend    │      │
│         └─────────────── WebSocket ────────│ (React App)  │      │
│                    (Real-time)             │  Smartphone  │      │
│                                            │              │      │
│                                            └──────────────┘      │
│                                                                   │
└────────────────────────────────────────────────────────────────────┘

User: Farmer (local Greenhouse)
Goal: Monitor environment, get alerts, optimize conditions
```

---

## LEVEL 2: Containers

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ESP32 Firmware Container                                               │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌──────────���──────────┐  ┌──────────────┐  ┌─────────────────┐        │
│  │  SensorManager      │  │ WiFi/MQTT    │  │  StorageManager │        │
│  │  (Read sensors)     │  │ (Connectivity)   │  (Local cache)  │        │
│  │                     │  │              │  │                 │        │
│  │ - DHT22             │  │ - WiFi       │  │ - SPIFFS        │        │
│  │ - Atlas pH          │  │ - MQTT pub   │  │ - Rotation      │        │
│  │ - Power meter       │  │ - HTTP POST  │  │ - Sync logic    │        │
│  │ - Calibration       │  │ - TLS/Certs  │  │ - OTA updates   │        │
│  └─────────────────────┘  └──────────────┘  └─────────────────┘        │
│           │                       │                   │                 │
│           └───────────────────────┼───────────────────┘                 │
│                           main.cpp                                       │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  Server Backend Container (NAS Synology)                                │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────┐          │
│  │  API Server    │  │ WebSocket      │  │  Services        │          │
│  │  (Express)     │  │  (socket.io)   │  │  (Business Logic)│          │
│  │                │  │                │  │                  │          │
│  │ - POST /meas   │  │ - measurements │  │ - Recording      │          │
│  │ - GET /devices │  │ - alerts       │  │ - Statistics     │          │
���  │ - Auth         │  │ - device_sync  │  │ - Alerts         │          │
│  │ - Rate limit   │  │                │  │ - Archive        │          │
│  └────────────────┘  └────────────────┘  └──────────────────┘          │
│           │                   │                    │                    │
│           └───────────────────┼────────────────────┘                    │
│                       │                                                 │
│                   Database Layer                                        │
│                   ┌──────────────┐                                      │
│                   │ PostgreSQL   │                                      │
│                   │              │                                      │
│                   │ - users      │                                      │
│                   │ - devices    │                                      │
│                   │ - measurements │                                    │
│                   │ - alerts     │                                      │
│                   └──────────────┘                                      │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  Frontend Container (React App - Smartphone)                            │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────┐   │
│  │  Pages/Views     │  │ Components       │  │ Services/Hooks     │   │
│  │                  │  │                  │  │                    │   │
│  │ - Dashboard      │  │ - MeasChart      │  │ - useWebSocket     │   │
│  │ - Alerts         │  │ - AlertCard      │  │ - useMeasurements  │   │
│  │ - Devices        │  │ - DeviceStatus   │  │ - API client       │   │
│  │ - Settings       │  │ - Notification   │  │ - useAuth          │   │
│  └──────────────────┘  └──────────────────┘  └────────────────────┘   │
│                              │                         │                │
│                              └─────────┬───────────────┘                │
│                           HTTP REST + WebSocket                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## LEVEL 3: Components

### ESP32 Components (Firmware)

```
SensorManager (Reads physical sensors)
├── DHT22Sensor
│   ├── init()           : Initialize GPIO
│   ├── read()          : Returns {temp, humidity}
│   └── isHealthy()     : Checks CRC
├── pHSensor (Atlas Scientific)
│   ├── init()          : Initialize Serial
│   ├── read()          : Returns pH value
│   └── calibrate()     : Calibration procedure
└── PowerMeter
    ├── init()          : Setup ADC
    ├── read()          : Returns kWh consumption
    └── reset()         : Clear counter

WiFi/ConnectivityManager
├── initWiFi()         : Connect to SSID
├── handleReconnect()  : Auto-reconnect logic
├── getStatus()        : Connected / IP address
└── getTiming()        : Latency to server

MQTTClient
├── connect()          : Establish MQTT connection
├── publish()          : Send measurement to topic
├── subscribe()        : Listen for commands
└── handleDisconnect() : Queue messages

HTTPClient (Fallback)
├── postMeasurement()  : HTTP POST to /api/measurements
├── getConfig()        : Fetch updated config
└── retry()            : Exponential backoff

StorageManager (SPIFFS)
├── writeLocal()       : Cache measurement if offline
├── readBatch()        : Get queued measurements
├── delete()           : After successful upload
└── getStats()         : Space usage, retention
```

### Backend Components

```
AuthService
├── signup()            : Create user account
├── login()            : Email + password → JWT
├── validateToken()    : JWT verification
└── refreshToken()     : Extend session

DeviceService
├── registerDevice()   : onboard new ESP32
├── getDevices()       : List user's devices
├── updateConfig()     : Send settings to device
└── reportStatus()     : Uptime, battery, errors

MeasurementService
├── recordMeasurement() : Store sensor reading
├── getMeasurements()   : Retrieve with filters
├── calculateStats()    : avg/min/max/trend
├── checkThresholds()   : Trigger alerts if exceeded
└── archive()           : Move old data

AlertService
├── createAlert()       : New threshold breach
├── getAlerts()        : User's alert history
├── acknowledge()       : Mark as read
└── notify()           : Send notification

DatabaseLayer (Prisma ORM)
├── Users
├── Devices
├── Measurements
├── Alerts
└── Configurations
```

### Frontend Components

```
Dashboard (Page)
├── MeasurementCard
│   ├── DisplayValue (current temp/pH)
│   ├── Trend (24h graph)
│   └── Status (last update, battery)
├── AlertSection
│   ├── AlertCard
│   ├── Acknowledge button
│   └── Clear all action
├── DeviceStatus
│   └── WiFi signal, battery %, last sync
└── Refresh button (manual data fetch)

Charts
├── LineChart (24h temperature trend)
├── AreaChart (pH history)
└── BarChart (Power consumption)

Notifications
├── Real-time toast (WebSocket)
├── Alert badge (count)
└── Sound/vibration (browser permissions)

Settings
├── Device config form
├── Alert threshold settings
├── Notification preferences
└── User profile
```

---

## LEVEL 4: Code - Data Models

```typescript
// ============= Database Schemas (PostgreSQL) =============

User {
  id: UUID (PK)
  email: String (unique)
  password_hash: String (bcrypt)
  created_at: DateTime
  updated_at: DateTime
}

Device {
  id: UUID (PK)
  user_id: UUID (FK → User)
  name: String
  model: String (e.g., "ESP32-WROOM-32")
  mac_address: String (unique)
  location: String (optional)
  active_sensors: String[] (["temperature", "ph", "power"])
  last_seen: DateTime
  battery_percent: Int (0-100)
  firmware_version: String
  created_at: DateTime
}

Measurement {
  id: UUID (PK)
  device_id: UUID (FK → Device)
  timestamp: DateTime (server-time, not device-time)
  readings: Sensor[] {
    name: String
    value: Float
    unit: String
  }
  created_at: DateTime
}

SensorReading {
  sensor_name: String (e.g., "temperature", "ph", "power_consumption")
  value: Float
  unit: String (e.g., "°C", "pH", "kWh")
  min_threshold: Float (optional)
  max_threshold: Float (optional)
}

Alert {
  id: UUID (PK)
  device_id: UUID (FK → Device)
  user_id: UUID (FK → User)
  measurement_id: UUID (FK → Measurement)
  type: Enum ("threshold_breach", "device_offline", "low_battery")
  severity: Enum ("INFO", "WARNING", "CRITICAL")
  message: String
  acknowledged: Boolean (default: false)
  created_at: DateTime
  acknowledged_at: DateTime (nullable)
}

// ============= API Payloads =============

// ESP32 → Server (MQTT payload)
MQTT_Measurement {
  deviceId: String (UUID)
  timestamp: ISO8601
  sensors: [
    {
      name: String
      value: Float
      unit: String
    }
  ]
}

// Frontend ← Server (WebSocket event)
WS_MeasurementUpdate {
  event: "measurement_received"
  data: {
    deviceId: String
    device_name: String
    measurements: Measurement
    alerts: Alert[] (if any triggered)
  }
}
```

---

## Data Flow

### 1️⃣ Normal Operation (Online)

```
┌────────────────────────────────────────────────────────────────┐
│                      Measurement Flow                          │
└────────────────────────────────────────────────────────────────┘

ESP32:
  1. SensorManager reads DHT22
     └─ temp=23.5°C, humidity=65%

  2. WiFi connects (cached SSID/pass)
  
  3. MQTT connects to broker
  
  4. Publish to /greenhouse/measurements
     └─ Payload: {deviceId, timestamp, sensors: [temp, humidity]}

Server:
  5. MQTT broker receives
  
  6. Backend MQTTClient subscribes
  
  7. MeasurementService.recordMeasurement()
     ├─ Validate data schema
     ├─ Store in PostgreSQL
     ├─ Check AlertService thresholds
     └─ Trigger alerts if needed

  8. Emit WebSocket event "measurement_received"
     └─ All connected clients receive

Frontend:
  9. WebSocket listener catches event
  
  10. Update React state (measurement + alerts)
  
  11. Dashboard re-renders with new data

Total latency: ~500ms (ESP32 MQTT pub → Frontend display)
```

### 2️⃣ Offline Mode (WiFi Lost)

```
ESP32:
  1. WiFi disconnects (or MQTT timeout)
  
  2. StorageManager.writeLocal() to SPIFFS
     └─ Queue: [measurement1, measurement2, ...]

  3. Continue reading sensors, queuing locally
  
  4. Retry WiFi connect (exponential backoff)

[Network restored]

  5. WiFi reconnects
  
  6. MQTT connects
  
  7. StorageManager.readBatch()
  
  8. Publish queued measurements in batch
  
  9. Delete from local storage

Server → Frontend: (Same as step 5-11 above)
```

### 3️⃣ Alert Triggered

```
Measurement arrives with temp=35°C (threshold=30°C)
  ↓
AlertService.checkThresholds() → BREACH DETECTED
  ↓
Create Alert record in DB
  ↓
Emit WebSocket: {event: "alert_triggered", severity: "WARNING"}
  ↓
Frontend receives → Show toast + badge
  ↓
Optional: Send FCM notification to phone
```

---

## Technology Stack

| Layer | Component | Technology |
|-------|-----------|-----------|
| **ESP32 Firmware** | Language | C++17, Arduino Framework |
| | IDE | PlatformIO |
| | Libraries | DHT.h, PubSubClient, SPIFFS, HTTPClient |
| | Sensors | DHT22, Atlas Scientific pH, Power meter |
| **Backend** | Runtime | Node.js 18 LTS |
| | Framework | Express.js |
| | Language | TypeScript (strict mode) |
| | ORM | Prisma |
| | Database | PostgreSQL 14 |
| | Real-time | socket.io (WebSocket) |
| | Auth | JWT + bcrypt |
| **Frontend** | Framework | React 19 |
| | Build | Vite |
| | Language | TypeScript |
| | Styling | Tailwind CSS |
| | Charts | recharts |
| | API | REST (fetch) + WebSocket |
| | Storage | localStorage (auth token) |

---

## Network Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Network Topology                             │
└─────────────────────────────────────────────────────────────────┘

ESP32 (Greenhouse - fixed location)
  │
  ├─ WiFi 2.4GHz (local network or external)
  │  └─ IP: Assigned by router (DHCP or static)
  │
  └─ MQTT over TLS
     └─ tcp://mosquitto.local:8883 (or external MQTT broker)
        └─ TLS Certificate validation
           └─ Fallback: HTTP POST to server


NAS Synology (local network or accessible via domain)
  │
  ├─ MQTT Broker (mosquitto)
  │  └─ Listens on :8883 (TLS)
  │
  ├─ Backend API (Express)
  │  └─ Listens on :8080 or 443 (HTTPS)
  │
  └─ WebSocket server (socket.io)
     └─ Integrated with Express on same port


Frontend (Smartphone - remote)
  │
  ├─ HTTPS connection to Server API
  │  └─ Fetch REST endpoints
  │
  └─ WebSocket connection to Server
     └─ Real-time measurement updates
```

---

## Constraints & Considerations

| Aspect | Constraint | Reason |
|--------|-----------|--------|
| **ESP32 Memory** | < 520KB RAM total | Microcontroller limited |
| | < 30KB per module | Multiple modules loaded |
| **ESP32 Storage** | SPIFFS max 2MB | Measurement cache only |
| **WiFi** | 2.4GHz only | ESP32 limited band |
| **MQTT** | Max 60KB payload | MQTT protocol limit |
| **Database** | PostgreSQL 14+ | Performance + JSON support |
| **Frontend Latency** | < 2s initial load | Mobile UX |
| **WebSocket** | < 1s message delivery | Real-time perception |
| **Scalability** | 100+ devices | Per-user isolation |

---

## Security Considerations

1. **ESP32 ↔ Server**: TLS/SSL for MQTT and HTTP (prevent man-in-the-middle)
2. **Credentials**: API keys not hardcoded (use config file or cert-based auth)
3. **Server ↔ Frontend**: HTTPS only, JWT for auth
4. **Database**: Parameterized queries (Prisma prevents SQL injection)
5. **Rate Limiting**: Protect API from brute force
6. **Data Privacy**: Measurements are per-user isolated

---

## 📚 Next Steps

- See `DOMAIN.md` for ubiquitous language & bounded contexts
- See `API.md` for detailed endpoint specifications
- See `SETUP.md` for local development setup

---

**Architecture complete! All layers documented.** 🏗️
