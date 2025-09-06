# Atlas Bake – A Blender Add-on for Texture Baking

## Overview
**Texture baking** includes complex, multi-material shader setups into a compact set of textures (Base Color, Normal, Metallic, Roughness, AO, Opacity, etc.). Game engines like **Unity** and **Unreal** expect these textures in **specific color spaces** and often **packed** into channels (for example, Unity’s Metallic+Smoothness or Unreal/glTF’s ORM).  

A typical manual workflow can be long and error-prone:

1. Pick the right render engine and samples.  
2. Ensure correct **UVs** (create/unwrap or use a specific UV map).  
3. Create images per map with correct **color space**.  
4. For **Metallic/Roughness**, wire nodes or reroute to Emission to bake scalar data.  
5. Bake **Normal** with the right **convention** (OpenGL Y+ vs DirectX Y-).  
6. Bake **AO** and **Opacity** if needed.  
7. Generate **packed** textures (MS/HDRP/ORM).  
8. Build an **atlas material** for the result.  
9. (Optional) **Export** the mesh with the atlas material to FBX.  
10. Keep everything organized, logged, and **repeatable**.

**Atlas Bake** automates this end-to-end. It bakes the maps you choose (with correct color spaces), supports **OpenGL/DirectX** normal conventions, creates **custom channel packs**, handles **UVs** (manual or Smart UV), supports **high→low projected baking**, builds the **atlas material** (with selected only materials if required), and **exports** the baked FBX, all with clear status & progress bar. It runs in a **safe background subprocess** using a temporary copy of your scene, so your open `.blend` stays untouched and completely responsive.

### Design philosophy: simple workflow, uncluttered UI

This add-on's goal is to make baking fast and predictable without burying you in toggles. If there’s an essential control you feel is missing, tell me. Whilst This is built to stay lean, I'd be happy to add genuinely high-value options.

<p align="center">
  <img width="1576" height="830" alt="process_3" src="https://github.com/user-attachments/assets/a5e171af-b647-4e3a-a4c7-38b9f72541be" />
</p>

---

## Key features at a glance

- **Per-map baking (toggle what you need)**  
  Base Color (sRGB), Emission (sRGB), Normal (Non-Color), Metallic (Non-Color), Roughness (Non-Color), Ambient Occlusion (Non-Color), Opacity/Alpha (Non-Color). Metallic & Roughness are baked via a **temporary Emission reroute** for reliable scalar output.

- **Material selection (whitelist)**  
  **Materials to Include** list: only checked materials are processed in **Atlas / Smart-UV**. Others remain intact so you can keep special materials (glass/decals) separate.

- **Custom Packs (channel packing)**  
  Build your own channel packs (e.g., **MS**, **HDRP mask**, **ORM**) and store them in a **preferences folder** for reuse across projects. Dependency-aware: packs verify required inputs (Metallic/Roughness/AO) before generating. The mentioned packs (**MS**, **HDRP mask**, **ORM**) also exist as presets and are available for use.

- **UV handling: “Smart UV” or “Use Selected UV”**  
  **MANUAL**: pick an existing UV map (aborts safely if none exists).  
  **SMART**: auto-create/unwrap `AtlasBakedUV` with **Angle Limit** and **Island Margin** controls; only applied to **included** materials.

- **High→Low projected baking**  
  **Bake to Target** with **Max Ray Distance**, **Cage Object**, or **Virtual Cage Extrusion** to control projection hits.

- **Exports when you’re done**  
  
  **Export FBX** of a duplicate that has the atlas material assigned. UV0 is enforced for predictable results in engines.
  
  **Keep Final .blend**  
  Writes a clean `.blend` into your output folder containing the baked/atlased mesh and textures. Image paths are saved **relative** (or **packed**, if enabled), and nothing is opened / edited during creation, so your working file stays untouched.

- **Normals Flip Handling**  
  **Normal Convention** selector: **OpenGL (Y+)** or **DirectX (Y−)** with optional **Flip Normal Y**.

- **Performance & quality**  
  **GPU Compute** (when available, optional), **Resolution**, separate **Bake Margin (px)** vs **UV Island Margin**.

- **Clear status & safe operation**  
  Live **status/progress** in the UI without interruption and freezes.  
  Runs in a **background headless Blender subprocess** using a **temporary copy** of your scene, so your open file is **never modified** by the bake.

---

## Installation

1. Download the packaged `.zip`.
2. In Blender: **Edit → Preferences → Add-ons → Install…** and select the file.
3. Enable **Atlas Bake**.
4. Find it in **3D View → N-Panel → Atlas Bake**.

### Add-on Preferences
- **Enable Logs** (global default): writes logs of the bake processes to `logs.txt` in the Output Folder.  
- **Custom Channel Pack Output Folder**: where your **Custom Packs** (channel configs) are saved/loaded.  
  Supports absolute paths and `//` relative to the current `.blend`.
  
<p align="center">
  <img width="690" height="604" alt="atlas_bake_metadata" src="https://github.com/user-attachments/assets/bb40ed4e-9963-4282-8d12-410f5ddba367" /><br>
  <em>Add-On Preferences.</em>
</p>

---

## Quick start

### Simple model baking

1. **Select exactly one source mesh.** (For high→low, also set your **Target**.)  
2. In **Textures**, enable the maps you want.  
3. Choose **UV Mode**:  
   - **MANUAL** → pick **UV Map**; or  
   - **SMART** → set **UV Angle Limit** and **UV Island Margin**.  
4. In **Materials to Include**, check the materials you want to include in the Atlas (by default all are included).  
5. Pick **Normal Convention** (OpenGL vs DirectX). Optionally **Flip Normal Y**.  
6. Set **Resolution**, **Samples**, **Bake Margin (px)**, and **Use GPU** if available.  
7. (Optional) Add **Custom Packs** (MS/HDRP/ORM or your own) — ensure required inputs are enabled.  
8. Choose a valid **Output Folder**.  
9. (Optional) Enable **Export FBX**.  
10. Click **Bake**. Watch **status & progress** update while your open file remains safe and responsive.

### High → Low poly baking (keyboard keys scenario)

Goal: Bake the **High** keyboard (with text meshes) into the **Low** keyboard (no text meshes) so the low-poly keycaps get textures containing the legends.

<p align="center">
  <img width="1258" height="890" alt="process_1" src="https://github.com/user-attachments/assets/f32bc9f2-38e1-4200-bb54-85be553c84d2" /><br>
  <em>The high-poly and low-poly models.</em>
</p>

1. **Select Source & Target**  
   - Select **High** (source), enable **Bake to Target**, set **Target = Low** (receiver).  
   - Keep both objects aligned; identical origins minimize projection errors.  

2. **Apply Transforms**  
   - On both meshes: **Apply Scale & Rotation** (Object → Apply). This avoids normal skewing and projection offsets.

3. **Prepare UVs on Low**  
   - **UV Mode**: MANUAL or SMART.  
   - **UV island margin**: **8–16 px** for **2–4K** maps (≥4 px for 1K).  
   - Hard edges ≈ UV splits; enable Auto Smooth **30–60°** to reduce shading seams.

4. **Projection Control**  
   - Start with **Max Ray Distance** ≈ **0.2–0.5%** of model size (e.g., 2–5 mm on a 1 m span).  
   - For tight gaps/decals, use a **Cage** or **Cage Extrusion** (~0.2–0.5% outward).  
   - Thin legends benefit from a cage to capture shallow relief reliably.

5. **Textures to Bake**  
   - Enable: **Normal**, **Base Color**, **Roughness**, **Metallic**, **AO**, **Emission** (if needed), **Opacity** (if used).  
   - **Scalar passes** (Metallic/Roughness/AO/Opacity) are emission-based → **1–16 samples**.  
   - **Normal/AO** if noisy → **32–128 samples**.

6. **Normals Convention**  
   - Choose **OpenGL (Y+)** or **DirectX (Y–)** to match your engine.  
   - If vertical shading looks inverted, toggle **Flip Normal Y**.

7. **Materials to Include**  
   - Tick only materials that should contribute to the atlas (reduces bake time and cross-hits).

8. **Legend Depth (when needed)**  
   - If legends look flat: give High text a small **thickness 0.2–0.5 mm**, **or** use a **Cage**, **or** increase **Max Ray Distance** slightly.

9. **Output & Performance**  
   - Set a valid **Output Folder**; choose **Resolution** (2K default; 4K for hero shots).  
   - Set **Bake Margin**: **8–16 px** (2–4K).  
   - Enable **Use GPU** if available. Optional **Export FBX**.

<p align="center">
  <img width="477" height="843" alt="keyboard_atlas_settings" src="https://github.com/user-attachments/assets/93d0e30b-c50e-49e3-93fc-9c183135f68f" /><br>
  <em>Add-On Selected Settings.</em>
</p>

10. **Bake & Verify**  
   - Click **Bake** and monitor progress; your open file remains safe and responsive.
   - Apply the baked textures to the **Low** keyboard materials and verify the legends on the keycaps.

<p align="center">
  <img width="1330" height="982" alt="process_2" src="https://github.com/user-attachments/assets/47379327-88a7-44ba-bf0d-6019d8e7bd38" /><br>
  <em>Bake process completed. The low-poly result model with the assigned textures.</em>
</p>

---

## Output & naming

- Textures are saved to your **Output Folder**. Typical names:
  ```
  <Object>_<Map>_<Resolution>.png
  # e.g., Crate_BaseColor_2048.png, Crate_Normal_2048.png
  ```
- Packed outputs follow their pack name, e.g.:
  ```
  <Object>_MS_2048.png
  <Object>_HDRPMask_2048.png
  <Object>_ORM_2048.png
  ```

---

## Contact

For more information or issues, feel free to contact me: **john.deligiannis1@gmail.com**

---

## License

**License:** _GPL-3.0-or-later. See the LICENSE file for details._

