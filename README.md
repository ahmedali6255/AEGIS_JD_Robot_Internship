# AEGIS-JD — Object Detection & PID Tracking Module

This repository contains my individual contribution to **AEGIS-JD (Autonomous Engagement & Guardian Intelligence System)** — a unified, multi-modal control system built for the EZ-Robot JD Humanoid, developed during my robotics internship as part of a 5-member team.

The full AEGIS-JD platform integrates object detection, gesture/emotion/pose control, speech control, conversational AI, facial recognition, and a security gate. **This repo covers only the module I built: real-time object detection, AI-based target reasoning, and PID-based servo tracking.**

## What this module does

Given a spoken/typed command like *"find my bottle"*, the robot:

1. Uses **Gemini API** to figure out which object the user is asking for
2. Scans its surroundings (pan/tilt sweep) while running **YOLOv8** object detection on each captured frame
3. Once the object is found, uses a **PID controller** to smoothly center the camera on it — no jerky or overshooting movement
4. Uses **Gemini** again to describe *where* the object is in natural, spoken language
5. Speaks the result back through the robot

Python (this module) communicates with the robot hardware via **file-based IPC** with `ARC_SCRIPT.py`, which runs inside **Synthiam ARC** and directly drives the JD's head (pan/tilt), arm, and RGB LED servos/effects.

## Files

| File | Purpose |
|---|---|
| `object_finder.py` | Main orchestrator — ties detection, reasoning, and PID tracking together |
| `yolo_detector.py` | Runs YOLOv8 on captured frames, returns detected objects + positions |
| `pid_controller.py` | Reusable PID controller for smooth servo tracking |
| `vision.py` | Gemini API integration — target extraction from natural language + location description |
| `coco_classes.py` | List of the 80 object classes YOLOv8 can recognize |
| `ARC_SCRIPT.py` | Runs inside Synthiam ARC — reads commands from shared files and drives the robot's pan/tilt head servos, arm/pointing servos, and RGB LED expressions |
| `requirements.txt` | Python dependencies |

> **Note:** In the full AEGIS-JD system, `object_finder.py` also calls into a face-recognition module built by another teammate (for "who is this?" style commands). That module is **not included here** — it belongs to a different contributor. To run this repo standalone, that part of the code path is inactive/removed.

## Tech Stack

- **Object Detection:** YOLOv8 (Ultralytics, local inference, no internet required)
- **AI Reasoning:** Google Gemini API (`google-genai` SDK, `gemini-flash-latest`)
- **Control:** Custom PID controller (Kp/Ki/Kd tuned for smooth servo tracking)
- **Hardware Bridge:** File-based IPC between Python and Synthiam ARC
- **Language:** Python

## Setup

1. Clone this repo and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Copy `.env.example` to `.env` and add your own Gemini API keys:
   ```
   GEMINI_API_KEY_SEARCH=your_key_here
   GEMINI_API_KEY_LOCATION=your_key_here
   ```
3. The YOLOv8 model (`yolov8n.pt`) auto-downloads on first run — no manual download needed.
4. Paste `ARC_SCRIPT.py` into Synthiam ARC's script editor and run it (Alt-R) — it listens for commands via shared text files (`trigger.txt`, `servo.txt`, `arm.txt`, `led.txt`, `speak.txt`) and drives the physical robot accordingly. Without a connected JD robot, `object_finder.py` won't have hardware to drive, but `yolo_detector.py` and `pid_controller.py` can be tested independently.

## About the full project

AEGIS-JD was built by a 5-member team as part of a robotics internship:
- **Object Detection & PID Servo Control** — this repo
- **Gesture, Emotion, Pose & Speech Control, System Integration** — teammate contribution
- **Facial Recognition & Security Gate** — teammate contribution
- **Conversational AI ("Ask JD")** — teammate contribution

This repo reflects only my individual work on the project.
