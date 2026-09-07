# 🛡️ Multi-Layer Anti-Drone Defense System

<p align="center">
  <strong>AI-Powered Multi-Sensor Counter-UAS Platform</strong>
</p>

<p align="center">
  A real-time aerial threat detection, tracking, prioritization, and visual servoing system powered by dual-model YOLOv8, RGB + thermal sensor fusion, multi-object tracking, NMSE threat scoring, PID control, and optional reinforcement learning.
</p>

<p align="center">

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge\&logo=python)
![YOLOv8](https://img.shields.io/badge/YOLOv8-Computer%20Vision-red?style=for-the-badge)
![OpenCV](https://img.shields.io/badge/OpenCV-Sensor%20Fusion-green?style=for-the-badge\&logo=opencv)
![Arduino](https://img.shields.io/badge/Arduino-Hardware-00979D?style=for-the-badge\&logo=arduino)
![PyTorch](https://img.shields.io/badge/PyTorch-Deep%20Learning-ee4c2c?style=for-the-badge\&logo=pytorch)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

</p>

---

## 🚀 Overview

**Multi-Layer Anti-Drone Defense System** is a real-time **Counter-UAS (C-UAS)** research and engineering platform designed to detect, classify, track, prioritize, and visually follow aerial objects using multiple sensing modalities.

Unlike traditional single-camera systems, this project combines:

* 📷 **RGB Computer Vision**
* 🌡️ **Thermal Computer Vision**
* 🎯 **Multi-Object Tracking**
* 📡 **Radar/Distance Sensing**
* 🧠 **Threat Prioritization**
* 🎛️ **PID Visual Servoing**
* 🤖 **Optional Reinforcement Learning**
* 🔌 **Arduino-based low-level actuation**

The architecture intentionally separates **high-level AI computation** from **low-level hardware control**:

> **Laptop = Intelligence & Decision Making**
> **Arduino = Real-Time Hardware Control**

No Raspberry Pi is required.

---

# ⭐ What Makes This Project Different?

The system is designed as a **layered perception-to-control pipeline**, rather than simply running an object detector.

```text
              ┌───────────────────────┐
              │      RGB Camera       │
              └───────────┬───────────┘
                          │
                    YOLOv8 RGB
                          │
                          ▼
                    RGB Detections
                          │
                          │
                          ├───────────────┐
                          │               │
                          ▼               ▼
              ┌────────────────┐  ┌─────────────────┐
              │ Sensor Fusion  │  │ Thermal Camera  │
              │ IoU + Transform│  │    YOLOv8       │
              └───────┬────────┘  └────────┬────────┘
                      │                    │
                      └─────────┬──────────┘
                                ▼
                     ┌────────────────────┐
                     │ Multi-Object       │
                     │ Tracker            │
                     └─────────┬──────────┘
                               ▼
                     ┌────────────────────┐
                     │ NMSE Threat        │
                     │ Prioritization     │
                     └─────────┬──────────┘
                               ▼
                     ┌────────────────────┐
                     │ PID Visual         │
                     │ Servoing            │
                     │ + Optional RL      │
                     └─────────┬──────────┘
                               ▼
                         Serial / USB
                               │
                               ▼
                     ┌────────────────────┐
                     │     Arduino        │
                     │                    │
                     │ Pan / Tilt         │
                     │ Radar Sweep        │
                     │ Actuators          │
                     └────────────────────┘
```

The result is a complete **Perception → Fusion → Tracking → Decision → Control** pipeline.

---

# 🧠 System Layers

| Layer              | Component             | Purpose                                  |
| ------------------ | --------------------- | ---------------------------------------- |
| **Layer 1**        | Dual YOLOv8           | Detect aerial objects from RGB + thermal |
| **Layer 2**        | Sensor Fusion         | Align and merge observations             |
| **Layer 3**        | Multi-Object Tracking | Maintain persistent target identities    |
| **Layer 4**        | NMSE Prioritization   | Rank threats based on multiple factors   |
| **Layer 5**        | PID Servoing          | Keep the selected target centered        |
| **Layer 6**        | RL Tuning             | Adapt controller aggressiveness online   |
| **Hardware Layer** | Arduino               | Execute real-time actuator commands      |

---

# 🔥 Key Features

### 🎥 Dual-Sensor Perception

Both sensors operate concurrently.

**RGB YOLOv8**

Detects:

* 🐦 Bird
* 🚁 Drone
* ✈️ Airplane
* 🚁 Helicopter

**Thermal YOLOv8**

Detects thermal aerial targets and maps them to the canonical:

```text
Drone
```

There is **no day/night mode switching**.

Both streams are processed continuously.

---

### 🌡️ RGB + Thermal Sensor Fusion

Thermal detections are transformed into RGB coordinates using a calibrated similarity transform.

The system uses:

* Rotation
* Translation
* Scale
* IoU-based matching
* Weighted bounding-box fusion
* Cross-sensor confidence boosting

Conceptually:

```text
RGB Detection
      +
Thermal Detection
      ↓
Geometric Alignment
      ↓
IoU Matching
      ↓
Weighted Fusion
      ↓
High-Confidence Target
```

Unmatched detections are preserved instead of being discarded.

This allows the system to remain robust when one sensor experiences:

* Glare
* Low contrast
* Thermal ambiguity
* Occlusion
* Poor visibility

---

# 🎯 Threat Prioritization

Detection alone isn't enough.

When multiple aerial objects are present, the system needs to answer:

> **Which target should receive attention first?**

The project introduces an **NMSE-based threat prioritization engine**.

Each active track is represented using normalized threat features:

```text
Distance
Closing Speed
Detection Confidence
Bounding Box Size
```

The ideal threat vector is:

```text
[1, 1, 1, 1]
```

The system computes:

```text
score = 1 − Σ wᵢ · (1 − fᵢ)²
```

Where:

* `fᵢ` = normalized threat feature
* `wᵢ` = configurable feature weight

Higher score:

```text
        ↓
Higher Threat Priority
        ↓
Selected Target
```

This creates a deterministic and auditable target-selection mechanism.

---

# 🎛️ PID Visual Servoing

Once a target is selected, the system controls the pan-tilt gimbal using a full PID controller.

For each axis:

```text
Error = Target Position − Image Center
```

The controller computes:

```text
PID = Kp × Error
    + Ki × Integral(Error)
    + Kd × Derivative(Error)
```

### P — Proportional

Responds to the current tracking error.

### I — Integral

Compensates for persistent offset and drift.

### D — Derivative

Reduces overshoot and improves stability.

### Anti-Windup

Integral accumulation is clamped to prevent excessive actuator commands.

---

# 🤖 Optional Reinforcement Learning Layer

The project also includes an optional lightweight **Q-Learning PID gain tuner**.

The RL agent observes tracking behavior and can adaptively adjust controller aggressiveness.

Important architectural constraint:

> **RL does not choose the target.**

Target selection remains deterministic:

```text
Sensors
   ↓
Detection
   ↓
Fusion
   ↓
Tracking
   ↓
NMSE Priority
   ↓
TARGET SELECTION
```

Only after the target is selected:

```text
Selected Target
       ↓
PID Controller
       ↓
RL Gain Tuner
       ↓
Gimbal Command
```

This separation makes the system easier to analyze, debug, and audit.

---

# 📡 Radar + Camera Fusion

The camera gimbal and radar sweep are treated as separate subsystems.

### Current Prototype

Ultrasonic distance sensor mounted on a sweeping servo.

### Future Upgrade

The architecture is designed to support a mmWave radar such as:

```text
IWR1843
```

The sensing interface is abstracted so that the distance-reading implementation can be replaced without redesigning the entire perception pipeline.

---

# 🏗️ System Architecture

```text
                         ┌─────────────────┐
                         │   RGB Camera    │
                         └────────┬────────┘
                                  │
                                  ▼
                           ┌─────────────┐
                           │ YOLOv8 RGB  │
                           │   4 Classes │
                           └──────┬──────┘
                                  │
                                  │
                                  ▼
                           ┌─────────────┐
                           │             │
                           │   SENSOR    │
                           │   FUSION    │
                           │             │
                           └──────┬──────┘
                                  ▲
                                  │
                           ┌──────┴──────┐
                           │ YOLOv8      │
                           │ Thermal     │
                           │ 2 Classes   │
                           └──────▲──────┘
                                  │
                         ┌────────┴────────┐
                         │ Thermal Camera │
                         └─────────────────┘

                                  │
                                  ▼
                        ┌───────────────────┐
                        │ Multi-Object      │
                        │ Tracker           │
                        └─────────┬─────────┘
                                  │
                                  ▼
                        ┌───────────────────┐
                        │ NMSE Threat       │
                        │ Prioritization    │
                        └─────────┬─────────┘
                                  │
                                  ▼
                        ┌───────────────────┐
                        │ PID Visual        │
                        │ Servoing          │
                        └─────────┬─────────┘
                                  │
                         Optional RL Tuning
                                  │
                                  ▼
                           Serial / USB
                                  │
                                  ▼
                         ┌─────────────────┐
                         │    Arduino     │
                         ├─────────────────┤
                         │ Pan Servo       │
                         │ Tilt Servo      │
                         │ Radar Servo     │
                         │ Sensors         │
                         └─────────────────┘
```

---

# 📁 Project Structure

```text
anti_drone_system/
│
├── firmware/
│   └── anti_drone_controller/
│       └── anti_drone_controller.ino
│
├── anti_drone_pipeline/
│   ├── config.py
│   ├── serial_link.py
│   ├── detection_source.py
│   ├── fusion.py
│   ├── tracker.py
│   ├── priority.py
│   ├── pantilt_controller.py
│   ├── rl_tuner.py
│   ├── logger.py
│   └── main.py
│
├── models/
│   ├── rgb_multiclass_yolov8s_best.pt
│   └── thermal_yolov8_best.pt
│
├── tools/
│   └── calibrate_thermal_offset.py
│
├── assets/
│   ├── training_results.png
│   ├── recall_confidence_curve.png
│   └── confusion_matrix.png
│
├── logs/
│
├── requirements.txt
└── README.md
```

---

# 🔬 Detection Pipeline

## RGB Detection

The RGB model is a YOLOv8s detector trained for four aerial-object classes:

```text
Bird
Drone
Airplane
Helicopter
```

Only the `Drone` class is considered a direct threat.

The remaining classes remain available for situational awareness and classification context.

---

## Thermal Detection

A dedicated thermal YOLOv8 model processes the thermal stream independently.

Thermal detections are converted into the system's canonical target representation:

```text
Thermal Target → Drone
```

This allows both sensing modalities to communicate through a common representation.

---

# 🔄 Sensor Calibration

Before running the complete system, the RGB and thermal cameras must be geometrically aligned.

Run:

```bash
python -m tools.calibrate_thermal_offset
```

The calibration process estimates a similarity transformation using:

```python
cv2.estimateAffinePartial2D
```

The resulting transformation accounts for:

```text
Translation
Rotation
Scale
```

Thermal detections are then projected into RGB image coordinates.

---

# 🎯 Multi-Object Tracking

The system maintains persistent target identities across frames.

A track contains information such as:

```text
Track ID
Bounding Box
Class
Confidence
Position
Velocity
Age
Distance
Threat Score
```

The current implementation uses centroid/IoU-based association.

The tracker architecture is intentionally modular and can later be replaced with:

```text
DeepSORT
ByteTrack
BoT-SORT
```

without redesigning the rest of the system.

---

# 📊 Model Performance

## RGB YOLOv8s

Training configuration:

```text
Model: YOLOv8s
Classes: 4
Epochs: 100
```

### Results

| Metric    | Performance |
| --------- | ----------: |
| Precision |       ~0.96 |
| Recall    |       ~0.97 |
| mAP@50    |       ~0.98 |
| mAP@50–95 |       ~0.73 |

### Training Curves

<p align="center">
  <img src="assets/training_results.png" alt="YOLOv8 Training Results" width="900">
</p>

---

## Recall–Confidence Analysis

The detector reaches approximately:

```text
99% recall
```

at a confidence threshold close to:

```text
0.000
```

<p align="center">
  <img src="assets/recall_confidence_curve.png" alt="Recall Confidence Curve" width="700">
</p>

---

## Confusion Matrix

<p align="center">
  <img src="assets/confusion_matrix.png" alt="Normalized Confusion Matrix" width="650">
</p>

> The thermal detector is trained independently on a dedicated thermal drone dataset. Its model-specific evaluation metrics are maintained separately.

---

# 🎬 System Demonstration

## Concept Demo

<p align="center">

[▶️ Watch the System Demonstration](https://github.com/user-attachments/assets/fdaeaf48-755c-4317-a3a3-448422952287)

</p>

The demonstration showcases the core concept of the perception, tracking, prioritization, and control pipeline.

---

# 💻 Getting Started

## 1. Clone the Repository

```bash
git clone https://github.com/Ahmd00z/Multi-Layer-Anti-Drone-Defense-System.git

cd Multi-Layer-Anti-Drone-Defense-System
```

---

## 2. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 3. Add Model Weights

Place the trained models inside:

```text
models/
```

Expected files:

```text
models/
├── rgb_multiclass_yolov8s_best.pt
└── thermal_yolov8_best.pt
```

---

## 4. Configure the System

Open:

```text
anti_drone_pipeline/config.py
```

Configure:

```text
RGB camera index
Thermal camera index
Arduino serial port
Detection thresholds
Fusion weights
Priority weights
PID gains
Tracker parameters
```

---

## 5. Flash Arduino Firmware

Open:

```text
firmware/anti_drone_controller/anti_drone_controller.ino
```

using the Arduino IDE.

Upload the firmware to the Arduino board.

The firmware uses:

```cpp
Servo.h
```

and does not require external Arduino libraries.

---

## 6. Calibrate Cameras

Run:

```bash
python -m tools.calibrate_thermal_offset
```

This generates the transformation required for RGB/thermal alignment.

---

## 7. Start the System

```bash
python -m anti_drone_pipeline.main
```

Press:

```text
q
```

to exit safely.

The shutdown routine is designed to restore the hardware to a safe state before terminating.

---

# 🔌 Hardware Architecture

```text
                    USB
                     │
                     ▼
              ┌─────────────┐
              │   Laptop    │
              │             │
              │ YOLOv8      │
              │ Fusion      │
              │ Tracking    │
              │ Priority    │
              │ PID / RL    │
              └──────┬──────┘
                     │
                  Serial
                     │
                     ▼
              ┌─────────────┐
              │   Arduino   │
              └──────┬──────┘
                     │
        ┌────────────┼─────────────┐
        ▼            ▼             ▼
    Pan Servo     Tilt Servo   Radar Servo
        │            │             │
        └────────────┴─────────────┘
                     │
               Sensor Platform
```

### Hardware Components

* Laptop
* RGB Camera
* Thermal Camera
* Arduino
* Pan Servo
* Tilt Servo
* Radar Sweep Servo
* Ultrasonic Distance Sensor

### Future Radar

The ultrasonic sensor can be replaced with a mmWave radar such as:

```text
Texas Instruments IWR1843
```

---

# 🧩 Software Architecture

The codebase is intentionally modular.

### `config.py`

Centralized configuration and tunable parameters.

### `detection_source.py`

Handles:

```text
RGB capture
Thermal capture
YOLO inference
Detection normalization
```

### `fusion.py`

Responsible for:

```text
Thermal → RGB transformation
IoU matching
Box fusion
Confidence boosting
```

### `tracker.py`

Maintains persistent target tracks.

### `priority.py`

Computes normalized threat scores and maintains the priority queue.

### `pantilt_controller.py`

Implements:

```text
PID control
Anti-windup
Gimbal commands
```

### `rl_tuner.py`

Optional Q-learning controller-gain adaptation.

### `serial_link.py`

Provides threaded communication between Python and Arduino.

### `logger.py`

Records events for offline analysis.

### `main.py`

Coordinates the complete state machine.

---

# 📝 Event Logging

System events are written to CSV for later analysis.

Example information:

```text
Timestamp
Track ID
Class
Confidence
Distance
Velocity
Threat Score
Selected Target
Pan Error
Tilt Error
PID Output
```

This makes experiments reproducible and enables post-run analysis.

---

# ⚙️ Configuration

All major parameters are centralized in:

```text
anti_drone_pipeline/config.py
```

This includes:

```text
Detection thresholds
IoU thresholds
Fusion weights
NMSE weights
PID gains
Integral limits
Servo limits
Serial configuration
Camera configuration
RL parameters
```

The goal is to make experimentation possible without modifying the core pipeline.

---

# 🛣️ Roadmap

### Tracking

* [ ] Integrate DeepSORT
* [ ] Add appearance-based Re-ID
* [ ] Improve swarm tracking
* [ ] Handle long-term occlusions

### Radar

* [ ] Replace ultrasonic sensor with mmWave radar
* [ ] Integrate radar range/velocity measurements
* [ ] Implement camera-radar association
* [ ] Improve 3D target localization

### Sensor Fusion

* [ ] Full intrinsic/extrinsic camera calibration
* [ ] Improve temporal synchronization
* [ ] Confidence-aware probabilistic fusion
* [ ] Radar + RGB + thermal fusion

### Control

* [ ] Automated PID parameter optimization
* [ ] Advanced trajectory prediction
* [ ] Model-based predictive control
* [ ] Extended RL controller training

### Machine Learning

* [ ] Expand aerial-object dataset
* [ ] Improve thermal classification
* [ ] Domain adaptation between RGB and thermal
* [ ] Quantization / TensorRT optimization
* [ ] Edge inference benchmarking

### Deployment

* [ ] GPU acceleration
* [ ] Docker deployment
* [ ] Real-time performance profiling
* [ ] Hardware-in-the-loop testing
* [ ] Multi-camera support

---

# 📈 Future System

The long-term architecture is designed to evolve from:

```text
RGB + Thermal
      ↓
YOLOv8
      ↓
Tracking
      ↓
NMSE
      ↓
PID
```

into:

```text
       RGB Camera
            │
            ▼
        YOLOv8
            │
            │
Thermal ──► Fusion ◄── Radar
            │
            ▼
       Multi-Object
          Tracking
            │
            ▼
    Threat Assessment
            │
            ▼
      Target Ranking
            │
            ▼
   Predictive Controller
            │
            ▼
     Gimbal / Sensors
```

This creates a scalable architecture for future research in:

* Multi-modal perception
* Autonomous tracking
* Sensor fusion
* Robotics
* Computer vision
* Intelligent control
* Reinforcement learning

---

# 🧪 Research & Engineering Concepts

This project combines several important AI and robotics concepts in one system:

```text
Computer Vision
      +
Deep Learning
      +
Multi-Sensor Fusion
      +
Object Tracking
      +
State Estimation
      +
Threat Assessment
      +
Control Theory
      +
Reinforcement Learning
      +
Embedded Systems
```

Rather than treating these technologies independently, the project connects them into one end-to-end pipeline.

---

# 🏆 Technical Highlights

### Deep Learning

* YOLOv8
* Transfer learning
* Multi-class object detection
* Thermal object detection

### Computer Vision

* OpenCV
* Bounding-box processing
* IoU matching
* Geometric transformations
* Image-space visual servoing

### Sensor Fusion

* RGB + Thermal fusion
* Similarity transformation
* Confidence-weighted fusion

### Tracking

* Multi-object tracking
* Centroid association
* IoU-based association
* Persistent track IDs

### Decision Making

* NMSE threat scoring
* Weighted feature normalization
* Priority queue

### Control

* PID controller
* Integral anti-windup
* Pan-tilt control

### Reinforcement Learning

* Q-Learning
* Adaptive gain tuning
* Online controller adjustment

### Embedded Systems

* Arduino
* Servo control
* Serial communication
* Sensor integration

---

# 🔐 Safety & System Boundaries

This repository is intended as an **educational and research prototype for perception, tracking, sensor fusion, and robotic control**.

The architecture separates target detection and tracking from any physical intervention mechanism, allowing the perception and control stack to be evaluated independently in simulation or with benign test targets.

For experimentation, use controlled environments and non-harmful test setups.

---

# 📜 License

This project is released under the **MIT License**.

See:

```text
LICENSE
```

for more information.

---

# 👨‍💻 Author

**Ahmed Maged**

Machine Learning Engineer / AI Developer

<p align="center">

⭐ If you found this project interesting, consider giving the repository a star!

</p>

---

<p align="center">

### 🛡️ Detect. Fuse. Track. Prioritize. Control.

<strong>Multi-Layer Anti-Drone Defense System</strong>

</p>
