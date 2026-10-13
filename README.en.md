# NING YI AI · Assembly Verification System — YOLOv26n

**The next step in assembly inspection: from "detecting an action" to "maintaining an assembly-process state"**

[中文版 README](README.md) · [Live article page (EN)](https://nyav.github.io/assembly-check/index.en.html) · [Live article page (中文)](https://nyav.github.io/assembly-check/)

---

## Overview

`NING YI AI · Assembly Verification YOLOv26n` is a real-time visual inspection solution for industrial assembly lines. Built on an end-to-end **YOLOv26n** detector, it monitors a single assembly station 24/7, recognising both the **surface condition of the workpiece** and the **assembly actions** being performed — deciding in real time whether the white plug has been seated correctly and whether the instruction sheet has been placed. It replaces manual visual inspection with a traceable machine verdict.

The engineering argument behind it: an assembly inspection system should not just detect an action. It should **maintain a consistent, always-current assembly-process state** that a quality engineer or an MES can actually consume.

## Detection classes (5)

| ID | Class | Stage | Question it answers |
| --- | --- | --- | --- |
| 0 | Visual check — top | Inspect | Is the top surface acceptable? |
| 1 | Visual check — copper body | Inspect | Is the copper body acceptable? |
| 2 | Installing white plug | Assemble | Is the plug-insertion action taking place? |
| 3 | Placing instruction sheet | Assemble | Is the instruction sheet being placed? |
| 4 | White plug seated | Confirm | Is the plug fully seated — completion verified? |

Classes 0/1 decide *whether the part may be assembled*, class 2 is *the assembly action in progress*, class 4 is *the completion check that drives the state machine*. Together they close the loop.

## Technical parameters

- Model: **YOLOv26n** — end-to-end, NMS-free · ~2.4 M parameters · 43 % faster CPU inference
- Detection classes: 5
- Frame rate: 25 FPS
- Resolution: 1920 × 1080
- Frames processed: 6,701
- Output tensor: fixed shape `(N, 300, 6)` — directly consumable downstream
- Frame-level traceability: every verdict carries a timestamp

## Live pages

- **English article** — <https://nyav.github.io/assembly-check/index.en.html>
- **中文文章** — <https://nyav.github.io/assembly-check/>
- Source of the page: [`index.en.html`](index.en.html) / [`index.html`](index.html) (deployed via GitHub Pages)

## Screenshots (on-site, captured from live screen recording)

Four key frames covering inspect → assemble → confirm:

| No. | Stage | Frame |
| --- | --- | --- |
| 01 | Inspect | `images/01_外观检查_顶端.jpg` — top surface check |
| 02 | Inspect | `images/02_外观检查_铜体.jpg` — copper body check |
| 03 | Assemble | `images/03_装白色堵头.jpg` — plug insertion |
| 04 | Confirm | `images/04_白色堵头装配完成.jpg` — plug seated, verified |

All images live in [`images/`](images/) and are embedded in the article page. Class 3 (instruction sheet) was not triggered in this recording — it is an optional station, so no frame is attached.

---

© 凝逸Ai · nyav
