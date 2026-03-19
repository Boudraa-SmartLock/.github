![ESP32](https://img.shields.io/badge/ESP32-C++-blue?logo=espressif)
![Node.js](https://img.shields.io/badge/Node.js-Express-green?logo=node.js)
![.NET](https://img.shields.io/badge/.NET_8-EF_Core-purple?logo=dotnet)
![React Native](https://img.shields.io/badge/React_Native-Expo-61DAFB?logo=react)
![Azure](https://img.shields.io/badge/Azure-SQL+App_Service-0078D4?logo=microsoftazure)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
# SmartLock

A full-stack IoT smart lock system built from scratch, with the following technologies used: ESP32 firmware in C++, Node.js backend for REST and WebSocket communication, .NET EF Core API for database management, React Native mobile app, and Azure cloud infrastructure.

> RFID key fob access, remote lock/unlock, real-time status, offline event caching, configurable settings - all controlled from your phone!

## 📑 Table of Contents

- [Demo](#demo)
- [Architecture](#️architecture)
- [Schema Diagram](#schema-diagram)
- [Features](#features)
  - [Hardware](#hardware)
  - [Software](#software)
- [Hardware Details](#-hardware)
  - [Components](#components)
  - [Pin Assignments](#pin-assignments)
  - [Circuit Diagram](#circuit-diagram)
- [Repositories](#-repositories)
- [Security](#-security)
- [Challenges & Lessons Learned](#-challenges--lessons-learned)
- [Contributing](#-contributing)
- [Sequence Diagrams](#sequence-diagrams)
  - [Smart Lock Boot](#smart-lock-boot)
  - [RFID Unlock](#rfid-unlock)
  - [Remote Lock/Unlock](#remote-lockunlock-from-mobile-app)
- [License](#-license)

## Demo

| Demo | Link |
|---|---|
| Key Registration Walkthrough | [Link to video](https://drive.google.com/file/d/1Lrqyny7eYYVuOpfj4T0UdPLoxUwSyFK3/view?usp=sharing) |

## Architecture

<img width="1364" height="458" alt="architecture_diagram" src="https://github.com/user-attachments/assets/02779569-5238-470a-9963-11ee3c6a7d6d" />


## Schema Diagram

<img width="867" height="587" alt="image" src="https://github.com/user-attachments/assets/be879562-936b-4bf7-acc6-b0d368b84be3" />

## Features

### Hardware
- **RFID unlock** - tap a registered NFC key fob to unlock
- **Physical buttons** - dedicated lock (outside) and unlock (inside) buttons
- **Reed switch** - detects door open/closed state
- **RGB status LED** - rainbow when connected, red flash when connecting, yellow when offline, green/red flash for unlock success/fail
- **Passive buzzer** - configurable unlock tones (Mario Coin, Bladee songs, Beatles melodies, and more)
- **Real-time clock** - DS3231 RTC with NTP sync for accurate timestamps (even when offline!)
- **Offline event caching** - stores up to 200 events on flash, syncs events with database in bulk when back online

### Software
- **Remote lock/unlock** - from the mobile app via WebSocket relay
- **Real-time status** - live lock state, door state, and connection status pushed to mobile
- **Auto-lock** - configurable timer that automatically locks after unlock
- **Door open warning** - buzzer alarm if door stays open too long
- **RFID key management** - register, delete, and color-code key fobs from the mobile app
- **NFC tap registration** - scan a key fob with your phone's NFC to register it
- **Event log** - full history of all lock/unlock/door events with key attribution
- **Configurable settings** - auto-lock delay, door warning delay, unlock tone selection
- **Dark mode** - manually toggled, persisted across app restarts
- **Offline resilience** - firmware operates fully offline using cached whitelist, caches new events, syncs events on reconnect
- **HMAC device authentication** - ESP32 authenticates via HMAC-SHA256 with a shared secret
- **Hashed UIDs** - RFID tag UIDs are salted and hashed before storage or transmission
- **JWT-secured DB API** - backend signs requests with JWT; DB API verifies

## Hardware

### Components

| Component | Model | Purpose |
|---|---|---|
| Microcontroller | ESP32 DevKit V1 | Main controller |
| RFID Reader | MFRC522 | Read NFC key fobs |
| Real-Time Clock | DS3231 | Accurate timestamps offline |
| Relay Module | 2-channel, active-low | Control solenoid |
| Solenoid Lock | 12V electric bolt | Physical locking mechanism |
| Reed Switch | Magnetic, normally open | Door open/closed detection |
| RGB LED | Common cathode | Status indicator |
| Buzzer | Passive | Tones and warnings |
| Buttons | Momentary push (×2) | Physical lock/unlock |
| Power | 12V 2A adapter | Solenoid power |

### Pin Assignments

| Pin | Component | Notes |
|---|---|---|
| GPIO 5 | MFRC522 SS | SPI chip select |
| GPIO 4 | MFRC522 RST | Reset |
| GPIO 18 | SPI SCK | Shared SPI clock |
| GPIO 19 | SPI MISO | Shared SPI data |
| GPIO 23 | SPI MOSI | Shared SPI data |
| GPIO 21 | DS3231 SDA | I2C data |
| GPIO 22 | DS3231 SCL | I2C clock |
| GPIO 26 | Relay IN | Solenoid control |
| GPIO 27 | Reed Switch | Door sensor (INPUT_PULLUP) |
| GPIO 32 | Lock Button | Physical lock (INPUT_PULLUP) |
| GPIO 33 | Unlock Button | Physical unlock (INPUT_PULLUP) |
| GPIO 25 | Passive Buzzer | Tone output |
| GPIO 13 | RGB Red | Status LED |
| GPIO 12 | RGB Green | Status LED |
| GPIO 14 | RGB Blue | Status LED |

### Circuit Diagram

<img width="3000" height="1885" alt="circuit_image (1)" src="https://github.com/user-attachments/assets/5a13300f-53da-4bc5-9b8e-19d0f6fc733c" />

## 📦 Repositories

| Repo | Description |
|---|---|
| [`SmartLock-Firmware`](https://github.com/adamboudruh/SmartLock-Firmware) | ESP32 C++ firmware (PlatformIO) |
| [`SmartLock-Backend`](https://github.com/adamboudruh/SmartLock-Backend) | Node.js WebSocket relay + REST API |
| [`SmartLock-DB-API`](https://github.com/adamboudruh/SmartLock-DB-API) | ASP.NET Core database API |
| [`SmartLock-Mobile`](https://github.com/adamboudruh/SmartLock-Mobile) | React Native mobile app (Expo) |
| [`SmartLock-WebApp`](https://github.com/adamboudruh/SmartLock-WebApp) | Web dashboard |

## 🚀 Setup

### Firmware (ESP32)

1. Install [PlatformIO](https://platformio.org/)
2. Clone `SmartLock-Firmware`
3. Create `include/Config.h` from `Config.h.example` with your WiFi credentials and backend URL
4. Provision device credentials to NVS (one-time):
   ```cpp
   Preferences prefs;
   prefs.begin("smartlock", false);
   prefs.putString("device_id", "YOUR_DEVICE_GUID");
   prefs.putString("device_secret", "YOUR_BASE64_SECRET");
   prefs.end();
   ```
5. Build and upload: `pio run -t upload`

### Backend (Node.js)

1. Clone SmartLock-Backend
2. `npm i`
3. Create `.env`:
   ```env
   PORT=3000
   DB_API_URL=https://your-db-api.azurewebsites.net
   DEVICE_ID=your-device-guid
   BACKEND_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
   ```
4. `npm start`

### DB API (ASP.NET Core)

1. Clone `SmartLock-DB-API`
2. Set connection string in `appsettings.json` or environment:
   ```json
   { "ConnectionStrings": { "DefaultConnection": "Server=...;Database=smartlock;..." } }
   ```
3. Set `BACKEND_PUBLIC_KEY` environment variable (PEM format) or place `public.pem` in project root
4. `dotnet ef database update` (apply migrations)
5. `dotnet run`

### Mobile App (Expo)

1. Clone `SmartLock-Mobile`
2. `npm install`
3. Create `.env`:
   ```env
   EXPO_PUBLIC_API_URL=http://your-backend-ip:3000
   EXPO_PUBLIC_WS_URL=ws://your-backend-ip:3000/mobile
   ```
4. `npx expo start`

## 🔒 Security

| Layer | Mechanism |
|---|---|
| ESP32 --> Backend | HMAC-SHA256 authentication with device-specific shared secret |
| Backend --> DB API | RS256 JWT signed with private key, verified by DB API with public key |
| RFID UIDs | Salted SHA-256 hash before storage or transmission |
| Device Secret | Stored in ESP32 NVS (encrypted flash partition) |
| Private Key | Azure Key Vault reference (production) or `.env` (development) |

## 🤝 Contributing

1. Fork the relevant repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit changes: `git commit -m 'Add my feature'`
4. Push: `git push origin feature/my-feature`
5. Open a Pull Request

## Sequence Diagrams

### Smart lock boot
## Sequence Diagrams
```mermaid
sequenceDiagram
    participant ESP as ESP32
    participant BE as Backend (Node.js)
    participant DB as DB API (.NET)

    Note over ESP: Power on / reboot
    ESP->>ESP: Init hardware (SPI, I2C, GPIO)
    ESP->>ESP: Mount LittleFS
    ESP->>ESP: Load whitelist from flash
    ESP->>ESP: Load settings from flash
    ESP->>ESP: Load credentials from NVS

    ESP->>ESP: Connect to WiFi
    alt WiFi connected
        ESP->>ESP: Sync RTC from NTP (if stale)
        ESP->>BE: WebSocket connect to /device
        BE-->>ESP: Connection opened

        ESP->>ESP: Build HMAC(deviceId:timestamp, secret)
        ESP->>BE: { deviceId, timestamp, hmac }

        BE->>DB: GET /devices/{id}/secret
        DB-->>BE: Base64 secret
        BE->>BE: Verify HMAC + check timestamp freshness

        alt Auth valid
            BE-->>ESP: { action: "AUTH_OK" }
            ESP->>BE: { event: "INIT", isLocked, isAjar }

            Note over BE: Trigger SYNC
            BE->>DB: GET /keys
            BE->>DB: GET /devices/{id}/settings
            DB-->>BE: keys[], settings[]
            BE-->>ESP: { action: "SYNC", whitelist, settings }
            ESP->>ESP: Save whitelist + settings to flash

            alt Has cached offline events
                ESP->>BE: { event: "OFFLINE_SYNC", events: [...] }
                BE->>DB: POST /events/bulk
                DB-->>BE: 201 Created
                BE-->>ESP: { action: "SYNC_OK" }
                ESP->>ESP: Clear event cache
            end

            BE-->>BE: Push STATE_UPDATE to mobile (if connected)
        else Auth invalid
            BE-->>ESP: Close connection (1008)
        end
    else WiFi failed
        Note over ESP: Continue in offline mode
        ESP->>ESP: Set status LED to yellow, cache future events
    end
```

### RFID Unlock
```mermaid
sequenceDiagram
    participant Fob as Key Fob
    participant ESP as ESP32
    participant BE as Backend
    participant DB as DB API
    participant App as Mobile App

    Fob->>ESP: Tap NFC tag
    ESP->>ESP: Read UID via MFRC522
    ESP->>ESP: Hash UID (SHA-256 + salt)
    ESP->>ESP: Check hashed UID against in-memory whitelist

    alt Match found in cached whitelist
        ESP->>ESP: pendingUnlock = true
        ESP->>ESP: Play unlock tone (buzzer)
        ESP->>ESP: Flash LED green 3×
        ESP->>ESP: Energize relay (unlock solenoid)
        ESP->>ESP: Start auto-lock timer

        alt Online
            ESP->>BE: Send "UNLOCK_SUCCESS" event with hashedUID
            BE->>DB: POST /events with respective eventTypeId
            DB->>DB: Resolve tagUid → keyId via Keys table
            DB-->>BE: 201 Created
            BE->>App: WS STATE_UPDATE { isLocked: false, isAjar, online: true }
            App->>App: Update dashboard UI
        else Offline
            ESP->>ESP: Cache event to LittleFS
        end
    else No match found
        ESP->>ESP: Flash LED red 3×

        alt Online
            ESP->>BE: Send "FAIL_UNLOCK" event
            BE->>DB: POST /events with respective eventTypeId
        else Offline
            ESP->>ESP: Cache event to LittleFS
        end
    end
```

### Remote Lock/Unlock from Mobile APP
```mermaid
sequenceDiagram
    participant App as Mobile App
    participant BE as Backend
    participant DB as DB API
    participant ESP as ESP32

    App->>BE: POST /devices/lock (or /unlock)

    BE->>BE: Check current state
    alt Already in requested state
        BE-->>App: 409 Conflict { error: "Already locked/unlocked" }
    else Device offline
        BE-->>App: 503 { error: "Device offline" }
    else OK
        BE->>ESP: WS { action: "LOCK" } (or "UNLOCK")

        ESP->>ESP: pendingLock = true (or pendingUnlock)
        ESP->>ESP: Actuate relay
        ESP->>ESP: notifyLocked() / notifyUnlocked()

        Note over ESP: For UNLOCK: play tone, start auto-lock timer
        Note over ESP: For LOCK: cancel auto-lock timer

        BE->>BE: Update tracked state
        BE->>DB: POST /events { eventTypeId: 2 or 5 (RemoteLock/Unlock) }
        DB-->>BE: 201

        BE->>App: WS STATE_UPDATE { isLocked, isAjar, online }
        BE-->>App: 202 Accepted { commandId }

        App->>App: Update dashboard UI
    end
```

## Challenges & Lessons Learned
- **Relay EMI noise** crashing the MFRC522 RFID reader - solved with RC522 re-initialization after every relay actuation
- **WebSocket race conditions** between ESP32 dual cores - solved by keeping all WS operations on core 1
- **Offline-first architecture** - designing the event cache and bulk sync flow to handle multi-day outages gracefully
- **Cross-platform UID hashing** - ensuring identical SHA-256 output across C++ (mbedtls), TypeScript (expo-crypto), and .NET apps

## 📄 License

MIT — see [LICENSE](LICENSE) for details.
