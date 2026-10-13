# Remotion Promo-Clip Pattern · Engineering Notes

> English version · [中文版](L2_Remotion宣传片模式_经验笔记.md) · local paths genericised (`<…>` = placeholder)
>
> Source: `<kirin-ocr>` (Kylin V11 + RTX 3060 OCR, a 60-second promo, landscape + portrait).
> Purpose: a reusable template for **programmatically generated promo clips** — copy, colour scheme and duration are changed in a single file; the components themselves stay untouched.
> Applied example: the assembly-check project built a variant of this pattern in its local `_remotion\` working folder and rendered 48 s landscape and portrait clips.

---

## 1. The pattern in one sentence

> **A single reusable `Promo` component + centralised `theme.ts` config + two Compositions in Root for landscape and portrait.**

All copy, colours, fonts, timeline, assets and credits live in one file, `theme.ts`. The component lays scenes out along the `SCENES` timeline using `Sequence`, and the same component exports both a 1920×1080 and a 1080×1920 composition.

## 2. Directory structure (kirin-ocr)

```
<kirin-ocr>\
├─ src\
│  ├─ index.ts                    # Remotion entry point
│  ├─ Root.tsx                    # defines the two Compositions (landscape 1920×1080 / portrait 1080×1920)
│  └─ compositions\kirin-ocr\
│     ├─ theme.ts                 # ★ the editable constants: copy / colours / type scale / timeline / assets / credits
│     ├─ Promo.tsx                # main component: lays out scenes from SCENES via Sequence
│     ├─ fonts.ts                 # font references (Deng / Dengb / CascadiaMono)
│     ├─ components\
│     │  ├─ anim.ts               # animation helpers (clamp / fadeSlideUp / fadeSlideX / popIn)
│     │  ├─ Background.tsx        # tech background (SceneBackground, glow + grid)
│     │  ├─ Watermark.tsx         # credits bar, always on top for the whole clip (never fades)
│     │  └─ DemoScene.tsx         # live-demo scene (screen recording + framing, split by orientation)
│     └─ scenes\
│        ├─ IntroScenes.tsx       # numeric hook + title
│        └─ OutroScenes.tsx       # performance metrics + closing
├─ public\
│  ├─ assets\...                  # screen-recording assets
│  └─ fonts\...                   # Chinese font TTFs
├─ output\                        # rendered clips
├─ remotion.config.ts / tsconfig.json / package.json
└─ 发布文案.md                     # publishing-copy template (WeChat Channels / Bilibili / subtitles / cover / compliance)
```

## 3. Core design points

### 3.1 `theme.ts` is "the only file you edit"

Its header comment states it outright: **to change copy, colours or duration, edit this file only — never reach into the components.** It is split into a few constant blocks:

| Constant | Purpose |
|---|---|
| `VIDEO` | fps / durationInSeconds / durationInFrames |
| `BRAND` | colour tokens (background, accent, text, border…) |
| `FONTS` / `TYPE` | fonts + type scale (**designed against the 1080 short edge, valid in both orientations**) |
| `AUTHOR` / `VIDEO_TITLE` / `WATERMARK` | credits and the always-on top bar copy |
| `SCREEN` | source screen-recording dimensions |
| `SCENES` | timeline: `[{id,label,from,duration}]` |
| `CONTENT` | per-scene copy and data (hero numbers, titles, demo clips, metrics, logs, closing lines) |

### 3.2 The timeline is driven by `SCENES` + `Sequence`

In `Promo.tsx` each scene is placed with `Sequence from={at(id).from} durationInFrames={at(id).duration}`, and the demo scene can loop over several `DemoScene` renders. **Changing the duration means editing only `from` / `duration` in `SCENES`.**

### 3.3 `Watermark` sits above every scene, present for the whole clip

`<AbsoluteFill>…scenes…<Watermark/></AbsoluteFill>` — the credits bar does not take part in fades; it is present in every frame.

### 3.4 One component adapts to landscape and portrait

- `const isPortrait = height > width` (read from `useVideoConfig`)
- Layout: landscape splits left/right, portrait splits top/bottom, switched with `isPortrait ? … : …`
- Type sizes are designed against the short edge, so both orientations hold up

### 3.5 The live demo uses `OffthreadVideo` + `staticFile`

`<OffthreadVideo src={staticFile(SCREEN.src)} startFrom={clip.startFrom} muted>` — recordings live in `public/assets/`; each demo clip is located by `startFrom` (a **source frame number** in the recording), accompanied by a red "live capture" dot badge and a top fade that masks the recording's own title bar.

### 3.6 Animation helpers are centralised in `anim.ts`

`clamp(frame, [0,14],[0,1])` (opacity/offset interpolation), plus `fadeSlideUp / fadeSlideX / popIn`. Keeping them in one file means easing changes never touch the components.

## 4. Practical reuse experience (standing up a new project)

1. **Create the project directory**: copy kirin-ocr's structure (`src/Root` + `compositions/xxx`).
2. **Reuse `node_modules` directly** (skips the install) via a directory junction:
   ```
   cmd /c mklink /J <new-project>\node_modules <kirin-ocr>\node_modules
   ```
   Remotion CLI 4.0.534 lives at `node_modules\.bin\remotion.cmd` and can be invoked from another project.
3. **Copy the fonts**: the Chinese fonts (Deng.ttf / Dengb.ttf / CascadiaMono.ttf) go into `public/fonts/`.
4. **Copy the recordings** into `public/assets/`; set the `SCREEN` dimensions.
5. **The first run auto-downloads Chrome Headless Shell** — just wait it out.

### Remotion CLI commands (verified)

```
remotion compositions src\index.ts                            # verify composition registration
remotion render src\index.ts <comp-id> <output.mp4>           # render the finished clip
remotion still src\index.ts <comp-id> <output.jpg> --frame=60 # extract a still frame
```

- For `still`, the **output path is the 3rd positional argument**, and the **frame number must be passed as `--frame=`** (passing it positionally is ignored — you get frame 0).
- `Set-Location` into the project directory before rendering or extracting.

## 5. Pitfall log (things that happened — worth remembering)

| Pitfall | Symptom | Fix |
|---|---|---|
| Wrong frame argument to `remotion still` | the extracted still is frame 0 (only the Watermark is visible) | pass the frame as `--frame=`; the output path is the 3rd positional argument |
| First render | auto-downloads Chrome Headless Shell; takes a while | let it finish |
| Rendering from another directory | "composition not found" | `Set-Location` into the project directory first |
| Landscape output held open by a preview | ffmpeg fails to write with Permission denied | write to a temp file, then replace with `Move-Item -Force` |
| Video frame numbers vs. wall clock | the recording source is 30 fps and playback speed differs | for a demo's `startFrom` use the **source frame number**, never wall-clock seconds |
| Missing Chinese font | Chinese renders as tofu boxes | copy the TTFs into `public/fonts/` and reference them in `fonts.ts` |
| PowerShell passes a colon-containing path to the subtitles filter | "Option not found" | `Set-Location` into the subtitle directory and use a relative filename |

## 6. Publishing-copy template (kirin-ocr's `发布文案.md`, reusable as-is)

Five sections, a very stable structure — copy it and fill in the blanks for the next clip:

1. **WeChat Channels**: caption (the first 3 lines matter most — beyond 3 lines it collapses) + topic hashtags + alternative titles (≤40 characters).
2. **Bilibili**: title (15–30 characters, keyword first) + description (core message in the first 3 lines) + tags (5–10) + suggested category.
3. **In-video subtitle copy**: a subtitle timeline table (15–18 characters per line, each line on screen for ≥2 s, leaving a bottom safe area for portrait).
4. **Cover copy**: portrait + landscape sets.
5. **Compliance checklist**: absolute claims, how figures are stated (label them "measured on our own machine"), no lead-generation phrasing, platform differences, publishing cadence and best time slots.

## 7. How it landed in the assembly-check project

| kirin-ocr (OCR) | assembly-check `_remotion\` |
|---|---|
| SCENES: 8 segments / 60 s | SCENES: 6 segments / 48 s — hook → title → demo → **actions (5 class cards)** → metrics → outro |
| DemoScene: 4 recording clips | DemoScene: 1 full-screen recording + 4 action marks |
| metrics: 302 images/s, etc. | metrics: 5 classes / 25 FPS / 2.4 M / 43 % + raw logs |
| CONTENT.demos | added `ActionsScene` (5 action cards, class 3 marked "optional integration point") |
| `发布文案.md` template | produced a voice-over/subtitle script + WeChat article copy on the same template |

> Conclusion: for any future project that needs landscape/portrait promo clips — stand the project up from kirin-ocr, edit copy and timeline in `theme.ts`, `remotion render` both versions, then produce the publishing assets from the `发布文案.md` template. **Content changes; the skeleton doesn't.**
