# ocr-kylin-v11-Linux64_ctc.so · Extracting Key Frames from Video · Engineering Notes

> English version · [中文版](ocr-kylin_拆视频取关键帧_经验笔记.md) · local paths genericised (`<…>` = placeholder)
>
> Source: `github.com/nyav/ocr-kylin-v11-Linux64_ctc.so` (the Kylin V11 + RTX 3060 OCR benchmark repository).
> Core idea: **split the video → pull key frames → pick the on-site screenshot for each action → number and caption them** (the README carries numbered 01–05 on-site frames, a metrics table and a cover).
> Applied example: four on-site frames were extracted from the assembly-check recording (visual check — top / copper body / plug insertion / plug seated), embedded in the article page and pushed to the GitHub repository.

---

## 1. The approach in one sentence

> **Treat the on-site recording as evidence material: split frames → locate the key frame for each action on demand → extract clean on-site stills → number and caption them (01/02/03…) alongside a metrics table and a cover, for showcasing and publishing.**

This is not "grabbing a few frames" — it is **deliberate selection**: every image must correspond to an identifiable action or state, and the evidence must be visible in the image itself (detection boxes, red class labels, logs).

## 2. The "on-site screenshot" idiom in the presentation layer (how the OCR repo README does it)

The OCR repo README is the end state of this approach:

1. **Cover images** (landscape cover-16x9 + portrait cover-9x16).
2. **Metrics table**: one measured value per row (302 images/s throughput, 3 ms, 99.0 %, 14 concurrent channels…), all genuinely measured.
3. **Numbered 01–05 on-site screenshots**: each with a line of description, all "captured live, not mock-ups".
   - 01 deployment environment → 02 start-up / loading → 03 concurrent inference → 04 recognition results → 05 metrics summary.
4. **Applicable scenarios + full-stack self-controlled deployment note.**

> The discipline that matters: **images must be real runtime output** (UI, logs and detection boxes clearly visible), and the text states only measured values — never adjusted data. That is the difference between an "on-site screenshot" and a "mock-up", and it is where the credibility comes from.

## 3. How to split a video and pull key frames (assembly-check walkthrough)

### 3.0 Probe the video first

```
ffmpeg -hide_banner -i <video>   # Duration / resolution / frame rate
```

Measured on the assembly-check source: 51.02 s · 1318×598 · 24.7 fps — which disagrees with the 1920×1080 recorded in the detection log. Trust the actual file.

### 3.1 Split at 1 fps to build the full set

```
ffmpeg -y -i <video> -vf "fps=1" out\f_%03d.jpg   # 51 s → 51 frames
```

### 3.2 Build a contact sheet (PIL) to locate everything at once

- First build a **full-frame contact sheet** to read the overall rhythm.
- If the target (the red overlay text) is too small, build a **zoomed contact sheet cropped to the target region** (here: the left-hand video area, `40,60,660,560`).
- Label each cell with its timestamp — reading 51 frames in one pass is far more efficient than frame by frame.

### 3.3 Read the frames to determine the "currently active class"

In the assembly-check UI, **the currently detected class is overlaid in red text on top of the left-hand video** (e.g. `0=外观检查-顶端`, `2=装白色堵头`); the right-hand column is a static action list, so do not be misled by it. Read the overlaid class for each frame from the contact sheet and you get the action cycle.

### 3.4 Pick a "clean representative frame" for each class

- assembly-check runs a stable cycle: **0 top-surface visual → 1 copper-body visual → 2 plug insertion → 4 assembly complete**.
- For each class, pick a frame where the **overlay label is crisp and the image is not mid-transition**.
- Measured representative frames: t=9 s (0), t=14 s (1), t=22 s (2), t=5 s / 24 s (4).

### 3.5 Extract the full-resolution key frame with `-ss`

```
ffmpeg -y -ss <seconds> -i <video> -frames:v 1 out\01_外观检查_顶端.jpg
```

`-frames:v 1` takes exactly one frame; name the files semantically.

### 3.6 Verify every frame (read it back and confirm the overlaid class)

After extraction, read each file to confirm the red overlay really is the intended class. If not, pick another candidate. When the action cycle is fast, scan densely — extract a batch such as `@(3,4,5,6,23,…)` and build a small contact sheet to choose in one pass.

### 3.7 Numbering, captions, upload and embed

- Numbered names: `01_外观检查_顶端` / `02_外观检查_铜体` / `03_装白色堵头` / `04_白色堵头装配完成`.
- Embed in the article page as a 2×2 card grid titled "On-site screenshots · four assembly stages", each card with one line of description.
- Push into the GitHub repository's `images/` so they go live with the page.

## 4. Toolchain (verified on this machine)

| Purpose | Command / path |
|---|---|
| ffmpeg | `ffmpeg` (any recent build) |
| Split frames | `ffmpeg -i <v> -vf "fps=1" f_%03d.jpg` |
| Extract one frame | `ffmpeg -ss <seconds> -i <v> -frames:v 1 out.jpg` |
| Build a contact sheet | Python + PIL (use `<python-with-PIL>\python.exe`; the default `python` has no PIL) |

## 5. Pitfall log

| Pitfall | Symptom | Fix |
|---|---|---|
| Default `python` has no PIL | `ModuleNotFoundError: No module named 'PIL'` | use `<python-with-PIL>\python.exe` |
| OCR on the red video overlay is unstable | the overlay is read on one pass and missed on another, on the same frame | crop and zoom the target region, read several frames and take the intersection |
| Action cycle too fast | the single frame lands mid-transition and the class is unclear | extract densely (0.5–1 s steps) and choose from a small contact sheet in one pass |
| Some class never fires | assembly-check class 3 "placing instruction sheet" never appeared across 6,701 frames | do not force an image — label it "optional integration point" in the presentation instead of misleading |
| Extracted frame isn't the expected class | `-ss 38` yields `2=` | trust per-frame verification after extraction, not the time estimated from the contact sheet |
| The UI's static action list interferes | the right-hand list is mistaken for the current class | read only the **red overlay on the video**; treat the list as reference only |

## 6. Why this is generally useful

- **Evidence-style presentation**: on-site screenshots make "the system really runs" provable — far more persuasive than text or mock-ups.
- **Reusable**: any recording that has a detection UI or logs can use the same split → locate → pick representative frame → number and caption pipeline.
- **Where it lands**: demo-video key frames, repository README images, WeChat article images, promo-clip assets, defect/action evidence archives.

> Conclusion: **the hard part of key-frame extraction is not splitting — it is choosing. Every image should correspond to an identifiable, labelable, genuinely occurring state, then be presented numbered and captioned.**
