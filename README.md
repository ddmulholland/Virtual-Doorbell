# Virtual-Doorbell
Virtual doorbell for factory usage.

## Overview
A Virtual Doorbell system provides a safe, reliable way for visitors to alert floor personnel to a presence at an entrance. This proposed solution would include a **digital access console** at the entryway, and a **visual notification display** inside the shop to alert staff to a visitor's presence.

## Problem
Currently floor personnel deal with a **loud environment** that necessitatese the use of noise-reducing headphones or other sensory bockers that avoid harm. When visitors arrive at the shop, they are forced to knock on the door and hope someone is aware enough to allow entry. Therefore, a solution is needed that provides **visual notification** to the specified floor personnell based on **user input at the entry** to gain access.

## Solution
Using a touch display (HMI or Rasp. Pi screen), a 'Ring' button UI + status indicator, logic processor, and visual alert display, we can easily implement a scalable system for usage on the floor.

---

## System Flow
#### 1. Visitor arrives at the door
  - Observe **HMI screen** mounted at the entryway with clear prompting
    - "Press for Entry"
  - Tap the touchscreen

#### 2. Door Raspberry Pi detects the request
  - Small app/service on the Pi listens for:
    - Touch events from the digital UI
    - GPIO signal changes (if using physical button)
  - When triggered, Pi will:
    - Debounce the press (ignore rapid repeats)
    - Stamp with **time**, **door id** (if multiple locations, "Main Entry")

#### 3. Event is logged and turned into an Alert
  - Pi runs lightweight backend (Flask)
  - Record a "doorbell pressed" event:
    - Saved into a local **SQLite database** or **log file** for event history.
    - Marked as **active alert**

#### 4. Pi broadcasts the alert to the shop
  - On the shop LAN (existing or new) ethernet from Pi, Pi will:
    - Push an **alert message** to the display nodes using **MQTT topics**
  - Message will include: alert id, door id, timestamp, status="ACTIVE"
  
#### 5. Display nodes show the alert
  - Each **display node** will include a Pi and:
    - Monitor: This could be used if 2 way communication was desired, i.e. "Acknowledge Request" prompt for the person answering the door, "Request Acknowledged" for the person waiting.
    - Industrial Alert Stack Light: Straightforward, slightly less implementation effort, no 2-way communication. Can be in parallel with the monitor.
  - Monitor displays the **alert messasge** and the stack light is changed to on/specific color/flashing (staff input required)
  - (Assuming monitor setup for below process)

#### 6. Staff acknowledges the alert
  - From the display node user hits "Acknowledge" on the screen
  - Sends a message back to the Pi backend with inerpretable **alert id** and **ack source**
  
#### 7. Pi updates status and re-broadcasts
  - Backend marks the alert as **ACKED** in the database
  - Pushes out an updated alert state to display nodes
    - All endpoint alert nodes change to "Alert Acknowledged by Location X - [HH:MM]".
    - Stack light changes from **ON** state to **OFF** state
  
#### 8. Alert is cleared
  - After a timeout or when staff manually clear:
    - backend status set to "CLEARED" for that alert
    - display returns to idle state
  
#### Additional Considerations:
  - Timeout for non-acknowledgement (nobody home) - ask metal fab how long alert should be active for
  - Specify what is wanted - acknowledgement system sounds good on paper but confirm that additional scope is desired.
  - Make sure safety is not interfered with - Use specific colors for signal not already present on floor, do not interfere with current safety warning signs.
  - Confirm Pi can be used (wifi / bluetooth not used in this case but capability may make this solution nonconforming - can consider industrial PLCs if needed.)

---

## Implementaion plan
#### 1. Base Platform Setup
- Install OS on the Pi
- Connect HMI screen to the Pi via HDMI (confirm necessary connection types and permissible data transmission)
- Configure:
    - Auto-Login
    - Screen resolution
    - Kiosk mode browser
- Goal: Allow Pi boot straight to a test screen
#### 2. Basic Door UI + Input Handling
- Build simple UI:
  - one big button "Press to Request Entry"
- Wire up touch input to app
- Goal: Verify we can detect a press and print something
  - DOORBELL PRESSED AT [HH:MM:SS] from MAIN_ENTRY
#### 3. Backend Event Logic + Local Logging
- On the doorbell Pi add small backend (Flask, Node, whatever team stanadard is)
- Implement
  - Function to create **alert object** {ID, Time, Status}
  - Logging to preferred logging of team (SQLite, simple JSON file)
- Goal: Press doorbell -> new alert stored, See list of active alerts via log
#### 4. Simple Internal Display UI
- For testing on the system, implement a simulated display node on the Door Pi
  - Open different browser tab
  - build minimal webpage that:
    - Connects to backend (HTTP)
    - shows "No alerts" vs. "Visitor at door"
- Goal: Prove end-to-end flow with the single point
  - Press doorbell -> alert appears on display page
#### 5: Add Acknowledge/Clear logic
- On display page add an acknowledge button -> send request with alert id to the backend
- Backend will:
  - update alert status to ACKED/CLEARED
  - broadcast state to all connected clients
- Goal: Confirm that Press -> Appears; Ack -> status changes, UI Updates, alert clears
#### 6: Replicate Display nodes
- Scale to one additional display node (should be able to replicate process for as many nodes as needed)
  - Put anothe R Pi on the LAN
  - Point browser to same display URL
- Goal: Press doorbell -> all displays update; Acknowledge from one display -> all displays update
#### 7; Add I/O, Stack Lights
- Add extra Pi for driving Stack lights:
  - subsribe to the same alert state
  - Map states (example):
    - Flashing amber - new request
    - Solid amber - request acknowledged
    - Off - request fullfilled / no active request.







