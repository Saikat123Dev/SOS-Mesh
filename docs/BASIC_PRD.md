Your idea is good, but it needs to be refined because the part saying:

> “nearby phones get SOS regardless of whether they have installed the system or not”

is mostly **not technically possible** on normal Android/iOS devices. Modern phones do not allow a random app to push a notification/alert to another nearby phone just because Bluetooth or Wi-Fi is on. The receiving phone usually must have an app installed, or the alert must go through an OS-level emergency system controlled by governments/mobile operators.

But we can refine your idea into a realistic and strong project:

# Refined Project Idea

## Project Name Example

**SOS Mesh Alert System**

## Core Idea

A user presses an emergency/SOS button. The phone/device broadcasts an emergency alert using:

1. **Bluetooth Low Energy / BLE**
   - For nearby devices that have the app installed and are scanning.

2. **Wi-Fi**
   - If devices are on the same Wi-Fi network, use local network broadcast/multicast.
   - If internet is available, send SOS to a cloud server.

3. **Internet / Cloud**
   - Send SOS to trusted contacts, nearby registered users, or emergency dashboard.

The goal becomes:

> Send an SOS alert to nearby devices that can receive it, and also send it through internet/mobile network whenever possible.

Not:

> Force every nearby phone to receive it without any app.

---

# Main Problem With the Original Idea

## 1. Bluetooth Does Not Allow Arbitrary Push to Nearby Phones

If your phone broadcasts over Bluetooth, nearby phones do not automatically show a popup.

For example:

- Android phones can receive Bluetooth/BLE data only if an app is scanning.
- iPhones heavily restrict background Bluetooth scanning.
- Normal phones do not show emergency notifications from unknown Bluetooth devices automatically.

So, Bluetooth can work like this:

```text
Your phone broadcasts SOS beacon
        ↓
Nearby phone with your app installed scans BLE
        ↓
App detects SOS
        ↓
App shows notification/alert
```

But it cannot work like this:

```text
Your phone broadcasts SOS
        ↓
Any random nearby phone automatically shows SOS
```

That second case is not allowed by Android/iOS for security and privacy reasons.

---

## 2. Wi-Fi Does Not Automatically Reach Nearby Phones

Wi-Fi is not a proximity broadcast technology by default.

If your phone sends a Wi-Fi packet, nearby phones will not receive it unless:

- They are connected to the same Wi-Fi network, or
- They are connected to your phone through hotspot/Wi-Fi Direct, or
- They are using Wi-Fi Aware/NAN, which still requires app support and OS support.

So Wi-Fi can be used like this:

```text
All phones connected to same Wi-Fi network
        ↓
SOS phone sends UDP broadcast/multicast
        ↓
Other phones with your app receive SOS
```

But not like this:

```text
Any phone nearby with Wi-Fi on automatically receives SOS
```

---

# Better Version of Your Idea

## Final Refined Project Statement

Build an emergency alert system where a user can press an SOS button, and the system tries multiple communication paths:

1. BLE broadcast to nearby app users.
2. Wi-Fi local broadcast to devices on the same network.
3. Internet upload to a server.
4. Optional mesh relaying by nearby devices.
5. Optional SMS/cellular fallback.
6. Optional hardware SOS button using ESP32 or similar device.

The system should be:

- Lightweight
- Privacy-preserving
- Works partially offline
- Uses C++ for core logic
- Uses platform APIs for Bluetooth/Wi-Fi/mobile UI

---

# Recommended System Architecture

## High-Level Architecture

```text
                 +-------------------+
                 |   SOS Sender App  |
                 |  Phone / Device   |
                 +---------+---------+
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
   BLE Broadcast     Wi-Fi Local       Internet
   Nearby Devices    UDP/Multicast     Cloud Server
          |                |                |
          v                v                v
   Nearby App Users  Same Network     Trusted Contacts
                     App Users        Emergency Dashboard
```

---

# Communication Layers

## Layer 1: Bluetooth Low Energy / BLE

Use BLE advertising.

### What it does

The SOS device broadcasts a small emergency beacon.

### Example BLE advertisement content

```text
Magic: SOS1
Type: EMERGENCY
Incident ID: 123e4567
Timestamp: 1710000000
Device ID: A1B2C3
Signature: optional
```

### Receiver behavior

A nearby phone with your app installed scans for this BLE service.

When detected:

```text
1. Validate message
2. Check duplicate
3. Show SOS alert
4. Optionally forward/rebroadcast
5. Optionally send response/ACK
```

### Important limitation

Only devices with your app can detect and respond properly.

---

## Layer 2: Wi-Fi Local Network

Use UDP broadcast or multicast when devices are on the same Wi-Fi network.

Example:

```text
Sender IP: 192.168.1.10
Broadcast: 192.168.1.255
Port: 50000
Protocol: UDP
```

Packet:

```json
{
  "version": 1,
  "type": "SOS",
  "incident_id": "abc-123",
  "sender_id": "user_001",
  "timestamp": 1710000000,
  "lat": 12.9716,
  "lon": 77.5946,
  "message": "Emergency help needed"
}
```

### Receiver behavior

The app listens on UDP port.

When SOS packet arrives:

```text
1. Verify message
2. Deduplicate
3. Show alert
4. Forward to server if internet available
```

### Limitation

This only works if devices are connected to the same Wi-Fi network.

---

## Layer 3: Internet / Cloud

This is the most reliable path.

When SOS is pressed:

```text
1. Get GPS location
2. Create SOS message
3. Send to server using HTTPS or MQTT
4. Server notifies trusted contacts/nearby registered users
```

This works even if BLE/Wi-Fi nearby fail.

---

# Recommended Features

## Core Features

### 1. SOS Button

A big emergency button in the app.

Possible activation methods:

- Button press
- Long press power button
- Shake gesture
- Volume button pattern
- Hardware button via BLE device

---

### 2. Location Sharing

When SOS is triggered:

```text
Get GPS location
Get network location if GPS unavailable
Include timestamp
Include accuracy
```

If location permission is denied, still send SOS without location.

---

### 3. Multiple Transport Channels

The app should try all available paths:

```text
BLE broadcast
Wi-Fi local broadcast
Internet HTTPS/MQTT
SMS fallback
```

Priority:

```text
Internet > BLE > Wi-Fi local > SMS
```

Or for offline mode:

```text
BLE > Wi-Fi local > Store and retry
```

---

### 4. Message Deduplication

The same SOS may be received multiple times through BLE, Wi-Fi, and internet.

Use:

```text
incident_id + sender_id + timestamp
```

or

```text
SHA256(message)
```

to detect duplicates.

---

### 5. SOS Cancellation

The sender should be able to cancel.

Message types:

```text
SOS_START
SOS_UPDATE
SOS_CANCEL
SOS_ACK
```

Example:

```json
{
  "type": "SOS_CANCEL",
  "incident_id": "abc-123",
  "timestamp": 1710000500
}
```

---

### 6. Nearby Receiver Alert

When a nearby device receives SOS:

```text
Show full-screen alert
Play loud sound
Vibrate
Show distance estimate if possible
Show direction only if available
Allow user to respond: "I can help" / "Not available"
```

---

### 7. Mesh Relay

If a nearby device receives SOS via BLE, it can rebroadcast it.

Example:

```text
Phone A sends SOS
Phone B receives via BLE
Phone B rebroadcasts SOS over BLE/Wi-Fi
Phone C receives from Phone B
```

Important:

- Limit relay count using TTL
- Prevent infinite loops
- Use signature or hash to avoid tampering

---

# Recommended Message Format

Since you want C++, use a compact binary format for BLE and maybe JSON for internet.

## C++ SOS Message Structure

```cpp
struct SosMessage {
    uint8_t version;
    uint8_t type;          // 1 = SOS_START, 2 = SOS_CANCEL, 3 = SOS_UPDATE
    uint64_t timestamp;
    uint8_t incident_id[16];
    uint8_t sender_id[8];
    int32_t latitude;      // latitude * 1e7
    int32_t longitude;     // longitude * 1e7
    uint16_t accuracy;     // meters
    uint8_t ttl;           // relay limit
    uint8_t signature[32]; // optional authentication
};
```

---

## Message Types

```cpp
enum class MessageType : uint8_t {
    SOS_START = 1,
    SOS_UPDATE = 2,
    SOS_CANCEL = 3,
    SOS_ACK = 4
};
```

---

## Why Binary Format?

BLE advertisements have small payload limits.

Legacy BLE advertisement size:

```text
Usually around 31 bytes
```

Extended BLE advertising can carry more, but older devices may not support it.

So for BLE, send a small message:

```text
incident_id
sender_id
timestamp
short signature/hash
```

Then the receiver can request more details using:

- BLE GATT characteristic
- Internet server
- Wi-Fi peer-to-peer connection

---

# C++ Project Design

Use C++ for the core engine, but understand this:

## C++ Cannot Directly Control Everything on Mobile

For Android/iOS:

- Bluetooth APIs are mostly Java/Kotlin on Android.
- Bluetooth APIs are Objective-C/Swift on iOS.
- C++ can be used through JNI or Objective-C++ bridges.
- UI is easier using Kotlin/Swift/Flutter/React Native/Qt.

So the best architecture is:

```text
C++ Core Library
    - Message encoding/decoding
    - Security/signature
    - Deduplication
    - Retry logic
    - Transport selection

Platform Adapter
    - Android BLE/Wi-Fi using Kotlin/Java + JNI
    - iOS BLE/Wi-Fi using Swift/Objective-C++
    - Desktop/embedded using native sockets and BLE libraries

UI App
    - Android/iOS app
    - Desktop app optional
    - Hardware device optional
```

---

# Suggested Technology Stack

## Option 1: Mobile-First with C++ Core

Best if your main target is Android/iOS.

```text
Core: C++
Android UI: Kotlin
iOS UI: Swift
Android BLE: Android Bluetooth API
iOS BLE: CoreBluetooth
Bridge: JNI / Objective-C++
Server: HTTPS + MQTT
Database: PostgreSQL + PostGIS
```

---

## Option 2: Qt C++ App

Best if you want maximum C++ usage.

```text
Core: C++
UI: Qt Widgets or Qt Quick
Bluetooth: Qt Bluetooth
Network: Qt Network
Server: MQTT/HTTPS
```

Limitations:

- Mobile BLE background behavior is still controlled by Android/iOS.
- Qt Bluetooth may not expose all advanced BLE features.

---

## Option 3: Embedded SOS Device

Best for a physical emergency button.

```text
Hardware: ESP32 / nRF52
Language: C++
BLE: ESP-IDF BLE / Arduino BLE
Wi-Fi: ESP-IDF Wi-Fi / Arduino Wi-Fi
Optional: GPS module, cellular modem
```

This can be a dedicated SOS button, wristband, or vehicle emergency device.

---

# Recommended System Flow

## SOS Sender Flow

```text
User presses SOS
        ↓
App collects:
    - User ID
    - Device ID
    - Timestamp
    - GPS location
    - Battery level
        ↓
Create incident ID
        ↓
Sign or hash message
        ↓
Start multi-channel broadcast:
    - BLE advertisement loop
    - Wi-Fi UDP broadcast loop
    - Internet upload
        ↓
Keep broadcasting until:
    - User cancels
    - Timeout
    - Battery saver mode
```

---

## SOS Receiver Flow

```text
App starts scanning:
    - BLE scan
    - Wi-Fi UDP listener
        ↓
Receives SOS packet
        ↓
Validate:
    - Version
    - Signature/hash
    - Timestamp
    - TTL
        ↓
Deduplicate using incident ID
        ↓
Show emergency alert
        ↓
Optional:
    - Forward/rebroadcast
    - Send ACK
    - Notify server
```

---

# Security and Trust

This is very important because it is an emergency system.

## Problems to Solve

1. Fake SOS alerts
2. Spam/replay attacks
3. Privacy leakage
4. Stalking through location
5. Battery drain
6. Network flooding

---

## Recommended Security Model

### Basic Version

Use a shared secret key among trusted devices/users.

```text
signature = HMAC-SHA256(secret_key, message)
```

Good for:

- Family group
- Campus system
- Office emergency system

---

### Better Version

Use public-key signatures.

Each device/user has:

```text
Private key: stored securely
Public key: registered with server/trusted contacts
```

SOS message is signed with private key.

Receiver verifies with public key.

Example:

```text
signature = Ed25519(private_key, message)
```

Recommended library:

```text
libsodium
```

---

## Anti-Replay Protection

Include:

```text
timestamp
incident_id
nonce
```

Receiver rejects messages older than a certain time.

Example:

```text
Reject if timestamp is older than 5 minutes.
```

---

## TTL for Relay

Prevent infinite mesh rebroadcast.

```text
TTL = 3
```

Each relay decreases TTL:

```text
TTL = TTL - 1
```

If TTL reaches 0, do not rebroadcast.

---

# Privacy Design

Do not broadcast full personal identity publicly.

Instead use:

```text
sender_id = temporary pseudonymous ID
```

The real identity is revealed only to trusted contacts or emergency server.

Example:

```text
BLE broadcast:
    sender_id = hash(user_id + day_key)

Internet server:
    maps sender_id to real user
```

This prevents random nearby people from knowing who sent the SOS.

---

# Legal and Safety Warning

Since this is an emergency system, be careful.

Your app should clearly say:

```text
This app is not a replacement for official emergency services.
```

Also:

- Do not automatically contact police/ambulance unless legally allowed.
- Get user consent before sharing location.
- Add false-alarm prevention.
- Allow easy cancellation.
- Log events securely.
- Follow local privacy laws.

---

# What “No App Required” Alternatives Exist?

If your main requirement is truly:

> Nearby phones receive alert without installing anything

then BLE/Wi-Fi alone are not enough.

Possible alternatives:

## 1. SMS to Nearby Known Contacts

You can send SMS to saved emergency contacts.

But not to random nearby phones.

---

## 2. Cell Broadcast

This is used by governments for emergency alerts.

But normal apps cannot use it. It requires telecom/government integration.

---

## 3. Bluetooth Beacon with URL

You can broadcast a URL using BLE beacon.

But most phones will not automatically show it unless they have supporting OS features or apps.

This is unreliable.

---

## 4. Wi-Fi Captive Portal

If your SOS device creates a Wi-Fi hotspot, nearby users can connect and see an emergency page.

But they must manually connect to your Wi-Fi.

Not ideal for emergencies.

---

# Realistic Project Scope

I recommend you divide the project into phases.

---

# Phase 1: Basic SOS App

Build this first.

## Features

```text
SOS button
Get GPS location
Send SOS to server
Trusted contacts receive notification
Cancel SOS
```

## Technology

```text
Mobile app
HTTPS server
Database
Push notifications
```

This phase alone is already useful.

---

# Phase 2: BLE SOS Beacon

Add Bluetooth.

## Features

```text
Broadcast SOS over BLE
Nearby app scans BLE
Nearby app shows alert
Deduplicate alerts
```

## Technology

```text
BLE advertising
BLE scanning
C++ message core
Platform BLE adapter
```

---

# Phase 3: Wi-Fi Local Alert

Add local network support.

## Features

```text
UDP broadcast on same Wi-Fi
Listen for SOS packets
Validate and show alert
```

## Technology

```text
UDP sockets
JSON or binary packet
C++ network module
```

---

# Phase 4: Mesh Relay

Add offline relay.

## Features

```text
Nearby phones rebroadcast SOS
TTL limit
Signature validation
Dedup cache
```

This is more advanced.

---

# Phase 5: Hardware SOS Device

Build a physical emergency button.

## Hardware Example

```text
ESP32
Button
Battery
Optional GPS
Optional buzzer
Optional LED
```

## Firmware Features

```text
Press button
Broadcast BLE SOS
Send Wi-Fi SOS
Send to server
```

This can be written in C++ using ESP-IDF or Arduino.

---

# Suggested C++ Core Library Structure

```text
sos-core/
│
├── CMakeLists.txt
│
├── include/
│   └── sos/
│       ├── message.hpp
│       ├── transport.hpp
│       ├── ble_transport.hpp
│       ├── wifi_transport.hpp
│       ├── crypto.hpp
│       ├── dedup.hpp
│       └── engine.hpp
│
├── src/
│   ├── message.cpp
│   ├── ble_transport.cpp
│   ├── wifi_transport.cpp
│   ├── crypto.cpp
│   ├── dedup.cpp
│   └── engine.cpp
│
├── platform/
│   ├── android/
│   ├── ios/
│   ├── linux/
│   └── esp32/
│
└── tests/
    ├── test_message.cpp
    ├── test_dedup.cpp
    └── test_engine.cpp
```

---

# C++ Core Example

## message.hpp

```cpp
#pragma once

#include <cstdint>
#include <array>
#include <vector>
#include <string>

namespace sos {

enum class MessageType : uint8_t {
    SOS_START = 1,
    SOS_UPDATE = 2,
    SOS_CANCEL = 3,
    SOS_ACK = 4
};

struct SosMessage {
    uint8_t version = 1;
    MessageType type = MessageType::SOS_START;

    std::array<uint8_t, 16> incident_id{};
    std::array<uint8_t, 8> sender_id{};

    uint64_t timestamp = 0;

    int32_t latitude = 0;   // latitude * 1e7
    int32_t longitude = 0;  // longitude * 1e7

    uint16_t accuracy_meters = 0;
    uint8_t ttl = 3;

    std::vector<uint8_t> signature;

    std::vector<uint8_t> serialize() const;
    bool deserialize(const std::vector<uint8_t>& data);

    double latitudeDouble() const {
        return latitude / 10000000.0;
    }

    double longitudeDouble() const {
        return longitude / 10000000.0;
    }
};

} // namespace sos
```

---

## transport.hpp

```cpp
#pragma once

#include <vector>
#include <functional>
#include <string>

namespace sos {

class ITransport {
public:
    virtual ~ITransport() = default;

    virtual bool start() = 0;
    virtual void stop() = 0;

    virtual bool send(const std::vector<uint8_t>& data) = 0;

    virtual void onReceive(
        std::function<void(const std::vector<uint8_t>&)> callback
    ) = 0;

    virtual std::string name() const = 0;
};

} // namespace sos
```

---

## dedup.hpp

```cpp
#pragma once

#include <string>
#include <unordered_set>
#include <chrono>

namespace sos {

class Deduplicator {
public:
    bool isNew(const std::string& key);
    void cleanup();

private:
    std::unordered_set<std::string> seen_;
};

} // namespace sos
```

---

## engine.hpp

```cpp
#pragma once

#include <memory>
#include <vector>
#include <functional>

#include "message.hpp"
#include "transport.hpp"
#include "dedup.hpp"

namespace sos {

class SosEngine {
public:
    void addTransport(std::shared_ptr<ITransport> transport);

    void start();
    void stop();

    void sendSos(const SosMessage& msg);
    void cancelSos(const SosMessage& msg);

    void onSosReceived(
        std::function<void(const SosMessage&)> callback
    );

private:
    std::vector<std::shared_ptr<ITransport>> transports_;
    Deduplicator dedup_;
    std::function<void(const SosMessage&)> sosCallback_;
};

} // namespace sos
```

---

## engine.cpp Example

```cpp
#include "engine.hpp"

namespace sos {

void SosEngine::addTransport(std::shared_ptr<ITransport> transport) {
    transports_.push_back(transport);
}

void SosEngine::start() {
    for (auto& transport : transports_) {
        transport->start();

        transport->onReceive([this](const std::vector<uint8_t>& data) {
            SosMessage msg;
            if (!msg.deserialize(data)) {
                return;
            }

            std::string dedupKey = msg.incident_id.data();
            dedupKey += std::to_string(msg.timestamp);

            if (!dedup_.isNew(dedupKey)) {
                return;
            }

            if (sosCallback_) {
                sosCallback_(msg);
            }
        });
    }
}

void SosEngine::stop() {
    for (auto& transport : transports_) {
        transport->stop();
    }
}

void SosEngine::sendSos(const SosMessage& msg) {
    auto data = msg.serialize();

    for (auto& transport : transports_) {
        transport->send(data);
    }
}

void SosEngine::cancelSos(const SosMessage& msg) {
    SosMessage cancelMsg = msg;
    cancelMsg.type = MessageType::SOS_CANCEL;

    auto data = cancelMsg.serialize();

    for (auto& transport : transports_) {
        transport->send(data);
    }
}

void SosEngine::onSosReceived(
    std::function<void(const SosMessage&)> callback
) {
    sosCallback_ = callback;
}

} // namespace sos
```

---

# Example Platform Adapter Concept

For Android:

```text
Kotlin App
    ↓ JNI
C++ SosEngine
    ↓ JNI callback
Kotlin BLE Broadcaster / Scanner
```

For iOS:

```text
Swift App
    ↓ Objective-C++ bridge
C++ SosEngine
    ↓ Swift callback
CoreBluetooth
```

For ESP32:

```text
ESP32 C++ firmware
    ↓
BLE + Wi-Fi drivers
    ↓
SosEngine core
```

---

# Recommended Transport Implementation Strategy

## BLE Transport

The C++ core should not directly call Android/iOS BLE APIs.

Instead:

```cpp
class BleTransport : public ITransport {
public:
    bool start() override;
    void stop() override;
    bool send(const std::vector<uint8_t>& data) override;
    void onReceive(std::function<void(const std::vector<uint8_t>&)>) override;
};
```

But inside Android:

```text
BleTransport calls Kotlin BLE broadcaster through JNI
```

Inside iOS:

```text
BleTransport calls CoreBluetooth through Objective-C++
```

Inside ESP32:

```text
BleTransport uses ESP-IDF BLE APIs
```

---

# Wi-Fi Transport Using UDP

This part can be pure C++ on many platforms.

Example idea:

```cpp
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
```

Sender:

```text
Create UDP socket
Enable broadcast
Send packet to 255.255.255.255:50000
```

Receiver:

```text
Bind UDP port 50000
Listen for broadcast packets
Parse message
```

This works well on:

- Linux
- Windows
- macOS
- Android NDK partially
- Embedded Linux

But mobile apps still need proper permissions and background restrictions.

---

# Important Mobile Permissions

Your app will need:

## Android

```text
Bluetooth
Bluetooth Admin
Bluetooth Scan
Bluetooth Advertise
Bluetooth Connect
Access Fine Location
Access Background Location
Internet
Access Network State
Foreground Service
Post Notifications
```

## iOS

```text
Bluetooth Peripheral
Bluetooth Central
Location When In Use
Background Modes
```

---

# Recommended MVP

If you want a working project quickly, build this first:

## MVP Version

```text
1. Android app only
2. One SOS button
3. Send SOS over internet to server
4. Broadcast SOS over BLE
5. Another Android phone with same app receives BLE SOS
6. Show alert on receiver phone
```

Do not start with:

```text
Every phone without app
Full mesh
iPhone background scanning
Hardware device
```

Those are advanced.

---

# Improved Final Idea Statement

You can use this as your project description:

> This project is an emergency alert system that allows a user to broadcast an SOS signal using multiple communication channels. When the SOS button is pressed, the system attempts to notify nearby users through Bluetooth Low Energy and local Wi-Fi networks, while also sending the alert through the internet when available. Nearby devices with the application installed can receive, validate, display, and optionally relay the emergency message. The system is designed to work in low-connectivity environments and uses message deduplication, security signatures, TTL-based relaying, and privacy-preserving identifiers. The core protocol and message engine are implemented in C++, while platform-specific Bluetooth/Wi-Fi features are integrated through Android/iOS/embedded adapters.

---

# What You Should Not Promise

Avoid saying:

```text
Works on any phone without installation.
Nearby phones automatically receive SOS.
100% guaranteed emergency delivery.
Can replace police/ambulance systems.
```

Instead say:

```text
Works best with devices that have the app installed.
Uses BLE and Wi-Fi as best-effort nearby alert channels.
Uses internet/cloud for wider notification.
Designed as a supplementary emergency assistance tool.
```

---

# Final Recommended Direction

Your best project path is:

```text
C++ core protocol library
+
Android app first
+
BLE SOS broadcast/scanning
+
Wi-Fi local UDP broadcast
+
Cloud server for reliable delivery
+
Optional ESP32 SOS button later
```

This gives you:

- Strong C++ usage
- Real networking
- Bluetooth
- Mobile development
- Security
- Emergency system design
- Hardware expansion later

It becomes a very good final-year or portfolio project if you clearly state the limitations and build it in phases.