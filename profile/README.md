![ESP32](https://img.shields.io/badge/ESP32-C++-blue?logo=espressif)
![Node.js](https://img.shields.io/badge/Node.js-Express-green?logo=node.js)
![.NET](https://img.shields.io/badge/.NET_8-EF_Core-purple?logo=dotnet)
![React Native](https://img.shields.io/badge/React_Native-Expo-61DAFB?logo=react)
![Azure](https://img.shields.io/badge/Azure-SQL+App_Service-0078D4?logo=microsoftazure)

# SmartLock

A full-stack IoT smart lock system built from scratch spanning embedded firmware, cloud services, and a mobile app. The project combines C++ firmware on an ESP32 microcontroller, a Node.js WebSocket relay backend, a .NET EF Core database API secured with JWT, a React Native mobile app with NFC support, and Azure cloud infrastructure, all working together to deliver real-time RFID access control with offline resilience.

> RFID key fob access, remote lock/unlock, real-time status updates, offline event caching, configurable settings like auto-lock timers and custom unlock tones - all controlled from your phone!

## Repositories

| Repo | Description |
|---|---|
| [`SmartLock-Firmware`](https://github.com/Boudraa-SmartLock/SmartLock-Firmware) | ESP32 C++ firmware (PlatformIO) |
| [`SmartLock-Backend`](https://github.com/Boudraa-SmartLock/SmartLock-Backend) | Node.js WebSocket relay + REST API |
| [`SmartLock-DB-API`](https://github.com/Boudraa-SmartLock/SmartLock-DB-API) | ASP.NET Core database API |
| [`SmartLock-Mobile`](https://github.com/Boudraa-SmartLock/SmartLock-Mobile) | React Native mobile app (Expo) |
| [`SmartLock-WebApp`](https://github.com/Boudraa-SmartLock/SmartLock-WebApp) | Web dashboard |

## Table of Contents

- [Demo](#demo)
- [Architecture](#architecture)
- [Schema Diagram](#schema-diagram)
- [Features](#features)
  - [Hardware](#hardware)
  - [Software](#software)
- [Hardware](#hardware-1)
  - [Components](#components)
  - [Pin Assignments](#pin-assignments)
  - [Circuit Diagram](#circuit-diagram)
- [Setup](#setup)
  - [Firmware (ESP32)](#firmware-esp32)
  - [Backend (Node.js)](#backend-nodejs)
  - [DB API (ASP.NET Core)](#db-api-aspnet-core)
  - [Mobile App (Expo)](#mobile-app-expo)
- [Security](#security)
- [Contributing](#contributing)
- [Challenges & Lessons Learned](#challenges--lessons-learned)
- [Links to Other Docs](#links-to-other-docs)
- [License](#license)

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

## Setup

The backend and DB API are deployed to Azure App Service. The ESP32 firmware and mobile app connect to the deployed URLs.

| Service | URL |
|---|---|
| Backend (Node.js) | `https://smartlock-backend-ezhcfahsavavfehu.canadacentral-01.azurewebsites.net` |
| DB API (.NET) | `https://smartlock-db-api-cffjc8fthjfwb9hu.canadacentral-01.azurewebsites.net` |

### Firmware (ESP32)

1. Install [PlatformIO](https://platformio.org/)
2. Clone `SmartLock-Firmware`
3. Create `include/Config.h` from `Config.h.example`:
   ```cpp
   #pragma once

   #define WIFI_SSID     "YOUR_WIFI_SSID"
   #define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

   #define BACKEND_URL   "https://smartlock-backend-ezhcfahsavavfehu.canadacentral-01.azurewebsites.net"
   #define WS_URL        "wss://smartlock-backend-ezhcfahsavavfehu.canadacentral-01.azurewebsites.net/device"
   ```
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

Already deployed to Azure. To run locally for development:

1. Clone `SmartLock-Backend`
2. `npm install`
3. Create `.env`:
   ```env
   PORT=3000
   DB_API_URL=https://smartlock-db-api-cffjc8fthjfwb9hu.canadacentral-01.azurewebsites.net
   DEVICE_ID=your-device-guid
   BACKEND_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
   ```
4. `npm start`

### DB API (ASP.NET Core)

Already deployed to Azure. To run locally for development:

1. Clone `SmartLock-DB-API`
2. Set connection string in `appsettings.json` or environment:
   ```json
   { "ConnectionStrings": { "DefaultConnection": "Server=...;Database=smartlock;..." } }
   ```
3. Set `BACKEND_PUBLIC_KEY` environment variable (PEM format) or place `public.pem` in project root
4. Apply migrations: `dotnet ef database update`
5. `dotnet run`

### Mobile App (Expo)

1. Clone `SmartLock-Mobile`
2. `npm install`
3. Create `.env`:
   ```env
   EXPO_PUBLIC_API_URL=https://smartlock-backend-ezhcfahsavavfehu.canadacentral-01.azurewebsites.net
   EXPO_PUBLIC_WS_URL=wss://smartlock-backend-ezhcfahsavavfehu.canadacentral-01.azurewebsites.net/mobile
   ```
4. `npx expo start`

## Security

| Layer | Mechanism |
|---|---|
| ESP32 --> Backend | HMAC-SHA256 authentication with device-specific shared secret |
| Backend --> DB API | RS256 JWT signed with private key, verified by DB API with public key |
| RFID UIDs | Salted SHA-256 hash before storage or transmission |
| Device Secret | Stored in ESP32 NVS (encrypted flash partition) |
| Private Key | Azure Key Vault reference (production) or `.env` (development) |

## Contributing

1. Fork the relevant repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit changes: `git commit -m 'Add my feature'`
4. Push: `git push origin feature/my-feature`
5. Open a Pull Request

## Challenges & Lessons Learned

- **Relay EMI noise** crashing the MFRC522 RFID reader - solved with RC522 re-initialization after every relay actuation
- **WebSocket race conditions** between ESP32 dual cores - solved by keeping all WS operations on core 1
- **Offline-first architecture** - designing the event cache and bulk sync flow to handle multi-day outages gracefully
- **Cross-platform UID hashing** - ensuring identical SHA-256 output across C++ (mbedtls), TypeScript (expo-crypto), and .NET apps

## Links to Other Docs

- [API Specification](https://github.com/Boudraa-SmartLock/.github/blob/main/docs/API_SPEC.md)
- [Reflection](https://github.com/Boudraa-SmartLock/.github/blob/main/docs/REFLECTION.md)
- [Sequence Diagrams](https://github.com/Boudraa-SmartLock/.github/blob/main/docs/SEQ_DIAGRAMS.md)

## License

MIT — see [LICENSE](LICENSE) for details.
