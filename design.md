# LifePause AI - System Design Document

## Document Information
- **Project**: LifePause AI - Silent Emergency Detection System
- **Version**: 1.0
- **Date**: February 11, 2026
- **Status**: Design Phase

---

## Introduction and Overview

### System Purpose

LifePause AI is an intelligent mobile safety application designed to detect and respond to silent emergencies where users are unable to manually trigger an SOS alert. The system continuously monitors sensor data and behavioral patterns to identify dangerous situations such as kidnapping, assault, fainting, phone snatching, or forced silence, automatically alerting trusted contacts with location information and emergency details.

### System Context

Traditional emergency response systems require conscious user action—pressing a panic button, making a call, or shouting for help. However, many real-world emergencies render victims unable to take these actions. LifePause AI addresses this critical gap by providing autonomous threat detection that operates silently in the background, ensuring help arrives even when the user cannot call for it.

### High-Level Concept

The system employs a multi-layered approach combining sensor fusion, machine learning, and behavioral analysis:

1. **Continuous Monitoring**: Collects data from microphone (noise levels only), accelerometer, gyroscope, GPS, screen activity, and network status
2. **Pattern Recognition**: Analyzes sensor data to detect anomalies indicating potential danger
3. **Intelligent Scoring**: Calculates real-time danger scores based on multiple threat indicators
4. **Automated Response**: Triggers multi-channel alerts to emergency contacts when critical thresholds are exceeded

### Key Innovations

**Silent Emergency Detection**: Unlike conventional systems, LifePause AI detects danger through absence of normal activity—no sound, no movement, no screen interaction—combined with contextual anomalies like unexpected location changes or unusual travel speeds.

**Crime-Zone Awareness**: Proactive alerting when users enter high-risk areas, with increased monitoring sensitivity and safe route suggestions to prevent emergencies before they occur.

**Forced Shutdown Detection**: Heartbeat monitoring system that detects when an attacker forcibly shuts down the device, triggering alerts based on last-known location even when the phone is offline.

**Routine-Based Anomaly Detection**: Machine learning models that establish user behavior baselines and identify deviations indicating potential danger, such as being in unexpected locations at unusual times.

---

## 1. System Architecture

### 1.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      MOBILE CLIENT (Android/iOS)                 │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    SENSOR LAYER                              ││
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      ││
│  │  │   GPS    │ │Microphone│ │Accelero- │ │Gyroscope │      ││
│  │  │ Location │ │  Noise   │ │  meter   │ │          │      ││
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘      ││
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                   ││
│  │  │  Screen  │ │ Network  │ │  Power   │                   ││
│  │  │ Activity │ │  Status  │ │  State   │                   ││
│  │  └──────────┘ └──────────┘ └──────────┘                   ││
│  └─────────────────────────────────────────────────────────────┘│
│                              ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                 PROCESSING LAYER                             ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │         Sensor Fusion Engine                          │  ││
│  │  │  - Data normalization  - Timestamp sync               │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │         Danger Scoring Engine                         │  ││
│  │  │  - Multi-factor analysis  - Threshold evaluation      │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │         Crime-Zone Module                             │  ││
│  │  │  - Geofencing  - Risk assessment                      │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
│                              ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                 ALERT ENGINE                                 ││
│  │  - SMS Gateway  - WhatsApp API  - Silent Call Initiator     ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                    FIREBASE BACKEND                              │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Heartbeat Monitor - Receives periodic pings every 5 min    ││
│  │  Detects device shutdown or connectivity loss               ││
│  └─────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Cloud Functions - Trigger alerts on heartbeat failure      ││
│  └─────────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Firestore Database - User profiles, contacts, crime zones  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Components and Modules

### 2.1 Silent Emergency Detector
**Purpose**: Identifies emergencies through absence of normal activity patterns.

**Inputs**: Noise level, screen state, movement data, time of day  
**Outputs**: Silent emergency score (0-100)  
**Logic**: Detects prolonged periods (>15 min) of no sound + no movement + no screen interaction during expected active hours.

### 2.2 Movement Analyzer
**Purpose**: Analyzes motion patterns to detect falls, sudden impacts, and abnormal movement.

**Inputs**: Accelerometer (x, y, z), gyroscope (pitch, roll, yaw)  
**Outputs**: Fall detected (boolean), impact severity (0-10), movement state (stationary/walking/running/vehicle)  
**Logic**: Uses magnitude thresholds and pattern matching to identify fall signatures and classify activity states.

### 2.3 Crime-Zone Analyzer
**Purpose**: Monitors user location against known high-risk areas and triggers proactive alerts.

**Inputs**: GPS coordinates, time of day, crime database  
**Outputs**: Crime-zone risk level (low/medium/high), distance to nearest safe zone  
**Logic**: Geofencing with polygon-based boundary detection, risk scoring based on historical crime data and time factors.

### 2.4 Kidnap Pattern Detector
**Purpose**: Identifies kidnapping scenarios through travel anomalies and forced movement patterns.

**Inputs**: GPS trajectory, speed, noise level, screen state  
**Outputs**: Kidnap probability score (0-100)  
**Triggers**: Unusual travel speed (>80 km/h when user typically walks), sudden location jumps, forced silence during movement  
**Logic**: Compares current travel patterns against learned baseline, flags deviations exceeding thresholds.

### 2.5 Shutdown Detection Module
**Purpose**: Detects forced device shutdown by attackers through heartbeat monitoring.

**Inputs**: Device power state, network connectivity, last heartbeat timestamp  
**Outputs**: Shutdown alert trigger (boolean)  
**Logic**: If heartbeat fails for >10 minutes during active hours, triggers emergency alert using last-known location.

### 2.6 Routine Learning Module
**Purpose**: Establishes user behavior baselines and detects anomalies indicating potential danger.

**Inputs**: Historical location data, time patterns, frequent places  
**Outputs**: Expected location, anomaly score (0-100)  
**Logic**: Machine learning model (clustering + time-series analysis) learns daily routines over 7 days, flags deviations from expected patterns.

### 2.7 Alert Manager
**Purpose**: Orchestrates multi-channel emergency notifications to trusted contacts.

**Inputs**: Danger score, emergency type, user location, contact list  
**Outputs**: SMS, WhatsApp messages, silent calls, location links  
**Logic**: Priority-based delivery (SMS first, then WhatsApp, then calls), retry mechanism for failed deliveries, 60-second grace period for false alarm cancellation.

---

## 3. Core Algorithms and Pseudocode

### 3.1 Danger Score Calculation

```
function calculateDangerScore():
    score = 0
    
    // Silent emergency detection (weight: 30)
    if (noSound AND noMovement AND noScreenActivity) for > 15min:
        score += 30
    
    // Movement anomalies (weight: 25)
    if fallDetected:
        score += 25
    else if suddenImpact:
        score += 15
    
    // Location anomalies (weight: 20)
    if inCrimeZone:
        score += 20
    if suddenLocationJump > 5km:
        score += 15
    
    // Travel anomalies (weight: 15)
    if travelSpeed > 80 km/h AND userTypicallyWalks:
        score += 15
    
    // Routine deviation (weight: 10)
    if currentLocation != expectedLocation AND currentTime in activeHours:
        score += 10
    
    return min(score, 100)
```

### 3.2 Silent Emergency Detection

```
function detectSilentEmergency():
    currentTime = getCurrentTime()
    
    if currentTime NOT in userActiveHours:
        return false  // User likely sleeping
    
    silenceDuration = getTimeSinceLastSound()
    movementDuration = getTimeSinceLastMovement()
    screenDuration = getTimeSinceLastScreenActivity()
    
    if silenceDuration > 15min AND movementDuration > 15min AND screenDuration > 15min:
        if NOT inSafeZone():
            return true
    
    return false
```

### 3.3 Forced Shutdown Detection

```
function monitorHeartbeat():
    every 5 minutes:
        sendHeartbeatToFirebase(deviceId, location, timestamp)
    
// Firebase Cloud Function
function onHeartbeatTimeout(deviceId):
    lastHeartbeat = getLastHeartbeat(deviceId)
    timeSinceLastBeat = now() - lastHeartbeat.timestamp
    
    if timeSinceLastBeat > 10min AND lastHeartbeat.time in activeHours:
        user = getUserProfile(deviceId)
        triggerEmergencyAlert(user.contacts, lastHeartbeat.location, "Device shutdown detected")
```

### 3.4 Crime-Zone Risk Amplification

```
function amplifyRiskInCrimeZone(baseScore):
    if NOT inCrimeZone():
        return baseScore
    
    crimeZoneRisk = getCrimeZoneRiskLevel()  // low=1.2, medium=1.5, high=2.0
    timeMultiplier = getTimeRiskMultiplier()  // night=1.5, day=1.0
    
    amplifiedScore = baseScore * crimeZoneRisk * timeMultiplier
    
    return min(amplifiedScore, 100)
```

---

## 4. Data Flow and Sequence Diagrams (Text Format)

### 4.1 User State Monitoring Flow

```
Sensors → Sensor Manager → Data Normalizer → Sensor Fusion Engine
                                                      ↓
                                            Feature Extractor
                                                      ↓
                                    ┌─────────────────┴─────────────────┐
                                    ↓                                   ↓
                          Movement Analyzer                  Silent Emergency Detector
                                    ↓                                   ↓
                          Routine Learning Module            Crime-Zone Analyzer
                                    ↓                                   ↓
                                    └─────────────────┬─────────────────┘
                                                      ↓
                                          Danger Scoring Engine
```

### 4.2 Danger Detection Flow

```
[Every 30 seconds]
1. Sensor Manager collects: GPS, noise level, accelerometer, gyroscope, screen state
2. Sensor Fusion Engine normalizes and timestamps data
3. Parallel processing:
   - Movement Analyzer → checks for falls, impacts, stationary state
   - Silent Emergency Detector → checks for prolonged inactivity
   - Crime-Zone Analyzer → checks if location in high-risk area
   - Kidnap Pattern Detector → checks for unusual travel patterns
   - Routine Learning Module → checks for location/time anomalies
4. Danger Scoring Engine aggregates all scores → final danger score (0-100)
5. If score > 60: Alert Manager notified
6. If score > 85: Immediate emergency alert triggered
```

### 4.3 Alerting Flow

```
Danger Score > 85 (Critical)
    ↓
Alert Manager receives trigger
    ↓
Start 60-second grace period → Show in-app notification "Are you safe?"
    ↓
User responds "I'm safe" → Cancel alert, log false positive
    ↓
No response after 60 seconds
    ↓
Alert Manager executes parallel delivery:
    ├─→ SMS Gateway → Send SMS to all contacts with location link
    ├─→ WhatsApp API → Send WhatsApp message with emergency details
    └─→ Silent Call Initiator → Place calls to primary contacts
    ↓
Log delivery status for each channel
    ↓
Send location updates every 5 minutes until emergency resolved
```

### 4.4 Shutdown Event Flow

```
Mobile App (Background Service)
    ↓
Every 5 minutes: Send heartbeat to Firebase
    ↓
Firebase Realtime Database updates: /heartbeats/{userId}/lastBeat
    ↓
[Device forcibly shut down by attacker]
    ↓
Heartbeat stops
    ↓
Firebase Cloud Function monitors heartbeat timestamps
    ↓
Detects: (currentTime - lastBeatTime) > 10 minutes
    ↓
Cloud Function retrieves: user profile, emergency contacts, last known location
    ↓
Cloud Function triggers: SMS + WhatsApp alerts with message "Device shutdown detected"
    ↓
Contacts receive: Last known location + timestamp + emergency type
```

---

## 5. User Interface Flow

```
App Launch → Permission Request (Location, Sensors, Notifications)
    ↓
Onboarding → Add Emergency Contacts (min 2) → Set Active Hours → Start 7-day Learning
    ↓
Dashboard (Main Screen):
    - Safety Status Indicator (Safe / Monitoring / Alert)
    - Current Danger Score (optional display)
    - Quick SOS Button (manual trigger)
    - Recent Activity Log
    ↓
Settings Screen:
    - Manage Emergency Contacts
    - Configure Safe Zones
    - Adjust Sensitivity (Low / Medium / High)
    - Set Quiet Hours
    - View Privacy Settings
```

## 6. Assumptions

- Users carry their phones during daily activities
- Users grant all necessary permissions (location, sensors, background activity)
- Emergency contacts have active phone numbers and respond to alerts
- GPS accuracy is within 20 meters in urban areas
- Network connectivity is available most of the time for heartbeat transmission
- Users have established routines that can be learned within 7 days
- Crime-zone data is available and reasonably accurate
- Device sensors function correctly and provide reliable data

## 7. Limitations

- Cannot detect emergencies if phone battery is completely depleted
- Reduced accuracy in areas with poor GPS signal (tunnels, dense buildings)
- May generate false positives during legitimate activities (sleeping, meditation, movies)
- Cannot function if user force-stops the app or disables background services
- Heartbeat monitoring requires active internet connection
- SMS/WhatsApp delivery depends on carrier and network availability
- Machine learning models require 7-day learning period for optimal accuracy
- Cannot prevent emergencies, only detect and alert after they occur

---

**End of Design Document**
