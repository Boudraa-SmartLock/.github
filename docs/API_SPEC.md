# SmartLock API Specification

## Backend API (Node.js)

The backend serves as the relay layer between the mobile app and the ESP32 device. It forwards requests to the DB-layer API, manages WebSocket connections, and triggers device syncs on database changes.

Base URL: http://localhost:3000 (dev) / deployed Azure App Service URL (prod)

### Commands

- **POST /devices/lock** - Send lock command to ESP32
  - 202 - `{ accepted, commandId }`
  - 409 - already locked
  - 503 - device offline

- **POST /devices/unlock** - Send unlock command to ESP32
  - 202 - `{ accepted, commandId }`
  - 409 - already unlocked
  - 503 - device offline

### Device State & Settings

- **GET /devices/state** - Current device state
  - 200 - `{ isLocked, isAjar, online }`

- **GET /devices/settings** - Get all device settings
  - 200 - `DeviceSetting[]`
  - 500 - DB API unreachable

- **PUT /devices/settings** - Update settings + sync to device
  - Body: `{ "settings": [{ "settingId": 1, "value": "1" }, ...] }`
  - 200 - `DeviceSetting[]`
  - 400 / 404 / 500

### Keys

- **GET /keys** - List all registered keys
  - 200 - `Key[]`

- **POST /keys/register** - Register a new key fob
  - Body: `{ "name": "Red Key", "tagUid": "a1b2c3...", "color": "#EF4444" }`
  - 201 - `{ keyId }`
  - 400 - missing fields
  - 409 - duplicate UID

- **DELETE /keys/:id** - Delete key + resync device whitelist
  - 200 - `{ deleted: true }`
  - 404 / 500

- **POST /keys/sync** - Force whitelist sync to device
  - 200 - `{ synced: true }`
  - 503 - device offline

### Events

- **GET /events** - List all events (newest first)
  - 200 - `Event[]`

- **DELETE /events** - Clear all events
  - 200 - `{ cleared: true }`

### WebSocket Endpoints

- **/device** - ESP32 connection. Authenticates with HMAC-SHA256 generated using shared secret, then exchanges JSON messages for state events, commands, SYNC payloads, and offline bulk uploads.

- **/mobile** - Mobile app connection. Open (no auth currently). Receives real-time `STATE_UPDATE` pushes whenever lock/door state changes.

---

## DB API (ASP.NET Core / EF Core)

The DB API is the only layer with direct database access. All data mutations flow through here. Every request requires a valid `Authorization: Bearer <JWT>` header signed with RS256, and in the current architecture only the main backend server can generate a valid token.

Base URL: https://localhost:7110 (dev) / deployed Azure App Service URL (prod)

### Devices

- **GET /devices/{deviceId}** - Get device info
  - 200 / 404

- **GET /devices/{deviceId}/secret** - Get device secret (Base64-encoded)
  - 200 / 404

- **GET /devices/{deviceId}/settings** - Get merged settings (device overrides + global defaults)
  - 200 / 404

- **PUT /devices/{deviceId}/settings** - Upsert device settings
  - Body: `{ "settings": [{ "settingId": 1, "value": "0" }, ...] }`
  - 200 - returns full merged settings list
  - 400 - invalid settingId
  - 404 - device not found

Response shape for a single setting:
```json
{ "settingId": 1, "name": "AutoLockEnabled", "value": "0", "defaultValue": "1" }
```

### Events

- **POST /events** - Insert a single event
  - Body: `{ "eventTypeId": 6, "deviceId": "guid", "tagUid": "hashedUid", "createdAt": "2026-03-17T15:43:49.000Z" }`
  - 201 / 400 (invalid eventTypeId)

- **GET /events** - List events (newest first, max 200)
  - 200

- **DELETE /events** - Clear all events
  - 204

- **POST /events/bulk** - Bulk insert offline events
  - Body: `{ "events": [{ "eventTypeId": 1, "deviceId": "guid", "tagUid": null, "createdAt": "..." }, ...] }`
  - 201 - `{ data: count }`
  - 400 - empty or invalid

Response shape for a single event:
```json
{
  "eventId": "guid",
  "eventType": "SuccessKeyUnlock",
  "deviceId": "guid",
  "keyId": "guid | null",
  "keyName": "Red Key | null",
  "createdAt": "2026-03-17T15:43:49Z"
}
```

### Keys

- **GET /keys** - List all keys (ordered by createdAt)
  - 200

- **POST /keys** - Insert new key
  - Body: `{ "name": "Blue Key", "tagUid": "hashedUid", "color": "#3B82F6" }`
  - 201 / 400 (missing name or tagUid) / 409 (duplicate UID)

- **DELETE /keys/{id}** - Delete key and nullify its reference on all related events
  - 200 / 404

Response shape for a single key:
```json
{ "keyId": "guid", "name": "Blue Key", "tagUid": "a1b2c3...", "color": "#3B82F6", "createdAt": "..." }
```

---

## Reference

### Event Types

- 1 - **ButtonLock** - physical lock button pressed
- 2 - **RemoteLock** - mobile app lock command
- 3 - **AutoLock** - auto-lock timer expired
- 4 - **ButtonUnlock** - physical unlock button pressed
- 5 - **RemoteUnlock** - mobile app unlock command
- 6 - **SuccessKeyUnlock** - whitelisted RFID key tapped
- 7 - **FailKeyUnlock** - unknown RFID key tapped
- 8 - **Open** - door opened (reed switch)
- 9 - **Close** - door closed (reed switch)

### Settings

- 1 - **AutoLockEnabled** (bool, default 1) - enable auto-lock after unlock
- 2 - **AutoLockDelaySec** (int, default 30) - seconds before auto-lock triggers
- 3 - **DoorOpenBuzzEnabled** (bool, default 1) - enable door-open-too-long buzzer
- 4 - **DoorOpenBuzzDelaySec** (int, default 60) - seconds before door warning buzzer
- 5 - **OpenCloseToneId** (int, default 1) - unlock tone selection (0 = silent, 1–7 = melodies)

### Response Envelope (DB API only)

Every DB API response is wrapped in a `Status<T>` envelope:
```json
{
  "statusCode": 200,
  "statusDetails": null,
  "data": { ... }
}
```
The backend unwraps this before forwarding to the mobile app, so the mobile app never sees the `Status<T>` wrapper.