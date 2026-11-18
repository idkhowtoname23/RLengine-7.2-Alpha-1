# 🚀 RLengine 7.2 Alpha 1 — Pre-Release Notes

RLengine 7.2 Alpha 1 represents the first milestone of the 7.2 release cycle, focusing on high-velocity physics precision, micro-surface material detail, event-driven decoupling, and an isolated Blender 4.x DCC editor layout.

---

## 🌟 Highlights & Major Changes

### 1. Continuous Collision Detection (CCD)
* Ray-cast swept sphere continuous collision detection against static and dynamic geometries.
* Clamps Time-of-Impact (TOI) to prevent fast projectiles (tested up to 900+ units/sec) from tunneling through thin walls.
* Controlled via `m_ccdEnabled` and `PhysicsSystem::SetCCDEnabled(bool)`.

### 2. Trigger Zones (`IsTrigger`)
* Non-physical trigger volumes on both `BoxColliderComponent` and `SphereColliderComponent`.
* Overlap detection dispatches `PhysicsTriggerEvent` without applying velocity damping or physical separation impulses.

### 3. 15-Axis SAT OBB Rigid Body Physics
* Full 3D Separating Axis Theorem (SAT) collision solver across 15 candidate axes.
* Manifold contact generation for oriented boxes with angular momentum integration ($\Delta \omega = I^{-1}(r \times J)$).

### 4. Tangent-Space PBR Normal Mapping (TBN)
* Extension of vertex stream with 16-byte aligned `Tangent` vectors ($TBN = [T, B, N]$ frame).
* Normal map perturbation in HLSL Cook-Torrance GGX shaders with ACES tonemapping.
* DX11 fallback pipeline with flat normal compatibility.

### 5. Decoupled EventBus with `DepthGuard`
* Global publish-subscribe broker for lifecycle and physics events.
* Lua 5.4 scripting bridge via `RL.Events.Subscribe`.
* Stack protection recursion guard (`DepthGuard`) limiting depth to 8 to prevent infinite event cascades.

### 6. Blender 4.x DCC Station & Floating REPL Console
* Fixed overlapping dock splits: 3D Viewport, Left Tool Shelf, Right Outliner, Right Properties, and Bottom Timeline occupy dedicated non-colliding areas.
* Safe padding between diagnostic overlay and tool shelf.
* In-game Lua REPL Console (`~` or `F1`) converted to non-dockable floating overlay (`ImGuiWindowFlags_NoDocking`) preventing accidental docking splits.
* `Window ➔ Reset Layout to Default` added to main menu bar.

---

## 📊 Summary of Differences from 7.1

| Component | RLengine 7.1 | RLengine 7.2 Alpha 1 |
|---|---|---|
| Collision Detection | Discrete 60Hz AABB | **Swept-Sphere CCD + 15-Axis SAT OBB** |
| Sensor Zones | None (Impulse only) | **Trigger Colliders (`IsTrigger`)** |
| Materials | PBR Albedo / Rough / Metal | **PBR Normal Mapping (TBN)** |
| Event Architecture | Direct function calls | **Decoupled EventBus + `DepthGuard`** |
| Editor Layout | Monolithic fixed windows | **Blender 4.x DCC Station + Floating REPL** |
| Command History | Basic stack undo | **Deep Entity Cloning & Undo/Redo** |
| Memory Profiling | Basic logs | **Real-Time `MemoryStats` (PSAPI)** |

---

## 🛠️ Verification Commands
```powershell
# Automated DirectX 12 smoke test
.\RLengine7.2.exe --smoke-test

# Automated DirectX 11 Fallback smoke test
.\RLengine7.2.exe --smoke-test --dx11
```
