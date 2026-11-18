# RLengine 7.2 Alpha 1 (Pre-Release)

> **Release Stage**: 🧪 **Alpha (Alpha 1 - Pre-Release)**  
> **Repository Mode**: 🔒 Private Repository (`idkhowtoname23/RLengine-7.2-Alpha-1`)  
> **Build Target**: C++23 | Visual Studio 18 2026 / 2022 | DirectX 12 (FL 12_1) & DirectX 11 (FL 11_0)  
> **Code Quality**: Strict Zero Comments Policy enforced across all source, header, shader, script, and build configuration files.

---

## 📌 Overview & Development Status

**RLengine 7.2 Alpha 1** marks the first pre-release milestone of the next-generation RLengine 7.2 series. Engineered to modern **C++23** standards with an emphasis on physical accuracy, material realism, and professional workflow isolation, 7.2 Alpha 1 builds directly on the foundations established in version 7.1 while introducing critical architectural advancements in physics simulation, tangent-space rendering, event-driven scripting, and editor ergonomics.

All automated smoke tests pass cleanly with code `0` across both **DirectX 12 (Shader Model 6.5 DXIL)** and **DirectX 11 (Shader Model 5.0 DXBC)** pipelines.

---

## 🔍 What's New in 7.2 Alpha 1 vs 7.1 — Key Differences

| Architectural Subsystem | RLengine 7.1 | RLengine 7.2 Alpha 1 (New) | Impact & Improvements |
| :--- | :--- | :--- | :--- |
| **Physics Simulation** | Discrete 60 Hz Euler steps with AABB collision | **Continuous Collision Detection (CCD)** + **15-Axis SAT OBB** | Eliminates high-velocity tunneling; enables realistic physical rotation on arbitrary 3D oriented boxes. |
| **Physics Triggers** | Rigid physical separation impulses only | **Trigger Colliders (`IsTrigger`)** + Enter/Stay/Exit Events | Area volumes, pickup zones, and sensor fields without physical repulsion or velocity damping. |
| **Material Pipeline** | Cook-Torrance PBR (Albedo, Roughness, Metallic) | **Tangent-Space PBR Normal Mapping** (`TBN` frame) | Micro-surface details without geometry tessellation; full Normal Map slot in DX12 and DX11 shaders. |
| **Event Architecture** | Direct tight-coupled function calls | **Decoupled Type-Erased EventBus** with `DepthGuard` | Centralized publish-subscribe bus; Lua `RL.Events` API with stack protection against infinite recursion. |
| **Editor Workspace** | Fixed monolithic Blender layout | **Isolated Blender 4.x DCC Station** + Floating REPL Console | Clean non-colliding layout; Outliner and Properties isolated from Viewport; floating non-dockable console (`NoDocking`). |
| **Command History** | Basic stack undo | **Deep Entity Cloning & Undo/Redo** (`CommandHistory`) | Serialization snapshots capturing 16 components with deep subtree duplication and reparenting undo. |
| **OS Memory Telemetry** | Log-based profiling | **Real-Time `MemoryStats` Telemetry** | Real-time Working Set, Private Committed Bytes, and Peak RAM monitoring via Win32 PSAPI. |
| **Fallback RHI** | Basic DX11 rendering | **DX11 Normal Map Fallback + PCF Shadows** | Full tangent normal decoding and 3x3 PCF directional shadows on legacy `D3D_FEATURE_LEVEL_11_0` hardware. |

---

## 🚀 Deep Dive: Major New Features in 7.2 Alpha 1

### 1. ⚡ Continuous Collision Detection (CCD) & Trigger Zones
* **High-Speed Anti-Tunneling**: Discrete physics engines suffer from tunneling when high-velocity objects travel further than collider thicknesses in a single frame. 7.2 Alpha 1 introduces ray-cast swept sphere continuous collision detection (`m_ccdEnabled = true`, `SetCCDEnabled`). Dynamic rigidbodies predict motion paths and clamp time-of-impact (TOI) against geometry before position integration, preventing penetration up to 900+ units/sec.
* **Non-Physical Triggers**: Both `BoxColliderComponent` and `SphereColliderComponent` now feature an `IsTrigger` toggle. Triggers detect overlap and dispatch lifecycle events (`PhysicsTriggerEvent`) to the EventBus without applying physical collision impulses.

### 2. 🪐 15-Axis Separating Axis Theorem (SAT) OBB Physics
* **Arbitrary 3D Rotations**: Version 7.1 approximated oriented collisions through axis-aligned bounding boxes (AABB). Version 7.2 Alpha 1 implements the full 15-axis Separating Axis Theorem:
  * 3 face normals of Box A ($A_0, A_1, A_2$)
  * 3 face normals of Box B ($B_0, B_1, B_2$)
  * 9 cross products of box edge directions ($A_i \times B_j$)
* **Exact Contact Manifolds**: Computes minimal penetration depths, contact normals, and contact points, feeding an angular impulse solver ($\Delta \omega = I^{-1}(r \times J)$) with accurate 3D inertia tensors.

### 3. 🎨 Tangent-Space PBR Normal Mapping (TBN Frame)
* **Vertex Tangent Stream**: Vertex buffers (`MeshVertex`) are extended with 16-byte aligned `DirectX::XMFLOAT4 Tangent` vectors (with handness sign in `w`).
* **Procedural & glTF Tangents**: `GLTFLoader` and procedural primitives (Cube, Sphere, Cylinder, Plane, Grid) compute orthonormal Gram-Schmidt tangents and bitangents, preventing degenerate UVs and shader NaNs.
* **HLSL Normal Perturbation (`mesh3d.hlsl`)**: Reconstructs the orthonormal $TBN = [T, B, N]$ matrix on the GPU, transforms sampled tangent-space normal vectors into world-space, and applies Cook-Torrance GGX microfacet specular reflections with ACES tonemapping.
* **DirectX 11 Fallback**: Includes flat normal fallback (`0xFFFF8080`, tangent $\{0, 0, 1\}$) when normal textures are omitted.

### 4. 📡 Unified EventBus with Lua `RL.Events` & `DepthGuard`
* **Decoupled Communication**: Centralized publish-subscribe message broker allowing engine systems and scripts to communicate without direct dependencies.
* **Subscribed Events**: `EntityCreatedEvent`, `EntityDestroyedEvent`, `CollisionEvent`, `PhysicsTriggerEvent`, `WindowResizedEvent`, `ScriptReloadEvent`.
* **Lua 5.4 Scripting API**: Scripts register callbacks via `RL.Events.Subscribe("Collision", function(e) ... end)`.
* **Stack Overflow Prevention**: The `DepthGuard` recursion guard (depth limit: 8) detects and aborts re-entrant event cascades when script callbacks trigger chained entity creations or scene modifications.

### 5. 🎛️ Isolated Blender 4.x DCC Editor & Ergonomics
* **Dedicated Window Zones**: The 3D Viewport, Left Tool Shelf, Right Outliner (Scene Hierarchy), Right Properties (Inspector), and Bottom Timeline occupy dedicated, non-overlapping screen spaces.
* **Safe Overlay Padding**: Diagnostic text (`User Orthographic / Perspective`, Camera Coordinates, FPS) dynamically offsets from the Tool Shelf to guarantee zero visual collisions.
* **Floating Non-Dockable REPL Console**: The in-game Lua console (`~` or `F1`) is equipped with `ImGuiWindowFlags_NoDocking`, ensuring it acts as a floating terminal and cannot accidentally split or corrupt viewport docking nodes.
* **Reset Layout Shortcut**: `Window ➔ Reset Layout to Default` in the main menu bar allows one-click recovery of the standard Blender 4.x workspace layout.

---

## 🏗️ Building and Verification

### Prerequisites
* **OS**: Windows 10 / 11 (x64)
* **Compiler**: Visual Studio 2026 or 2022 (MSVC `/std:c++23`)
* **Build System**: CMake 3.25+
* **SDK**: Windows SDK with DirectX 12 and DirectX 11 support

### Build Commands
```powershell
# Configure CMake
cmake -B build -G "Visual Studio 18 2026" -A x64

# Build Release binary
cmake --build build --config Release --parallel
```

### Automated Verification Smoke Tests
```powershell
# Automated DirectX 12 Smoke Test (15 frames + physics + serialization verification)
.\build\bin\Release\RLengine7.2.exe --smoke-test

# Automated DirectX 11 Fallback Smoke Test
.\build\bin\Release\RLengine7.2.exe --smoke-test --dx11
```

---

## 📦 Pre-Release Archive Contents

The pre-release binary archive (`RLengine-7.2-Alpha-1-Windows-x64.zip`) contains:
* `RLengine7.2.exe`: Optimized Release executable.
* `dxcompiler.dll`, `dxil.dll`: Official Microsoft DirectX Shader Compiler runtime.
* `shaders/`: DX12 and DX11 HLSL shaders (`mesh3d.hlsl`, `quad.hlsl`, `grid3d.hlsl`, `shadow_depth.hlsl`).
* `scripts/`: Lua 5.4 script files (`main.lua`).
* `assets/`: Procedural 3D models, textures, branding, and default scene files.
* `imgui.ini`: Clean editor workspace layout.

---

## 📜 License
Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
