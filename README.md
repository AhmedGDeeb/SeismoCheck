# 🏗️ SeismoCheck

**Drone-to-ETABS: Rapid Structural Assessment from Aerial Imagery**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Status: Early Prototype](https://img.shields.io/badge/Status-Early%20Prototype-orange.svg)]()

SeismoCheck is an open-source initiative to turn a drone flight into a structural safety verdict. We are building a pipeline that captures aerial imagery of a building, uses computer vision to reconstruct its structural geometry, generates a model in **ETABS** automatically, and runs a rapid seismic assessment — all in minutes.

---

## 🚨 The Problem

After an earthquake, engineers and civil defense teams must rapidly assess hundreds or thousands of buildings to decide which are safe to occupy, which need evacuation, and which require urgent intervention.

Today this is done by hand:
*   **Too Slow:** A single structural assessment takes 3–6 hours — and that's only if drawings exist and an engineer can safely enter the building.
*   **Subjective:** Rapid visual screening relies on individual judgment, producing inconsistent verdicts.
*   **Unscalable & Dangerous:** Post-disaster, many buildings are unsafe to enter. Teams are overwhelmed and cannot physically reach every structure in time.
*   **Data Gap:** For many existing buildings, structural drawings are missing, outdated, or never existed.

## 💡 The Solution

**SeismoCheck** replaces the manual survey with an autonomous drone-to-model pipeline:

1.  **Capture:** A drone flies a planned path around and above the building, capturing high-resolution overlapping imagery (and optionally LiDAR / photogrammetry data).
2.  **Reconstruct:** Computer vision and photogrammetry (Structure-from-Motion, point cloud processing) extract the building's footprint, column grid, story heights, and element dimensions.
3.  **Model:** Python scripts generate a simplified structural model in **ETABS** through its API, assigning materials, sections, and boundary conditions automatically.
4.  **Analyze:** ETABS runs an equivalent static or response spectrum analysis.
5.  **Evaluate:** The pipeline extracts **Story Drifts** and compares them against local code thresholds (e.g., the Syrian Code for Assessment of Existing Buildings), producing a rapid verdict: **Safe / Needs Audit / Unsafe**.
6.  **Report:** A one-page structural report is generated — no engineer required on site, no drawings required.

### ⚙️ Technical Pipeline

```text
[ Drone Flight ] 
      │  aerial images / video / LiDAR
      ▼
[ Computer Vision ]  ──►  point cloud / building geometry extraction
      │  (OpenCV, OpenSfM, YOLO, segmentation)
      ▼
[ Geometry Parser ]  ──►  grids, columns, beams, story heights
      │  (Python / ezdxf-style structural abstraction)
      ▼
[ ETABS API Builder ] ──►  model generation
      │  (comtypes + ETABSv1.dll)
      ▼
[ Analysis ]  ──►  equivalent static / response spectrum
      │
      ▼
[ Evaluator ]  ──►  story drift vs. code limits
      │
      ▼
[ Report ]  ──►  Safe / Needs Audit / Unsafe + PDF
```

---

Yes, it changes the file structure significantly. Because the core input is now **aerial imagery/video** rather than a **DXF file**, you need dedicated modules for drone flight planning, computer vision (photogrammetry/point cloud processing), and geometry abstraction. 

Here is the updated, highly modular GitHub repository structure tailored for the **Drone-to-ETABS** pipeline.

### 📁 Repository Structure

```text
SeismoCheck/
│
├── README.md                  <- The updated Drone-to-ETABS overview
├── LICENSE                    
├── requirements.txt           <- New dependencies (opencv-python, open3d, ultralytics, etc.)
├── .gitignore                 
│
├── docs/                      
│   ├── architecture.md        <- Diagram of Image -> CV -> ETABS pipeline
│   ├── drone_protocols.md     <- Flight patterns, altitudes, overlap requirements
│   ├── syrian_code_specs.md   <- Drift limits and safety thresholds
│   └── roadmap.md             
│
├── src/                       
│   ├── __init__.py
│   │
│   ├── drone/                 <- NEW: Drone & Flight related logic
│   │   ├── flight_planner.py  <- (Future) Scripts for automated flight paths (e.g., DJI SDK)
│   │   └── image_capture.py   <- Tools for extracting frames from drone video
│   │
│   ├── vision/                <- NEW: Computer Vision & Photogrammetry
│   │   ├── photogrammetry.py  <- Structure-from-Motion (SfM) logic to create point clouds
│   │   ├── point_cloud.py     <- Processing 3D data (using Open3D) to find edges/planes
│   │   ├── damage_detection.py<- YOLO/CNN models to detect cracks (Phase 5)
│   │   └── geometry_extract.py<- Converts point clouds into structural grids/columns
│   │
│   ├── etabs/                 <- NEW: Isolated ETABS API interaction
│   │   ├── etabs_builder.py   <- Builds the model from extracted geometry
│   │   ├── materials.py       <- Pre-defined concrete/steel properties
│   │   └── analyzer.py        <- Runs analysis and extracts Story Drifts
│   │
│   ├── core/                  
│   │   └── evaluator.py       <- Compares results to Syrian Code limits
│   │
│   └── report/                <- NEW: Output generation
│       └── pdf_generator.py   <- Generates the one-page safety verdict report
│
├── data/                      
│   ├── raw_imagery/           <- Sample drone photos/videos for testing CV
│   ├── point_clouds/          <- Generated 3D models (.ply, .pcd)
│   └── test_models/           <- Sample ETABS models for validation
│
├── tests/                     
│   ├── test_vision.py         <- Tests for CV modules
│   ├── test_etabs_api.py      <- Tests for ETABS generation
│   └── test_evaluator.py      <- Tests for code compliance math
│
└── main.py                    <- The main CLI script to run the pipeline
```

### 🛠️ What changed and why?

1. **`src/drone/`**: Since the project relies on aerial data, separating the drone-specific logic (like extracting frames from a video or planning flight paths) makes the code scalable for future drone integrations.
2. **`src/vision/`**: This is the new "brain" of the input phase. It handles the heavy lifting of turning 2D photos into 3D point clouds (using tools like OpenSfM), and then turning those 3D points into a structural grid.
3. **`src/etabs/`**: We separated the ETABS logic from the vision logic. This is crucial. It means a computer vision engineer can work on `vision/` while a structural engineer works on `etabs/` without their code clashing.
4. **`data/raw_imagery/`**: You need a place to store sample drone photos so contributors can test the computer vision models.
5. **`requirements.txt`**: Your dependencies will now be much larger. You will need:
   * `opencv-python` (image processing)
   * `open3d` (3D point cloud processing)
   * `ultralytics` (for YOLOv8 damage detection)
   * `openSfM` or `colmap` (for photogrammetry - *note: these are often run via command line, so you might call them via `subprocess` in Python*)
   * `comtypes` (for ETABS)


## 🌍 Why This Matters

In crisis zones, time is measured in lives — and drones can reach places engineers cannot.

*   **Municipalities & Civil Defense:** Assess hundreds of buildings per day from the air, without putting inspectors at risk.
*   **Humanitarian Organizations (NGOs):** Prioritize shelter and reconstruction resources using defensible, quantitative data.
*   **Structural Engineers:** Arrive on site already knowing which buildings need deep inspection.
*   **Displaced Families:** Return home faster with verified structural safety.
*   **Buildings Without Drawings:** The drone captures the reality of the building as it stands — no archived drawings required.

---

## 🛠️ Current Status & Roadmap

**Current Stage:** Research & Early Prototype

We are building this in clear phases:

*   **Phase 1 — Geometry Extraction (Current):** Develop computer vision pipelines that convert drone imagery into measurable building geometry (facade dimensions, column spacing, story counts).
*   **Phase 2 — Automated ETABS Modeling:** Build the Python API layer that converts extracted geometry into a simplified but code-compliant ETABS model.
*   **Phase 3 — Drift Evaluation & Reporting:** Integrate the analysis loop and generate automated safety verdicts against local code thresholds.
*   **Phase 4 — End-to-End Drone Integration:** Flight planning, image capture protocol, and a full field-ready pipeline.
*   **Phase 5 — AI Damage Detection:** Add deep-learning models to detect visible damage (cracks, spalling, tilt, collapse) and feed those findings into the structural assessment.

---

## 🤝 Join Us! We Need Your Skills

SeismoCheck is an open, multidisciplinary civic-tech project. **We are actively recruiting collaborators.** This is the kind of project that only succeeds with a team spanning robotics, computer vision, and structural engineering.

**We need:**

*   **🛸 Drone / Robotics Engineers:** Flight planning, camera payloads, LiDAR, and data capture protocols.
*   **🤖 Computer Vision / AI Engineers:** Photogrammetry, Structure-from-Motion, point cloud processing, semantic segmentation, and damage detection (OpenCV, Open3D, YOLO, PyTorch).
*   **🐍 Python Developers:** Building the geometry parser, ETABS API layer, reporting engine, and user interface.
*   **🏗️ Structural Engineers:** Validating the simplified modeling logic, code compliance thresholds, and drift criteria (especially the Syrian Code).
*   **📊 Data Scientists:** Building and curating datasets of building imagery and labeled structural damage.
*   **🚀 Field Testers:** Anyone with access to drones or real building sites willing to help validate the pipeline.

### How to Get Started
1.  **Fork** the repository and create your branch (`git checkout -b feature/AmazingFeature`).
2.  **Commit** your changes (`git commit -m 'Add some AmazingFeature'`).
3.  **Push** to the branch (`git push origin feature/AmazingFeature`).
4.  Open a **Pull Request**.
5.  Or open an **Issue** to share ideas, report bugs, or propose a new module.

---

## 📬 Let's Connect

If you want to contribute, have ideas, own a drone, or want to collaborate on research — reach out directly. I'm especially interested in connecting with drone operators and CV engineers in the region.

*   **Author:** Ahmad Ghassan Deeb
*   **Email:** [ahmaddeebdev@gmail.com](mailto:ahmaddeebdev@gmail.com)
*   **Phone / WhatsApp:** +963 984 074 032
*   **LinkedIn:** [linkedin.com/in/ahmad-deeb-13699b39a](https://www.linkedin.com/in/ahmad-deeb-13699b39a)
*   **Location:** Damascus, Syria (Open to global remote collaboration)

*If you believe in this mission, please give this repository a ⭐ **Star** to help more engineers, drone pilots, and AI developers find us!*

---
*Built to make the world a safer place — one drone flight at a time.*
