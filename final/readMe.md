# CS460 Final Project — Prosthetic Biomechanics Simulator (WebGL / Three.js)

Author: **Syed Taswar Mahbub**  
Demo (GitHub Pages):  
https://taswarmahbub.github.io/cs460student/final/index.html

## Project Vision (Why this exists)
This project is an interactive **prosthetic biomechanics simulator** designed to help (especially in a **nursing education context**) visualize:
- **Anatomy + gait mechanics** in an immersive 3D environment
- **Pressure risk areas** (pressure ulcer prevention mindset)
- The relationship between **socket fit**, **load**, **movement**, and **tissue stress**

The simulator emphasizes **real-time visualization** of socket pressure + deformation to support learning and assessment workflows.

---

## Core Features (Currently Achieved in the Project)

### 1) Real-time 3D prosthetic visualization
- Web-based 3D viewer using **Three.js** + **WebGL**
- OrbitControls: drag to rotate, scroll to zoom, right-drag to pan
- Scene polish: fog, shadows, ground plane + grid, contact shadow circle

### 2) Prosthetic knee rig (upper fixed, lower rotates)
- Loads a GLB prosthetic model and attempts to locate a **knee joint node** by name:
  - `KNEE_NODE_NAME = "innerleg_low001"`
- **Splits the model** around the knee pivot:
  - meshes above knee → `upperPart` (fixed)
  - meshes below knee → `lowerPart` (rotates)
- Result: only the lower section responds to knee motion, improving realism.

### 3) Socket pressure heatmap overlay (vertex colors)
- A semi-transparent **socket cylinder overlay** renders a **color heatmap** using vertex colors.
- Pressure is **synthetic/educational** (fast + stable), driven by:
  - effective load (stance vs swing)
  - bend amount (knee angle)
  - socket tightness
- Visual intensity increases with load using emissive glow.

### 4) Socket fit + deformation controls
- **Socket Tightness** scales the socket overlay and slightly scales the upper assembly (fit effect).
- **Load Compression** compresses the lower leg (simple deformation cue).

### 5) Gait simulation + knee dynamics (mass–spring–damper)
You can drive the knee in two ways:
- **Manual Angle** slider
- **Auto Gait** sinusoidal target

Then the knee motion is physically smoothed using a tunable dynamic model:
- Inertia
- Stiffness
- Damping
- Extension/Flexion limits
- End-stop spring + damping (prevents unrealistic overshoot)

### 6) Ground contact estimation + effective load
- Estimates foot position and determines **contact** with the ground plane
- Computes **effective load**:
  - stance: full load
  - swing: reduced load (default 25%)
- Displays contact status in the UI readout.

### 7) Ground Reaction Force visualization
- A 3D **force arrow** appears at the estimated foot location.
- Arrow length scales with effective load.
- Adds a subtle forward component tied to knee angle to feel “alive”.

### 8) UI + HUD quality-of-life
**Tweakpane controls**
- Joint Angle, Load, Tightness, Opacity, Wireframe, Auto Gait, Flip Lower Leg
- Gait folder (Speed, Flex Amp)
- Knee Dynamics folder (inertia/stiffness/damping/limits/end-stops)
- Readout folder (target angle, measured angle, velocity, contact)
- Presets: Standing, Walking, High Load, Loose Socket, Tight Socket

**HUD overlay + keyboard shortcuts**
- `H` toggle help HUD
- `U` toggle Tweakpane UI
- `R` reset camera + parameters

### 9) Reliable fallback if the GLB fails
- If the model can’t load, the program **does not break**:
  - shows an error message
  - spawns a simple procedural “debug leg”
  - keeps all animation + controls testable

### 10) Performance + debugging
- **stats.js** FPS panel
- GLB load overlay tells you the attempted model path and common fixes

### 11) Shadows + depth cues (realism and readability)
- **Real-time shadow mapping** enabled (`renderer.shadowMap.enabled = true`) using **PCFSoftShadowMap** for softer edges.
- A **directional “sun” light** casts shadows (prosthetic onto ground) while ambient/hemisphere lights prevent overly dark areas.
- Shadow quality tuning:
  - higher-resolution shadow map (`2048×2048`)
  - `bias` and `normalBias` adjustments to reduce acne/peter-panning artifacts
- The ground plane is set to `receiveShadow`, and the model parts are set to `castShadow` + `receiveShadow`.

**Contact shadow**
- Adds a small semi-transparent circle (“contact shadow”) under the foot to keep the prosthetic visually grounded even when real shadows are subtle.
- The contact shadow can scale and change opacity with load/contact to reinforce stance vs swing.

---

## Tech Stack / Foundations
- **Three.js** (scene, materials, lighting, controls)
- **WebGL** (GPU rendering pipeline)
- **GLTFLoader** (GLB assets)
- **Tweakpane** (interactive control panel)
- **stats.js** (FPS/perf)

---

Optional add-ons (detailed):

### Cross-section slicing view (interactive cutaway)
Adds a “medical-style” cutaway so the user can look inside the prosthetic/socket region.

**What it does**
- Uses a clipping plane (e.g., horizontal plane on Y) to slice through the model in real time.
- Lets the user move the slice height with a slider to explore internal geometry and alignment.

**How it looks/behaves**
- When enabled, everything above (or below) the plane is clipped away.
- Optional: show a visible “slice plane” helper so users understand where the cut is happening.

**Why it’s useful**
- Great for educational demos: helps explain socket alignment, limb positioning, and how pressure areas relate to internal placement.

**Controls**
- Toggle: `Slice On/Off`
- Slider: `Slice Height`
- Optional: `Invert Slice` (flip which side is visible)

---

### Knee–foot measurement overlay (live distance readout)
Displays a real-time measurement between the knee pivot and the foot position.

**What it does**
- Draws a line from the knee pivot to the estimated foot contact point.
- Computes and displays the distance (in scene units) on screen as text.

**How it updates**
- Updates continuously as the knee rotates and the leg moves during gait.
- If a real foot bone exists in the GLB, it can use that; otherwise it can estimate foot position using the lower leg bounding box.

**Why it’s useful**
- Helps demonstrate how joint angle affects effective limb length and posture.
- Useful for showing extension vs flexion outcomes during gait.

**Controls**
- Toggle: `Show Measurement`
- Optional: `Units Scale` (e.g., 1 unit = 1 cm) if you calibrate the GLB scale

---

### Screenshot capture button (save a frame for reports)
Allows the user to export the current view as an image.

**What it does**
- Captures the WebGL canvas and downloads it as a PNG.
- Useful for assignments, progress reports, or comparing scenarios (tight socket vs loose socket, high load vs normal load).

**Implementation detail**
- Uses `renderer.domElement.toDataURL("image/png")`
- Most reliable when the renderer is created with `preserveDrawingBuffer: true`.

**Why it’s useful**
- Lets you generate “evidence” images for your final report/demo.
- Makes it easy to create before/after comparisons using presets.

**Controls**
- Button: `Save Screenshot`
- Optional: auto filename with timestamp
---

## How to Run Locally

### Folder expectation

project/
  index.html
  models/
    prosthetic_leg.glb


### Start a local server (required)
GLB loading won’t work reliably from `file://` due to browser security/CORS.

**Option A (Python)**
```bash
cd project
python3 -m http.server 8000
```
Then open:
```text
http://localhost:8000
```
## How to Run on the Web

```Github Pages
https://taswarmahbub.github.io/cs460student/final/index.html
```

---

## Configuration Knobs (Quick customization)
Inside `index.html`:
- `MODEL_PATH` — path to the GLB
- `KNEE_NODE_NAME` — the name of the joint node used to place the knee pivot
- camera presets in the UI buttons

---

## Known Limitations (By design / current stage)
- Pressure is **not clinical** (synthetic visualization model for learning + interaction).
- Foot position/contact is **estimated** (simple heuristic) rather than measured from a true foot bone/marker.
- Deformation is a **visual proxy** (scaling) rather than FEM/soft-tissue simulation.

---

## Challenges & Next Steps (Planned / Future Additions)

### Current challenges
- Improving real-time biomechanics accuracy
- Better vertex-level pressure modeling
- More precise heatmap interpolation
- Balancing performance with realism
- Gait animation synchronization
- Dynamic socket geometry (true shape change vs scale)

### Future enhancements (high-value upgrades)
- **Integrate additional real prosthetic GLB models** and choose between them at runtime
- **Shear stress visualization** (tangential forces) alongside normal pressure
- A “**Clinical Mode**” with preset scenarios (common fit issues, skin-integrity warnings)
- More explicit **ground reaction force** visualization (vectors, moments, time curves)


