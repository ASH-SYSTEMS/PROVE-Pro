
# PROVE-Pro(totype) Web Application — Full Documentation

**Version:** Documentation v1.01 (for `provepro.html`)  
**Author:** Avi Shaked  
**Date:** 2026-01-04

---

## Table of Contents

1. [User Manual](#user-manual)
   - [Overview](#overview)
   - [Quick Start](#quick-start)
   - [Toolbar & Panels](#toolbar--panels)
   - [Editing the Main Diagram](#editing-the-main-diagram)
   - [Artifact Lifecycle View](#artifact-lifecycle-view)
   - [Saving / Loading / Export](#saving--loading--export)
   - [Consistency Report](#consistency-report)
   - [Status Mode](#status-mode)
   - [Edit Connectors Mode](#edit-connectors-mode)
   - [Keyboard & Mouse](#keyboard--mouse)
2. [Web Application Description & Relation to PROVE](#web-application-description--relation-to-prove)
   - [PROVE Essentials](#prove-essentials)
   - [How the App Implements PROVE](#how-the-app-implements-prove)
   - [Design Patterns: Gating, Composite State, Sibling Aggregation](#design-patterns-gating-composite-state-sibling-aggregation)
   - [References](#references)
3. [LLM Developer Handbook](#llm-developer-handbook)
   - [Architecture Overview](#architecture-overview)
   - [Core Data Model & Keys](#core-data-model--keys)
   - [Rendering Pipeline](#rendering-pipeline)
   - [Common Pitfalls & How to Avoid Them](#common-pitfalls--how-to-avoid-them)
   - [Safe Patching Guidelines](#safe-patching-guidelines)
   - [Debugging Playbook](#debugging-playbook)
   - [Feature Extension Playbook](#feature-extension-playbook)
4. [Prompt Library for LLMs](#prompt-library-for-llms)
   - [Bug Fix Prompts](#bug-fix-prompts)
   - [Diagram Feature Prompts](#diagram-feature-prompts)
   - [Information Model / Conceptual Feature Prompts](#information-model--conceptual-feature-prompts)
   - [Feature Extension Prompts](#feature-extension-prompts)
5. [JSON Specification for Import](#json-specification-for-import)
   - [Schema](#schema)
   - [Constraints & Semantics](#constraints--semantics)
   - [Minimal Example](#minimal-example)
   - [Rich Example](#rich-example)
   - [Validation Checklist](#validation-checklist)

---

## User Manual

### Overview
The PROVE‑Pro web application is a **single‑file HTML/SVG** tool to design and analyze process descriptions via the **PROVE** methodology. You create **Activities** (process boxes) and connect them with **Artifact‑in‑State** flows. Multi‑level scoping is supported (parent/child processes). The app emphasizes:
- **Goal‑driven modeling** (outputs first),
- **Composite states** for artifacts,
- **Sibling/internal aggregation** of connectors,
- **Lifecycle view** per artifact,
- **Consistency checks**.

The delivered file in this package: **`index_patched_lifecycle.html`**. For reference, the previously agreed baseline was **`index_patched.html`**.

### Quick Start
1. **Open** the HTML file in any modern browser.
2. The app auto‑creates a **Master Process** box.  
3. Use toolbar buttons:
   - **Add Activity**: create a process box (optionally choose a parent).
   - **Add Artifact**: create a new artifact.
   - **Add Elemental State**: add states to an artifact.
   - **Link Input / Link Output**: connect activities with artifact‑in‑state flows.
   - **Insert Gating**: posts a canonical Development→Inspection→back(Approved) pattern.
   - **Artifact Lifecycle**: open lifecycle view for an artifact.
   - **Check Consistency**: run report.
   - **Status Mode**: visualize achieved artifact states.
   - **Edit Connectors**: edit bendpoints on main diagram connectors.
   - **Reset Connector Layouts**: clear manual routing on main diagram connectors.

### Toolbar & Panels
- **Left Panel**: Model Tree (hierarchy) and Artifacts list.  
  From Artifacts, you can **Open Lifecycle** for a selected artifact.
- **Canvas**: Main PROVE diagram (SVG). Drag to pan, wheel to zoom.  
- **Right Panel**: Selected Activity details; Consistency report; Selected Connector info.

### Editing the Main Diagram
- **Add Activity** → choose parent none/selected to place within hierarchy.
- **Add Artifact / Add Elemental State** → define artifacts and their states.
- **Link Input** (artifact‑in‑state flows pointing **into** activity) / **Link Output** (flows pointing **out** of activity).
- **Drag boxes**: Click & drag the transparent hit area of a box.
- **Zoom/pan**: Mouse wheel; drag canvas background.

**Connector Aggregation** (main diagram):
- Sibling connectors are **aggregated** per `(from, to, artifact, route)`; the label shows all **states** (composite state) in brackets.
- Internal parent↔child reflection flows are **aggregated** per direction.

### Artifact Lifecycle View
Open via the right panel or Artifacts list. Lifecycle nodes are **artifact states**; edges show **activities** that realize transitions.

**New dynamic abilities (this build):**
- **Drag states and START**: move nodes to any location.  
  Positions persist in JSON under `lcLayouts`.
- **Add bendpoints**: click an edge; orange handle appears. **Drag** handles to route.  
  Bendpoints persist in JSON under `lcRoutes` keyed by `"from->to"`.
- **Aggregate transitions**: multiple activities with the same **from→to** show as a **single edge**; label lists all activity names.

### Saving / Loading / Export
- **Save JSON**: writes all activities, artifacts, states, connections, manual routes, lifecycle layouts and routes, achieved states, etc.
- **Load JSON**: restores a previously saved model. The loader safely converts arrays to internal Sets and initializes missing optional fields.
- **Export SVG**: exports the current main diagram SVG.
- **Export Lifecycle SVG**: exports the current lifecycle SVG.

### Consistency Report
Run from the right panel. Checks include:
- Activities missing inputs/outputs.
- Parent/child reflection alignment (internal routing markers).
- Simple forward/sibling input/output expectations.

### Status Mode
Toggle **Status Mode** to color flows for **achieved** artifact states (green vs. blue default). Achieved states are set per artifact.

### Edit Connectors Mode
- Toggle **ON**.
- **Click** a connector once to select; **click again** to add a bendpoint.
- **Drag** bendpoint handles to adjust. 
- **Reset Connector Layouts** to clear manual routes.

### Keyboard & Mouse
- **Wheel**: zoom canvas.  
- **Drag background**: pan canvas.  
- **Drag box**: move activity.  
- **Click connector**: add bendpoint (Edit mode only).  
- **Drag bendpoint**: move.

---

## Web Application Description & Relation to PROVE

### PROVE Essentials
- **Artifact**: physical/logical entity (documents, components, results).  
- **Activity**: process transforming artifacts (creating, consuming, changing state).  
- **Composite State**: artifacts accumulate **elemental states**; a composite is the set of elemental states achieved concurrently.  
- **Scoping**: activities can be modeled as processes (hierarchical).  
- **Gating**: approval/inspection processes implementing governance transitions.

**Primary sources:**  
- Shaked & Reich (2019), *Systems Engineering* — PROVE method & notation, composite states, gating, scoping. citeturn6file14  
- Shaked (2022), *Software Impacts* — PROVE Tool views, lifecycle/status, DSM perspective. citeturn6file13

### How the App Implements PROVE
- **Activities & Artifacts‑in‑State**: The boxes and arrows implement the PROVE graphical notation, where each arrow is an Artifact carrying an elemental state toward/away from an activity. citeturn6file14
- **Composite States**: Connector labels on the main diagram aggregate multiple elemental states; lifecycle edges list transitions per artifact. citeturn6file14
- **Scoping**: Parent/child relations model hierarchical processes; internal reflection flows are aggregated accordingly. citeturn6file14
- **Lifecycle View**: Artifact‑centric state graph; achieved states colorization is aligned with the PROVE Tool status visualization. citeturn6file13
- **Consistency Checks**: Basic completeness/consistency rules inspired by PROVE feedback for process quality and reproducibility. citeturn6file13

### Design Patterns: Gating, Composite State, Sibling Aggregation
- **Gating pattern**: Development→Inspection→back (Ready/Approved).  
- **Composite state**: multiple elemental states listed in labels `{s1, s2, …}`.  
- **Sibling/internal aggregation**: one edge per (from,to,artifact,route) with merged states; internal parent↔child flows aggregated per direction. citeturn6file14

### References
- A. Shaked & Y. Reich, “Designing development processes related to system of systems using a modeling framework,” *Systems Engineering*, 2019. citeturn6file14
- A. Shaked, “PROVE Tool: A tool for designing and analyzing process descriptions,” *Software Impacts*, 2022. citeturn6file13

---

## LLM Developer Handbook

### Architecture Overview
- **Single HTML** app: `index_patched_lifecycle.html` (or `index_patched.html` baseline).
- **SVG Rendering**: main diagram, lifecycle diagram rendered via DOM/SVG APIs.
- **State Object**: global `state` holds model + UI routing:
  - `activities[]`, `artifacts[]`, `offsets`, `achieved`, `routes`, `selectedId`, `selectedEdge`,
  - **Lifecycle additions**: `lcLayouts`, `lcRoutes`.
- **Key functions**:
  - `addActivity`, `addArtifact`, `openDialog`, `render`, `layout`, `buildTree`, `drawPath`, `openLifecycle`, `linkIO`, `insertGating`, `renderConsistencyReport`.

### Core Data Model & Keys
- **Activity**: `{ id, name, parentId|null, isMaster:boolean, inputs:[], outputs:[] }`
- **IO Link**: entries in `inputs`/`outputs`: `{ artifactId, state }`
- **Artifact**: `{ id, name, states:Set }`
- **Routes (main diagram)**: `routes[key] = [{x,y}, …]` where `key = edgeKey(fromId,toId,artifactId,statesCsv)`
- **Lifecycle Layouts**: `lcLayouts[artifactId] = { nodes: { stateName:{x,y}, … }, start:{x,y} }`
- **Lifecycle Routes**: `lcRoutes[artifactId]['from->to'] = [{x,y}, …]`

### Rendering Pipeline
1. **Hierarchy** → `buildTree()` with cycle guard and `layout()` auto‑placement, offsets applied, grow boxes to contain children.  
2. **Index** → `indexNodes(tree)` caches activity box rectangles.  
3. **IO Index** → producer/consumer by artifact/state.  
4. **Aggregation** → sibling/internal connectors aggregated; external inputs/goals grouped.  
5. **Draw** → `drawPath()` builds path string, labels, bendpoints (edit mode).  
6. **Lifecycle** → `openLifecycle()` builds nodes, aggregates transitions per `from→to`, draws cubic or polyline via bendpoints.

### Common Pitfalls & How to Avoid Them
- **ID collisions after Load**: Ensure `addActivity`/`addArtifact` generate **unused IDs** when creating new elements. (The baseline already guards this.)
- **Recursion cycles**: Guard `buildTree()` with a `seen` Set to avoid stack overflow with cyclic parent/child references.
- **Malformed JSON.stringify**: Always keep `lcLayouts`/`lcRoutes` **inside** the object literal; `JSON.stringify(obj, null, 2)` — do not add extra arguments between.
- **Edge key mismatch**: Keep lifecycle edge keys as `from->to` and main diagram keys as `edgeKey(from,to,artifact,statesCsv)`.

### Safe Patching Guidelines
- **Preserve UI and behaviors** unless explicitly asked to change.
- **Touch only the requested subsystem** (e.g., lifecycle) and keep API/labels intact.
- **Test after patch**: run in browser, open console; confirm Save/Load work and no syntax errors.
- **Keep optional JSON fields optional** for backward compatibility.

### Debugging Playbook
1. **Reproduce**: load the reported JSON, perform user’s steps.
2. **Inspect console**: syntax errors (line/column), RangeError traces (recursion loops).  
3. **Instrument**: log keys and counts in aggregation maps; verify expected merges.  
4. **Minimal model**: create a tiny test with 2 activities, 1 artifact, 2 states; confirm connector directions/labels.  
5. **Lifecycle**: confirm bendpoint arrays exist, labels show aggregated activities.

### Feature Extension Playbook
- **New view**: add panel and data mapping; keep existing panels intact.
- **New rules**: extend `renderConsistencyReport`; keep output HTML simple.
- **New aggregation**: add a map pass before drawing; merge labels.
- **New persistence**: add fields to JSON object literal and loader; keep optional.

---

## Prompt Library for LLMs

### Bug Fix Prompts
- *Connector save error*:  
  “Open `index_patched_lifecycle.html`. In the **btnSave** handler, ensure `JSON.stringify({ routes:state.routes, lcLayouts:state.lcLayouts, lcRoutes:state.lcRoutes }, null, 2)` uses **exactly three arguments** and includes lifecycle fields **inside** the object. Remove any duplicate lines after the handler.”

- *Recursion RangeError*:  
  “Add a `seen` Set in `buildTree()`; if a child ID is already seen, return a leaf node and do not recurse.”

### Diagram Feature Prompts
- *Add curved internal edges only*:  
  “In `render()`, for internal parent↔child reflection, keep aggregation but draw **cubic curves** instead of polylines. Do not change labels.”

- *Add connector labels with composite states sorted*:  
  “In aggregation, call `states.sort()` before joining; ensure label format stays `${artifact} :: [state1, state2]`.”

### Information Model / Conceptual Feature Prompts
- *Add achieved status to lifecycle nodes*:  
  “In `openLifecycle()`, color nodes green if `isAchieved(artifactId, state)`, else gray; keep labels unchanged.”

- *Add DSM (Design Structure Matrix) read‑only*:  
  “Create a right‑panel section that renders a DSM table from producer/consumer indices; read‑only; no changes to main UI.”

### Feature Extension Prompts
- *Reset lifecycle layout*:  
  “Add a ‘Reset Layout’ button inside the lifecycle dialog that clears `lcLayouts[artifactId]` and `lcRoutes[artifactId]` and re‑renders.”

- *Delete bendpoint gesture*:  
  “In `openLifecycle()`, on **Shift+Click** near a bendpoint, remove the closest point from `lcRoutes[artifactId][edgeKey]` and re‑render.”

---

## JSON Specification for Import

This schema lets any LLM produce a JSON file importable by the app. Fields marked **optional** may be omitted; the loader initializes them.

### Schema
```json
{
  "activities": [
    {
      "id": "act1",               // unique string
      "name": "Development",
      "parentId": null,            // or parent activity id
      "isMaster": true,
      "inputs":  [ { "artifactId": "art1", "state": "Approved" } ],
      "outputs": [ { "artifactId": "art1", "state": "Ready"    } ]
    }
  ],
  "artifacts": [
    { "id": "art1", "name": "Design", "states": [ "Ready", "Approved", "CDR Ready" ] }
  ],
  "offsets": { "act1": { "dx": 0, "dy": 0 } },
  "achieved": { "art1": [ "Approved" ] },
  "routes": {
    // main diagram optional manual bendpoints. Key is edgeKey(fromId,toId,artifactId,statesCsv)
    "act1->act2:art1::Ready,Approved": [ { "x": 320, "y": 180 } ]
  },
  "lcLayouts": {                  // optional lifecycle layouts per artifact
    "art1": {
      "nodes": { "Ready": { "x": 160, "y": 120 }, "Approved": { "x": 340, "y": 180 } },
      "start": { "x": 60, "y": 240 }
    }
  },
  "lcRoutes": {                   // optional lifecycle bendpoints per artifact
    "art1": {
      "Ready->Approved": [ { "x": 240, "y": 160 } ]
    }
  }
}
```

### Constraints & Semantics
- **IDs** must be unique (`activities[i].id`, `artifacts[j].id`).
- **States** are **elemental** strings; composite states arise by **aggregation**.
- **Inputs/Outputs** define artifact‑in‑state flows for each activity.
- **parentId** defines hierarchy; cycles are discouraged (loader will still guard rendering).
- **routes** (main diagram) and **lcRoutes** (lifecycle) are optional manual bendpoints.
- **lcLayouts** records lifecycle node positions.
- **achieved** lists achieved states per artifact.

### Minimal Example
```json
{
  "activities": [ { "id": "act1", "name": "Master Process", "parentId": null, "isMaster": true, "inputs": [], "outputs": [] } ],
  "artifacts": [ { "id": "art1", "name": "Design", "states": [ "Defined" ] } ]
}
```

### Rich Example
```json
{
  "activities": [
    { "id": "act1", "name": "Master Process", "parentId": null, "isMaster": true,
      "inputs":  [ { "artifactId": "art1", "state": "Approved" } ],
      "outputs": [ { "artifactId": "art2", "state": "Validated" } ] },
    { "id": "act2", "name": "Development", "parentId": "act1", "isMaster": false,
      "inputs":  [ { "artifactId": "art1", "state": "Ready" } ],
      "outputs": [ { "artifactId": "art1", "state": "Approved" }, { "artifactId": "art2", "state": "Validated" } ] }
  ],
  "artifacts": [
    { "id": "art1", "name": "Design", "states": [ "Ready", "Approved" ] },
    { "id": "art2", "name": "Product", "states": [ "Validated" ] }
  ],
  "offsets": { "act2": { "dx": 60, "dy": 0 } },
  "achieved": { "art1": [ "Approved" ] },
  "routes": {},
  "lcLayouts": {
    "art1": { "nodes": { "Ready": { "x": 160, "y": 120 }, "Approved": { "x": 360, "y": 140 } }, "start": { "x": 60, "y": 240 } }
  },
  "lcRoutes": { "art1": { "Ready->Approved": [ { "x": 260, "y": 130 } ] } }
}
```

### Validation Checklist
- Activities/Artifacts present; IDs unique.
- Each IO link references existing **artifactId** and a known **state** for that artifact.
- No syntax errors; JSON is valid and loads without console errors.
- Optional fields (`routes`, `lcLayouts`, `lcRoutes`, `achieved`, `offsets`) may be absent; the app initializes defaults.

---

> **Note on Sources & Methodology Alignment:** All modeling concepts (artifact, activity, composite states, lifecycle, gating, scoping) and their use in this app are grounded in the PROVE research publications. See Shaked & Reich (2019) and Shaked (2022) for method details, notational foundations, and tool views. citeturn6file14turn6file13

