# FaceMocap

An add-on for real-time facial motion capture for Blender 4.2 and later verisions

![Blender](https://img.shields.io/badge/Blender-4.2%2B%20%7C%205.2-orange?logo=blender&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11%2B-blue?logo=python&logoColor=white)
![MediaPipe](https://img.shields.io/badge/MediaPipe-Face%20Landmarker-00A8E8?logo=google&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-Real--Time-green?logo=opencv&logoColor=white)

**FaceMocap** is a Blender add-on for real-time, markerless facial motion capture using a standard consumer RGB webcam. Powered by **Google MediaPipe Face Mesh** (468 3D landmarks) and stabilized via an adaptive **One Euro Filter ($1\text{€}$)**, it allows animators to drive both lightweight prototyping rigs and complex production-ready character armatures directly inside the 3D Viewport.

> 💡 *Insert a GIF or video link of your webcam preview next to the Blender viewport here!*  
> `![Demo](docs/demo.gif)`

---

## Key Features

* **Zero Per-User Training:** Works out-of-the-box on any face using a quick 30-frame neutral pose calibration to estimate rig scale and head orientation.
* **Dual Rigging Architecture:**
  * **Base Rig:** Automatically generates a hierarchical facial armature adapted to your mesh's bounding box, complete with automated island-aware bone heat weight binding. Ideal for topology testing, rapid prototyping, and shape key sculpting.
  * **Advanced Production Retargeting:** Generates a 60-bone `Source Rig` (landmarks) that drives a customizable `Target Rig` (up to 259 bones, inspired by the Blender Studio *Storm* rig and the *3-Curve Principle*) via a customizable UI mapping table (`RetargetMap`).
* **Anatomically Plausible Kinematics:** Decomposes jaw rotational lever mechanics (temporomandibular joint pivot) from residual local translations of the lower lip and mouth corners.
* **Adaptive $1\text{€}$ Noise Filtering:** Replaces standard static EMA smoothing with a velocity-adaptive low-pass filter (`OneEuroFilter` for 3D vectors and `OneEuroFilterQuaternion` using *slerp* and angular velocity). Eliminates high-frequency jitter when still while maintaining zero lag during fast expressions (e.g., eye blinks, speech).
* **Live OpenCV Preview:** Optional floating webcam feed with real-time landmark tessellation overlay, synchronized bidirectionally with the Blender UI.

---

## 📂 Repository Structure

```text
FaceMocap/
├── __init__.py              # Add-on registration & dependency bootstrapper
├── auto_load.py             # Automatic module & class loader
├── core/
│   ├── config.py            # Bone tables, landmark mappings & default constants
│   ├── one_euro.py          # 1€ Filter implementation (Vector & Quaternion)
│   ├── properties.py        # Blender RNA properties & UI state
│   ├── retarget.py          # RetargetSolver for Source-to-Target rig mapping
│   ├── rig.py               # Armature generation & pose utilities
│   ├── solver.py            # Coordinate space transformations & lever kinematics
│   ├── webcam_core.py       # OpenCV capture & MediaPipe Tasks API integration
│   └── weights.py           # Mesh cleanup & island-based automatic weight binding
├── gui/
│   └── gui.py               # 3D Viewport N-Panel UI & Mapping UIList
└── operators/
    ├── diagnostics.py       # Console scale & calibration diagnostics
    ├── import_model.py      # Custom 3D model importer & adaptive rig operators
    ├── mocap.py             # Main modal operator (30 FPS timer loop)
    └── rigging.py           # Base and Advanced armature generator operators
```

---

## Installation

1. **Download the Repository:** Clone this repository or download it as a `.zip` file.
2. **Install in Blender:**
   * Open Blender and go to `Edit` > `Preferences` > `Add-ons`.
   * Click **Install...** (or *Install from Disk* in Blender 4.2+) and select the downloaded archive.
   * Enable the **FaceMocap** checkbox.
3. **Dependencies & Model Weights:**
   * Required Python packages (`mediapipe`, `opencv-python`, `numpy`) and the MediaPipe `face_landmarker.task` model are handled automatically inside the add-on's local environment upon activation/first run.
4. **Test:**
   * inside the repository you can find a .blend file to test the add-on


---

## Usage Workflow

Open the 3D Viewport sidebar by pressing **`N`** and select the **FaceMocap** tab.

### Workflow A: Base Rig (Fast Prototyping)
1. **Import Model:** Click `Importa Modello` to load your 3D head mesh (`.obj`, `.fbx`, `.glb`, `.blend`), or select an existing mesh in the scene.
2. **Setup Armature:** Click `Generate Armature Base On Mesh`.
3. **Align Bones:** Enter `Edit Mode` on the armature and tweak bone head/tail positions to match your character's facial features (ensure the `Jaw` bone head sits at the ear pivot and the tail at the chin).
4. **Bind Mesh:** Return to `Object Mode`, select your mesh, and click `Link Manually` to compute automatic weights.
5. **Start Capture:** Press `Start Motion Capture`. Keep a neutral, relaxed face for the first 30 frames to complete calibration.

### Workflow B: Advanced Rig (Production Retargeting)
1. Under **Advance Setup**, select your `Source` landmark rig (or generate it via `Generate Source Rig`) and your custom `Target` character rig.
2. Click **Initialize** (refresh icon) to load the default 25-control-bone mapping, and **Validate** (checkmark icon) to verify bone names on your target rig.
3. Adjust per-bone **Mode** (`Translation` / `Rotation`) and **Gain** multipliers directly in the *Target Mapping* list.
4. Click `Start Motion Capture`.

### Controls During Capture
* **`C`**: Recalibrate neutral pose at any time.
* **`ESC`** or **`Right Click`**: Stop capture and close webcam stream.

---

## Tuning the $1\text{€}$ Filter

To calibrate the anti-jitter filter in real-time from the **Capture** panel:
1. Set **Beta** to `0.0` and keep your face completely still. Lower **Cutoff** (`min_cutoff`) until the residual tremor on the bones disappears without making slow movements feel overly sluggish.
2. Keeping that **Cutoff** value, perform rapid facial movements (blink quickly or open your mouth abruptly). Increase **Beta** in small steps until the perceived lag disappears.

---

## References
Information about the face detector model:
[https://developers.google.com/edge/mediapipe/solutions/vision/face_detector#blazeface_short-range](https://developers.google.com/edge/mediapipe/solutions/vision/face_detector#blazeface_short-range)

More details about the MetaPipe FaceMesh:
[https://storage.googleapis.com/mediapipe-assets/Model%20Card%20MediaPipe%20Face%20Mesh%20V2.pdf](https://storage.googleapis.com/mediapipe-assets/Model%20Card%20MediaPipe%20Face%20Mesh%20V2.pdf)

Link Model face_landmarker:
[https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/latest/face_landmarker.task](https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/latest/face_landmarker.task)

1. **MediaPipe Face Mesh:** Y. Kartynnik, A. Ablavatski, I. Grishchenko, and M. Grundmann, *"Real-time Facial Surface Geometry from Monocular Video on Mobile GPUs,"* CVPR Workshop, 2019.
2. **One Euro Filter:** G. Casiez, N. Roussel, and D. Vogel, *"1€ Filter: A Simple Speed-based Low-pass Filter for Noisy Input in Interactive Systems,"* ACM CHI, 2012.
3. **Production Facial Capture:** S. Laine et al., *"Production-Level Facial Performance Capture Using Deep Convolutional Neural Networks,"* ACM SIGGRAPH / Eurographics SCA, 2017.

---

## 👥 Authors

* **adgx**
* **F3de22** 
