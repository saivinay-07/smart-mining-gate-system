# Smart Vehicle Entry & Exit Monitoring System

**Project Type:** AI-based CCTV Vehicle Monitoring System
**Version:** 1.0
**Status:** Planning / Pre-Development
**Primary Goal:** Reduce manual work at gate operations

---

# 1. Project Overview

The current process at the site requires people to monitor CCTV cameras and manually record vehicle information when lorries enter and leave the site.

The main information that needs to be recorded is:

* Vehicle number
* Entry time
* Exit time

This project will use CCTV cameras and AI to automatically identify vehicles and record their entry and exit times.

The application will provide a dashboard where managers and gate operators can view the vehicle records.

The main purpose is **not to completely remove people from the process immediately**, but to reduce repetitive manual work and make the process faster and more reliable.

---

# 2. Problem Statement

Currently, gate operators need to manually:

1. Watch CCTV cameras.
2. Identify incoming lorries.
3. Note the vehicle number.
4. Record the entry time.
5. Identify the same vehicle when it leaves.
6. Record the exit time.
7. Maintain records for future reference.

This becomes difficult when:

* Many vehicles arrive at the same time.
* Multiple CCTV cameras are being monitored.
* The site operates for long hours.
* Night shifts are required.
* Operators make mistakes while reading or writing vehicle numbers.
* Previous vehicle records need to be searched.

The proposed system will automate most of this repetitive work.

---

# 3. Project Goal

The primary goal is:

> **Automatically record vehicle numbers, entry times, and exit times using CCTV and AI, reducing manual gate-operation work.**

The system should allow a manager to see what vehicles entered and exited without depending completely on manually written records.

---

# 4. Users

The first version will have two types of users.

## 4.1 Manager

The manager can:

* View the dashboard.
* View today's vehicles.
* Search vehicle records.
* View entry and exit times.
* View previous records.
* View system status.

## 4.2 Gate Operator

The gate operator can:

* View live vehicle activity.
* View detected vehicle numbers.
* Check whether the AI detection is correct.
* Correct a vehicle number when AI makes a mistake.
* View entry and exit records.

The operator is therefore still available as a verification layer, especially during the first version of the system.

---

# 5. Scope of Version 1

The first version will focus on only three important pieces of information:

```text
Vehicle Number
Entry Time
Exit Time
```

The system will use CCTV to detect vehicles and identify their number plates.

### Example

A lorry enters:

```text
Vehicle Number: AP39AB1234
Entry Time: 09:42:13 AM
```

Later the same lorry leaves:

```text
Vehicle Number: AP39AB1234
Exit Time: 10:25:41 AM
```

The system should combine these into one vehicle visit:

```text
AP39AB1234

Entry: 09:42:13 AM
Exit: 10:25:41 AM
Duration: 43 minutes
```

---

# 6. What We Are NOT Building in Version 1

To keep the project simple and achievable, the following features are outside the first version:

* Sand quantity detection
* Weighbridge integration
* Driver face recognition
* Driver database
* GPS tracking
* Payment management
* Invoice generation
* WhatsApp integration
* Mobile application
* Automatic government/e-way bill integration
* AI chatbot
* Multiple-site management

These can be considered later.

---

# 7. Basic System Workflow

The expected workflow is:

```text
                    CCTV Camera
                         |
                         v
                 Vehicle Detection
                         |
                         v
                Number Plate Detection
                         |
                         v
                      OCR
                         |
                         v
                Vehicle Number Found
                         |
                         v
                  Event Processing
                         |
              +----------+----------+
              |                     |
          ENTRY EVENT           EXIT EVENT
              |                     |
              +----------+----------+
                         |
                         v
                    PostgreSQL
                         |
                         v
                  Backend API
                         |
                         v
                 Web Dashboard
```

---

# 8. Entry Process

When a vehicle approaches the entry area:

### Step 1

CCTV captures the vehicle.

### Step 2

AI detects that the object is a vehicle/lorry.

### Step 3

AI identifies the number plate.

### Step 4

OCR reads the number plate.

Example:

```text
AP39AB1234
```

### Step 5

The system creates an entry record.

```text
Vehicle: AP39AB1234
Event: ENTRY
Time: 09:42:13
Camera: Entry Camera
```

### Step 6

The system stores the record in the database.

---

# 9. Exit Process

When the same vehicle reaches the exit:

### Step 1

Exit CCTV detects the vehicle.

### Step 2

AI reads the number plate.

```text
AP39AB1234
```

### Step 3

The system searches for an existing open entry record.

### Step 4

If a matching vehicle is found, the system records:

```text
Exit Time: 10:25:41
```

### Step 5

The visit is marked as completed.

```text
AP39AB1234

Entry: 09:42:13
Exit: 10:25:41
Status: COMPLETED
```

---

# 10. Important System Rule

The system must not create a new vehicle visit every time the camera sees the same lorry.

For example, one lorry may appear in 100 camera frames.

The system should NOT create:

```text
AP39AB1234 - ENTRY
AP39AB1234 - ENTRY
AP39AB1234 - ENTRY
AP39AB1234 - ENTRY
...
```

Instead, the AI/tracking system should understand that these detections belong to the same vehicle.

This is one of the important technical problems we will solve during development.

---

# 11. Vehicle Visit Concept

Instead of thinking only about individual camera detections, our application will think in terms of a **Vehicle Visit**.

Example:

```text
Vehicle Visit
----------------------------

Vehicle Number : AP39AB1234

Entry Time     : 09:42 AM
Exit Time      : 10:25 AM

Status         : COMPLETED
```

A vehicle that has entered but has not yet exited:

```text
Vehicle Number : AP05XY5678

Entry Time     : 11:10 AM
Exit Time      : -

Status         : INSIDE
```

This makes the application easier to understand.

---

# 12. Main Application Modules

Version 1 will contain the following modules.

## 12.1 Authentication

Users should log in before accessing the application.

Example:

```text
Email
Password
```

Roles:

```text
MANAGER
OPERATOR
```

---

## 12.2 Dashboard

The dashboard should show simple information.

Example:

```text
Today's Vehicles       126

Currently Inside        18

Completed Visits       108

AI Detections          132
```

Below that:

```text
Recent Vehicles

------------------------------------------------
Vehicle       Entry       Exit       Status
------------------------------------------------
AP39AB1234    09:42       10:25      Completed
AP05XY5678    10:10       --         Inside
AP37CD9012    10:18       10:45      Completed
```

---

# 13. Vehicle History

Users should be able to search previous records.

Example:

```text
Search Vehicle Number

[ AP39AB1234 ]     [Search]
```

Result:

```text
Vehicle: AP39AB1234

Date          Entry       Exit
------------------------------------
08-Aug-2026   09:42       10:25
07-Aug-2026   11:15       12:02
06-Aug-2026   08:51       09:34
```

---

# 14. Live Monitoring

The system should eventually provide a live monitoring page.

Example:

```text
ENTRY CAMERA

[ LIVE VIDEO ]

Detected Vehicle:
AP39AB1234

Confidence:
96%

Status:
ENTRY DETECTED
```

For Version 1, this can initially be kept simple.

We don't need to build a complicated CCTV management system.

---

# 15. AI System

The AI portion will be a separate service.

We will use Python for AI because the computer-vision ecosystem is stronger there.

Basic pipeline:

```text
CCTV
  |
  v
Video Frame
  |
  v
Vehicle Detection
  |
  v
Number Plate Detection
  |
  v
OCR
  |
  v
Vehicle Number
```

The first AI version will use **pre-trained models** rather than training a model from scratch.

This is important because you are new to AI.

---

# 16. AI Responsibilities

The AI service should answer questions such as:

### What is in this frame?

```text
Truck
```

### Where is the number plate?

```text
Plate bounding box
```

### What does the plate say?

```text
AP39AB1234
```

### How confident are we?

```text
96%
```

The AI service should return structured information to the backend.

Example:

```json
{
  "vehicleNumber": "AP39AB1234",
  "confidence": 0.96,
  "eventType": "ENTRY"
}
```

The backend should then decide what to do with this information.

---

# 17. Important Design Decision

The AI should **not directly control the database**.

Instead:

```text
AI Service
     |
     v
Backend API
     |
     v
PostgreSQL
```

This keeps responsibilities separate.

### AI Service

Responsible for:

* Video processing
* Vehicle detection
* Plate detection
* OCR
* Tracking

### Backend

Responsible for:

* Business rules
* Authentication
* Database operations
* Vehicle visits
* Reports
* User permissions

### Frontend

Responsible for:

* Dashboard
* Tables
* Forms
* Live status
* User interaction

---

# 18. Proposed Technology Stack

## AI Service

```text
Python
OpenCV
YOLO
PaddleOCR
ByteTrack
FastAPI
```

## Backend

```text
Node.js
Express.js
PostgreSQL
JWT
```

## Frontend

```text
Angular
TypeScript
SCSS
```

## Infrastructure

```text
Docker
Linux
Nginx
AWS
```

We can change individual technologies later if testing shows a better option.

---

# 19. Database – Initial Design

We should keep the first database simple.

## Users

```text
users
----------------
id
name
email
password
role
created_at
```

## Vehicles

This stores unique vehicle numbers.

```text
vehicles
----------------
id
plate_number
created_at
```

Example:

```text
1 | AP39AB1234
2 | AP05XY5678
```

## Vehicle Visits

This represents a vehicle entering and leaving the site.

```text
vehicle_visits
----------------
id
vehicle_id
entry_time
exit_time
entry_image
exit_image
status
created_at
```

Example:

```text
1
vehicle_id: 1
entry_time: 09:42
exit_time: 10:25
status: COMPLETED
```

---

# 20. Why We Separate Vehicles and Visits

The same vehicle can visit the site many times.

For example:

```text
Vehicle
AP39AB1234
```

could have:

```text
Visit 1 → 06-Aug → 09:20 → 10:10

Visit 2 → 07-Aug → 11:15 → 12:02

Visit 3 → 08-Aug → 09:42 → 10:25
```

Therefore:

```text
vehicles
```

stores the vehicle itself.

And:

```text
vehicle_visits
```

stores each visit.

This will make searching and reporting easier.

---

# 21. Camera Management

The system should know which camera produced an event.

Initial camera setup:

```text
Camera 1
Type: ENTRY

Camera 2
Type: EXIT
```

Later we can support:

```text
Camera 3
Camera 4
Camera 5
```

Each camera can have:

```text
id
name
location
type
stream_url
status
```

---

# 22. Images

When the AI detects a vehicle, we should save an image/snapshot as evidence.

For example:

```text
Entry
AP39AB1234
09:42
[vehicle image]
```

And:

```text
Exit
AP39AB1234
10:25
[vehicle image]
```

Initially, images can be stored locally during development.

Later, we can move them to cloud storage such as AWS S3.

---

# 23. Error Handling

AI will not always be correct.

For example, the camera may produce:

```text
AP39AB1234
```

but OCR may read:

```text
AP39A8134
```

Therefore the application should allow an operator to correct the vehicle number.

Example:

```text
Detected:
AP39A8134

[ Edit ]

Correct:
AP39AB1234

[ Save ]
```

This is important for making the system practical.

---

# 24. Confidence

Every AI detection should have a confidence score.

Example:

```text
AP39AB1234
Confidence: 97%
```

If confidence is high:

```text
97%
```

the system can accept it automatically.

If confidence is low:

```text
61%
```

the system can ask the operator to verify it.

The exact confidence threshold will **not be decided now**. We will determine it after testing real CCTV footage.

---

# 25. Security Requirements

The application should have basic security from the beginning.

* Passwords must be securely hashed.
* Users must authenticate before accessing protected APIs.
* Manager/operator permissions should be separated.
* Database credentials must not be stored in source code.
* CCTV credentials must not be exposed in frontend code.
* API inputs must be validated.
* Important actions should be logged.

---

# 26. Performance Requirements

The system should work continuously during operating hours.

The AI should process the CCTV stream without creating unnecessary duplicate records.

The application should remain usable while AI processing is happening.

We will measure actual performance during testing rather than setting unrealistic numbers before we know the hardware and camera specifications.

---

# 27. Reliability Requirements

The system should handle common problems such as:

```text
CCTV disconnected
Internet disconnected
AI service stopped
Backend stopped
Database unavailable
```

For example, if a camera disconnects, the dashboard should show:

```text
Entry Camera
OFFLINE
```

instead of silently failing.

---

# 28. Cloud Architecture

For the first real deployment, our preferred architecture is:

```text
              CCTV Cameras
                   |
                   v
          Local AI Computer
                   |
          +--------+--------+
          |                 |
      AI Service       Local Buffer
          |
          v
       Backend
          |
          v
       AWS Cloud
       /       \
      /         \
 PostgreSQL     S3
    RDS        Images
      |
      v
   Dashboard
```

The reason for keeping AI processing locally is that continuously sending CCTV video to the cloud can increase bandwidth and cloud-processing costs.

The exact deployment architecture will be finalized after we test the actual CCTV cameras and internet connection.

---

# 29. Development Environment

During development we will initially use:

```text
Developer Laptop
       |
       +--- Angular
       |
       +--- Node.js
       |
       +--- Python AI
       |
       +--- PostgreSQL
```

We don't need AWS on Day 1.

Cloud deployment comes after the local system works.

---

# 30. Development Phases

## Phase 1 — Documentation & Design

* Requirements
* Workflow
* Database design
* Architecture
* UI planning

**Result:** We know exactly what we are building.

---

## Phase 2 — Backend Foundation

Build:

* Authentication
* Users
* Vehicles
* Vehicle visits
* Camera APIs

**Result:** Working backend with PostgreSQL.

---

## Phase 3 — Frontend Foundation

Build:

* Login
* Dashboard
* Vehicle history
* Live status
* Record verification

**Result:** Working web application using dummy data.

---

## Phase 4 — AI Prototype

Build:

* Video reading
* Vehicle detection
* Number plate detection
* OCR
* Tracking

**Result:** AI can process recorded CCTV footage.

---

## Phase 5 — AI + Backend Integration

Connect:

```text
AI → Backend → Database
```

**Result:** Real vehicle events are stored automatically.

---

## Phase 6 — Real CCTV Testing

Connect the application to an actual camera.

Test:

* Daytime
* Nighttime
* Multiple vehicles
* Dirty plates
* Different vehicle angles
* Fast-moving vehicles
* Poor lighting
* Duplicate detections

**Result:** Understand real-world accuracy.

---

## Phase 7 — Production Deployment

Add:

* Docker
* Server deployment
* Cloud storage
* Database backup
* Monitoring
* Logging
* Security

**Result:** Deployable product.

---

# 31. MVP Definition

Our MVP is successful when this works:

```text
                    LORRY
                      |
                      v
                  CCTV CAMERA
                      |
                      v
                     AI
                      |
                      v
                 AP39AB1234
                      |
                      v
               ENTRY RECORDED
                      |
                      |
                 Lorry leaves
                      |
                      v
                     AI
                      |
                      v
                 AP39AB1234
                      |
                      v
                EXIT RECORDED
                      |
                      v
                  DATABASE
                      |
                      v
                  DASHBOARD
```

A manager should then be able to open the application and see:

```text
Vehicle       Entry       Exit       Status

AP39AB1234    09:42       10:25      Completed
AP05XY5678    10:10       --         Inside
```

If we can reliably achieve this with real CCTV footage, **Version 1 is successful.**

---

# 32. Success Criteria

We will consider the first version successful if:

1. CCTV footage can be processed.
2. Vehicles can be detected.
3. Number plates can be read.
4. Entry events can be created.
5. Exit events can be matched with previous entries.
6. Duplicate detections are controlled.
7. Records are stored in PostgreSQL.
8. Managers can search vehicle history.
9. Operators can correct incorrect AI results.
10. The system can run continuously without constant manual intervention.

We will measure actual AI accuracy using real site footage before claiming a specific accuracy percentage.

---

# 33. Future Version Ideas

After Version 1 is stable, possible additions include:

```text
Version 2
-----------
Weighbridge integration
Driver information
Automatic gate pass
Better reporting
Notifications


Version 3
-----------
Multiple locations
Mobile application
GPS
Advanced analytics
Cloud-based multi-tenant system
```

These are intentionally outside the current scope.

---

# 34. Important Development Principle

We will follow this rule throughout the project:

> **Build the simplest working version first.**

We will not try to build the complete commercial product immediately.

Our development path is:

```text
Simple Prototype
       ↓
Working MVP
       ↓
Real CCTV Testing
       ↓
Improve Accuracy
       ↓
Production System
       ↓
Commercial Product
```

---

# 35. Current Project Status

**Requirements:** Partially defined
**Architecture:** Initial design completed
**Database:** Initial design completed
**AI:** Not started
**Backend:** Not started
**Frontend:** Not started
**Deployment:** Not started

### Next step

Before writing production code, we should now finalize the **real-world CCTV workflow and camera setup**.

The next information we need from the site is:

* How many CCTV cameras are available?
* Which camera watches the entry?
* Which camera watches the exit?
* Are entry and exit physically separate?
* Does the CCTV provide an RTSP stream?
* Approximately how far is the camera from the lorry?
* Is the number plate clearly visible?
* Is there a separate day/night camera or IR/night vision?
* Approximately how many lorries pass through per day?

Once these are known, we can finalize the architecture instead of designing around assumptions.
