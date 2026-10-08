I understand exactly what you want:

> **Any nearby phone, just because Wi-Fi is ON, should automatically receive an SOS message without installing anything.**

I will give you a deep technical research-style answer.

## Short answer

With normal Android/iOS phones, this is **not possible in a legal, reliable, cross-platform way**.

Wi-Fi being ON does **not** mean the phone is allowed to receive arbitrary emergency messages from nearby devices. Modern phones block this for security, privacy, battery, and spam-prevention reasons.

However, there are a few **nearby communication technologies** that come close, and I will explain all of them: what works, what does not work, and what is the best realistic path for your project.

---

# 1. Why “Wi-Fi ON” Is Not Enough

A phone Wi-Fi radio can be in several states:

## State A: Wi-Fi ON, but not connected to any network

Example:

```text
Phone Wi-Fi is enabled
Phone is not connected to any hotspot/router
```

In this state:

- The phone has no IP address.
- Your app cannot send UDP/TCP packets to it.
- It cannot receive normal internet/local network packets.
- It may scan for Wi-Fi networks in the background, but only at OS level.
- The OS does not expose received Wi-Fi management frames to normal apps.

So this does **not** work:

```text
Your SOS device sends Wi-Fi packet
        ↓
Nearby phone with Wi-Fi ON automatically receives SOS
```

Because the nearby phone is not part of the same network and has no active receiving socket.

---

## State B: Phone connected to same Wi-Fi network

Example:

```text
SOS phone connected to Wi-Fi router
Receiver phone connected to same Wi-Fi router
```

Now you can use:

```text
UDP broadcast
UDP multicast
mDNS
TCP socket
HTTP
```

But only if:

- Both phones are on the same Wi-Fi network.
- The receiving phone has an app listening.
- The OS allows background network listening.

This is possible:

```text
SOS phone sends UDP broadcast
        ↓
Same Wi-Fi network
        ↓
Receiver app listening on UDP port
        ↓
App shows SOS alert
```

But this is **not** “any phone nearby with Wi-Fi ON”.

It is:

```text
Any phone on the same Wi-Fi network with the receiver app installed.
```

---

# 2. Deep Research: Possible Wi-Fi Methods

I will now list the possible technical methods and explain whether they can achieve your requirement.

---

# Method 1: UDP Broadcast / Multicast on Same Wi-Fi

## How it works

SOS phone sends UDP packet to:

```text
255.255.255.255:50000
```

or multicast address:

```text
224.0.0.x
```

Receiver phones listen on that port.

## Can it reach any phone with Wi-Fi ON?

No.

It only reaches phones that are:

```text
Connected to the same Wi-Fi network
AND
Have an app listening
```

## Verdict

Good for:

```text
Home
Office
School
Hospital LAN
Event venue Wi-Fi
```

Not good for:

```text
Random nearby phones just with Wi-Fi ON
```

---

# Method 2: Wi-Fi Aware / Neighbor Awareness Networking, NAN

This is the closest Wi-Fi technology to your idea.

## What is Wi-Fi Aware?

Wi-Fi Aware allows nearby devices to discover services without being connected to the same Wi-Fi network.

Example:

```text
Phone A publishes service:
    emergency-sos

Phone B subscribes to service:
    emergency-sos

Phone B detects Phone A nearby
```

This can work even if both phones are not connected to a Wi-Fi router.

## Android support

Android has Wi-Fi Aware APIs:

```text
WifiAwareManager
PublishConfig
SubscribeConfig
WifiAwareSession
```

Android 8.0+ supports Wi-Fi Aware on supported devices.

## Can any phone receive SOS automatically?

No.

The receiving phone usually needs:

- Android OS
- Wi-Fi Aware supported hardware
- App installed that subscribes to the SOS service
- Proper permissions
- App allowed to run in background

There is no public Android API where your app can force every Android phone to show an emergency popup from Wi-Fi Aware without a receiver app.

## iOS support

iPhone does not provide a public Wi-Fi Aware API for normal developers.

So iPhone cannot be used as a generic receiver.

## Verdict

Wi-Fi Aware is technically interesting, but it does **not** solve:

```text
Any phone nearby automatically receives SOS
```

It only helps with:

```text
Android phones with your app installed
```

---

# Method 3: Wi-Fi Direct / Wi-Fi P2P

## How it works

Wi-Fi Direct allows devices to connect directly:

```text
Phone A becomes Wi-Fi Direct group owner
Phone B connects to Phone A
```

Android supports Wi-Fi Direct.

## Problem

The receiving user usually has to accept or choose the device.

Example:

```text
Phone A requests Wi-Fi Direct connection
Phone B shows prompt:
    Accept connection?
```

That is not automatic.

Also, Wi-Fi Direct is not designed to push emergency alerts to random nearby phones.

## Verdict

Not suitable for automatic SOS broadcast to any nearby phone.

---

# Method 4: Phone Creates Emergency Wi-Fi Hotspot

Your SOS phone could create a Wi-Fi hotspot:

```text
SSID: EMERGENCY_SOS_123
Password: open
```

Nearby people could manually connect to it.

After connecting, they could see a captive portal page:

```text
EMERGENCY ALERT
User needs help
Location: ...
Contact: ...
```

## Can nearby phones automatically connect?

Usually no.

Modern phones avoid automatically connecting to unknown open hotspots.

Sometimes they may show:

```text
Open network available
```

But this is not guaranteed, not instant, and not a true SOS push.

## Verdict

This can be a fallback:

```text
SOS phone becomes visible as EMERGENCY_SOS Wi-Fi network
```

But it is not automatic.

---

# Method 5: Broadcast SOS Inside Wi-Fi SSID Name

You can broadcast a Wi-Fi network name like:

```text
SOS_HELP_NEAR_GATE_2
```

or:

```text
EMERGENCY_CALL_911
```

Nearby phones may see this in Wi-Fi settings.

## Problem

This only appears if the user opens Wi-Fi settings or the OS shows available networks.

It does not create a popup.

Also SSID length is limited:

```text
Max 32 bytes
```

So message space is small.

## Verdict

This is a human-visible fallback, not an automatic message system.

---

# Method 6: Raw 802.11 Management Frames / Beacon / Probe Response

This is the deepest technical area.

Wi-Fi has low-level frames:

```text
Beacon frames
Probe request
Probe response
Action frames
Management frames
Vendor-specific information elements
```

In theory, a device with a special Wi-Fi adapter could broadcast custom Wi-Fi frames.

Some research projects use:

```text
Beacon frames
Probe responses
Action frames
Vendor IEs
```

to send small messages to nearby Wi-Fi devices.

## Big problems

### Problem 1: Normal phones do not show these frames to apps

Android/iOS Wi-Fi drivers process these frames internally.

They do not say:

```text
Hey app, here is a custom SOS frame
```

They usually ignore unknown vendor elements.

### Problem 2: Phones are not always listening to all channels

Wi-Fi scanning is controlled by OS.

It is not continuous.

### Problem 3: Sending raw management frames usually requires special hardware

Normal phone Wi-Fi chips do not allow raw frame injection.

You may need:

```text
External USB Wi-Fi adapter
Monitor mode support
Packet injection support
Linux system
Rooted/custom device
```

### Problem 4: Legal and ethical risk

Broadcasting crafted Wi-Fi management frames can interfere with networks and may violate radio/communication laws in many countries.

Spoofing networks or frames can also be illegal.

## Verdict

This is not a safe or practical path for a normal C++ mobile/emergency project.

---

# Method 7: Hotspot 2.0 / Passpoint / Wi-Fi CERTIFIED Passpoint

Passpoint is a Wi-Fi standard for automatic network discovery and secure connection.

It can provide metadata through ANQP elements, such as:

```text
Venue name
Operator name
Network authentication type
Emergency service information
```

Some standards include emergency-related information elements.

## Problem

Phones only auto-connect to Passpoint networks if they are provisioned or trusted.

A random SOS Passpoint network will not automatically push an alert to every phone.

Also, this is not exposed as a general emergency messaging API for apps.

## Verdict

Not suitable for your requirement.

---

# Method 8: Bluetooth Beacon / Eddystone / iBeacon

You mentioned Bluetooth too, so this is important.

BLE can broadcast small messages:

```text
iBeacon
Eddystone UID
Eddystone URL
Custom service data
```

Older Android systems had concepts like Physical Web / Nearby Notifications, where a beacon URL could sometimes produce a notification.

But today:

- Support is not reliable.
- iOS does not openly show arbitrary beacon URLs automatically.
- Google deprecated or reduced many Physical Web/Beacon notification experiences.
- Modern privacy rules block silent broadcasting to arbitrary phones.

## Verdict

BLE is excellent for app-to-app or app-to-device discovery.

It is not reliable for:

```text
Any phone nearby automatically receives SOS
```

---

# Method 9: Official Wireless Emergency Alerts / Cell Broadcast

This is the only real-world system that can reach many phones automatically.

Examples:

```text
Wireless Emergency Alerts, WEA
EU-Alert
CMAS
Presidential alerts
Amber alerts
Flood warnings
```

These use cellular broadcast / cell broadcast technology, not normal Wi-Fi.

## How it works

Mobile network operators broadcast emergency messages to all phones in a cell area.

Phones receive them automatically if supported by OS and carrier.

## Problem

Normal developers cannot send these alerts.

Only authorized government/emergency authorities can send them through mobile operators.

## Verdict

If your real requirement is:

```text
Every nearby phone receives an emergency alert automatically
```

then the correct technology is:

```text
Government/cellular emergency broadcast system
```

not a C++ app using Wi-Fi/Bluetooth.

---

# Method 10: Custom Android ROM / System App / MDM Controlled Devices

If you control the devices, you can build something powerful.

Example:

```text
Company phones
Campus devices
Factory devices
Security guard devices
```

You can install a system-level receiver app or device policy.

Then the device can:

```text
Scan BLE
Scan Wi-Fi Aware
Listen for UDP
Show full-screen emergency alert
```

Because it is preinstalled and trusted.

## Verdict

This works for controlled fleets.

But it does not work for random public phones.

---

# Comparison Table

| Method | Any phone with Wi-Fi ON? | Needs app? | Automatic? | Legal/practical? | Verdict |
|---|---:|---:|---:|---:|---|
| UDP on same Wi-Fi | No | Yes | Only if app listening | Yes | Good for LAN |
| Wi-Fi Aware | Partial | Yes | No | Yes | Android app only |
| Wi-Fi Direct | No | Yes/user approval | No | Yes | Not automatic |
| SOS hotspot | No | No, but manual connect | No | Yes | Human fallback |
| SSID message | No | No | No | Yes | Human-readable only |
| Raw 802.11 frames | Theoretically | No app, but OS ignores | No | Risky/legal issue | Not recommended |
| Passpoint/Hotspot 2.0 | No | No | No | Yes | Not for public SOS |
| BLE beacon | No | Usually yes | No | Yes | App receiver needed |
| Cell Broadcast/WEA | Yes | No | Yes | Government only | Not available to public apps |
| Custom ROM/MDM | Only controlled devices | System agent | Yes | Yes | Good for private fleet |

---

# Final Technical Conclusion

There are only three ways to truly achieve:

```text
Any nearby phone automatically receives SOS
```

## Way 1: Use official cellular emergency alert systems

This is the real solution.

But:

```text
Requires government/telecom integration
Not possible for a normal app
Not Wi-Fi/Bluetooth based
```

---

## Way 2: Use controlled devices with preinstalled system receiver

This is possible if you are building for:

```text
College campus
Factory
Hospital
Security team
Event staff
School devices
```

You can install your receiver on all devices.

Then your system can work.

---

## Way 3: Use an installed receiver app

This is the realistic app project path.

Change the requirement from:

```text
Any phone nearby with Wi-Fi ON automatically receives SOS
```

to:

```text
Any phone nearby with the receiver app installed can receive SOS
```

This is technically feasible.

---

# Honest Answer

If you require:

```text
Any random nearby phone
No app installed
No user interaction
Works with normal Android/iOS
Uses only Wi-Fi/Bluetooth
```

then the answer is:

```text
Not possible with normal public technology.
```

Anyone who says it is possible is likely assuming one of these:

1. Receiver app already installed.
2. Custom Android firmware.
3. Government/cellular emergency system.
4. Illegal Wi-Fi frame injection/spoofing.
5. User manually connects to hotspot.
6. User manually opens Wi-Fi settings.

---

# Best Realistic Project Direction

Since you want to build this in C++, I recommend this refined version:

## Project Name

**Multi-Channel Emergency SOS Mesh**

## Goal

When a user presses SOS, the system sends the alert through every available channel:

```text
1. Bluetooth Low Energy
2. Wi-Fi Aware, if supported
3. Same Wi-Fi network UDP broadcast
4. Internet/cloud server
5. Optional SMS fallback
6. Optional emergency Wi-Fi hotspot fallback
```

Nearby devices that have the receiver app can receive the SOS and show an alert.

This is technically strong and realistic.

---

# Closest Possible Design to Your Original Idea

If you still want the closest possible “no-app” experience, use a hybrid approach:

## Layer 1: Wi-Fi Aware for nearby Android devices with app

```text
SOS phone publishes Wi-Fi Aware service:
    sos.emergency.v1

Receiver app subscribes to:
    sos.emergency.v1

When detected:
    Show SOS alert
```

This can work without same Wi-Fi network.

But receiver app is required.

---

## Layer 2: BLE beacon for nearby devices with app

```text
BLE advertise SOS service UUID
Nearby app scans for UUID
Show alert
```

This works well for Android and partially for iOS with restrictions.

---

## Layer 3: Same Wi-Fi UDP broadcast

```text
If devices are on same Wi-Fi network
Send UDP SOS packet
Receiver app listens
```

---

## Layer 4: SOS Wi-Fi hotspot fallback

SOS device creates hotspot:

```text
SSID: SOS_HELP_ID_123
Open network
```

Inside hotspot, run a small web server:

```text
http://192.168.43.1/emergency
```

If someone connects manually, they see:

```text
Emergency SOS
User needs help
Location if available
Contact button
```

This is not automatic, but it can help in some cases.

---

## Layer 5: Internet cloud notification

```text
Send SOS to server
Server notifies trusted contacts
Server shows dashboard
Server sends push notification
```

This is the most reliable layer.

---

# Recommended Architecture

```text
                    SOS BUTTON
                        |
        +---------------+---------------+
        |               |               |
        v               v               v
   BLE Broadcast   Wi-Fi Aware     Internet Cloud
        |               |               |
        v               v               v
   Receiver App    Receiver App    Server Dashboard
        |               |               |
        +-------+-------+-------+-------+
                |               |
                v               v
         Show SOS Alert   Notify Trusted Contacts
```

Optional fallback:

```text
SOS Wi-Fi Hotspot
SSID contains emergency text
Captive portal page
```

---

# C++ System Design

Use C++ for the core protocol.

But understand this:

```text
C++ cannot directly access Android/iOS Wi-Fi Aware/BLE easily.
```

So structure the project like this:

```text
C++ Core
    Message encoding
    Security/signature
    Deduplication
    Retry logic
    Transport manager

Platform Adapter
    Android Kotlin/Java for BLE/Wi-Fi Aware
    iOS Swift/Objective-C for BLE
    Linux/Windows/ESP32 for sockets/BLE if supported
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
};

} // namespace sos
```

---

## transport.hpp

```cpp
#pragma once

#include <vector>
#include <functional>

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
};

} // namespace sos
```

---

## Possible Transports

```cpp
class BleTransport : public ITransport { ... };

class WifiUdpTransport : public ITransport { ... };

class WifiAwareTransport : public ITransport { ... };

class InternetTransport : public ITransport { ... };

class HotspotFallbackTransport : public ITransport { ... };
```

---

# Example Transport Logic

```cpp
void sendEmergency(const SosMessage& msg) {
    auto packet = msg.serialize();

    bleTransport.send(packet);
    wifiAwareTransport.send(packet);
    wifiUdpTransport.send(packet);
    internetTransport.send(packet);
}
```

---

# Android Wi-Fi Aware Concept

On Android, Wi-Fi Aware is accessed through Java/Kotlin APIs.

Your C++ core can generate the SOS payload, then pass it to Kotlin.

Kotlin side does:

```text
Create WifiAwareManager session
Publish SOS service
Send SOS payload
```

Receiver app does:

```text
Subscribe to SOS service
Receive SOS payload
Show emergency UI
```

Concept:

```text
C++ Core
   ↓ JNI
Android Wi-Fi Aware Adapter
   ↓
WifiAwareManager
```

---

# Why You Should Not Try Raw Wi-Fi Frame Injection

You may find research about sending SOS using raw Wi-Fi frames.

But I strongly recommend avoiding this for your project because:

1. Normal phones will not show the message.
2. It often requires special Wi-Fi adapters.
3. It may require root/custom firmware.
4. It can violate wireless regulations.
5. It can be treated as network abuse.
6. It is not reliable.
7. It will fail on iPhone and most Android devices.

For an emergency system, reliability and legality matter.

---

# If Your Requirement Is Fixed

If you absolutely require:

```text
Any nearby phone automatically receives SOS
```

then you must change the project domain.

You need to work with:

```text
Mobile network operators
Government emergency authorities
Cell broadcast centers
```

The technology is:

```text
Wireless Emergency Alerts
Cell Broadcast
Public Warning System
```

That is not a normal C++ app project.

It is telecom infrastructure.

---

# Best Project Statement You Can Use

You can present your project like this:

> This project is a multi-channel emergency SOS system. When the user triggers an SOS, the system attempts to notify nearby devices using Bluetooth Low Energy, Wi-Fi Aware where supported, local Wi-Fi UDP broadcast, and internet-based notification. The core protocol engine is implemented in C++. Because Android and iOS do not allow arbitrary apps to push alerts to nearby phones without a receiver component, the system targets nearby devices with the receiver application installed, while also providing internet-based notification to trusted contacts. A Wi-Fi hotspot fallback can display an emergency page to users who manually connect.

This sounds professional and realistic.

---

# What You Should Build First

## Phase 1: Android-only MVP

Build:

```text
SOS button
Send SOS over internet
Another app receives SOS from server
Show alert
```

This proves the emergency flow.

---

## Phase 2: Add BLE

Build:

```text
BLE SOS broadcast
Nearby Android app scans BLE
Shows SOS alert
```

---

## Phase 3: Add Wi-Fi Aware

Build:

```text
Android Wi-Fi Aware publisher
Android Wi-Fi Aware subscriber
Show SOS alert
```

---

## Phase 4: Add Same Wi-Fi UDP

Build:

```text
UDP broadcast sender
UDP listener receiver
Deduplicate messages
```

---

## Phase 5: Add Hotspot Fallback

Build:

```text
SOS phone creates hotspot
Simple emergency web page
Manual connect fallback
```

---

# Final Answer

There is **no legitimate way** to make every nearby phone automatically receive an SOS just because Wi-Fi is ON.

The closest legitimate methods are:

1. **Wi-Fi Aware**
   - Works only on supported Android devices.
   - Requires receiver app.

2. **BLE**
   - Requires receiver app.
   - Works better than Wi-Fi for proximity.

3. **Same Wi-Fi UDP**
   - Requires same network and receiver app.

4. **Internet/cloud**
   - Most reliable.
   - Requires internet.

5. **Official emergency alerts**
   - Can reach phones automatically.
   - Requires government/telecom integration.

6. **Custom controlled devices**
   - Possible if all devices have your system preinstalled.

So the best fine-tuned version of your idea is:

> A multi-channel SOS system that broadcasts emergency alerts to nearby devices that have the receiver app installed, and also sends alerts through internet/cloud notification. It uses BLE, Wi-Fi Aware, local Wi-Fi broadcast, and optional hotspot fallback, with a C++ core protocol engine.

That version is technically possible, legal, and impressive as a project.