# Standing Up a Remotion Promo Project From Scratch · Checklist

> English version · [中文版](Remotion宣传片_从零新建工程_步骤清单.md)
>
> Goal: follow the `<kirin-ocr>` template project to build a new project from scratch and render landscape/portrait promo clips.
> Prerequisites: `<kirin-ocr>`'s `node_modules` is available as a reuse source; you also need ffmpeg and a Python with PIL available.
> Principle: **content changes; the skeleton doesn't** — all copy, colours and duration live in `theme.ts`.

---

## 0. Preparation

- [ ] Confirm ffmpeg is available (`ffmpeg -version`)
- [ ] Confirm a Python with PIL: `<python-with-PIL>\python.exe`
- [ ] Confirm the assets: screen recording, cover image, fonts (Deng.ttf / Dengb.ttf / CascadiaMono.ttf — copy from `<kirin-ocr>\public\fonts\`)

## 1. Create the directory skeleton

```
<new-project>\
├─ src\
│  ├─ index.ts                    # Remotion entry point
│  ├─ Root.tsx                    # two Compositions (landscape 1920×1080 / portrait 1080×1920)
│  └─ compositions\<name>\
│     ├─ theme.ts                 # ★ the only file you edit
│     ├─ Promo.tsx
│     ├─ fonts.ts
│     ├─ components\{anim.ts, Background.tsx, Watermark.tsx, DemoScene.tsx, ...}
│     └─ scenes\{IntroScenes.tsx, OutroScenes.tsx, ...}
├─ public\
│  ├─ assets\...                  # screen-recording assets
│  └─ fonts\...                   # Chinese fonts
├─ output\
├─ package.json / tsconfig.json / remotion.config.ts
```

## 2. Reuse `node_modules` (skips the install)

```
cmd /c mklink /J <new-project>\node_modules <kirin-ocr>\node_modules
```

## 3. Copy assets and fonts

- [ ] Recording → `public\assets\`
- [ ] Fonts → `public\fonts\` (Deng.ttf / Dengb.ttf / CascadiaMono.ttf)

## 4. Write the config files

- [ ] `package.json`: `"dependencies": { "remotion": "^4.0.534", "react", "react-dom" }`, `"scripts": { "render": "remotion render src/index.ts", "still": "remotion still src/index.ts" }`
- [ ] `tsconfig.json`: `"jsx": "react-jsx"`, `"moduleResolution": "node"`
- [ ] `remotion.config.ts`: `Config.setVideoImageFormat("jpeg")` / resolution settings (as needed)

## 5. Write `src` (copy kirin-ocr's skeleton, rename things)

- [ ] `fonts.ts`: reference the three fonts in `public/fonts/` (staticFile + loadFont)
- [ ] `anim.ts`: `clamp / fadeSlideUp / fadeSlideX / popIn`
- [ ] `Background.tsx`: SceneBackground (glow + grid)
- [ ] `Watermark.tsx`: the credits bar, present for the whole clip
- [ ] `Promo.tsx`: `<AbsoluteFill>` + scenes laid out from SCENES via `Sequence` + `<Watermark/>` on top
- [ ] `scenes/IntroScenes.tsx`: numeric hook + title
- [ ] `scenes/OutroScenes.tsx`: metrics + closing line
- [ ] `components/DemoScene.tsx`: screen recording + framing (split by orientation)
- [ ] `Root.tsx`: two Compositions — landscape 1920×1080 and portrait 1080×1920
- [ ] `index.ts`: register `RemotionRoot`

## 6. Fill in `theme.ts` (the core work)

- [ ] `VIDEO`: fps / seconds / durationInFrames
- [ ] `BRAND`: colour tokens (keep the deep navy `#070B14` + cyan `#22D3EE` tech look)
- [ ] `TYPE`: type sizes designed against the 1080 short edge
- [ ] `AUTHOR / VIDEO_TITLE / WATERMARK`: credit copy
- [ ] `SCREEN`: recording dimensions
- [ ] `SCENES`: timeline `[{id,label,from,duration}]`
- [ ] `CONTENT`: per-scene copy and data (hero numbers, titles, demo, metrics, logs, closing lines)

## 7. Verify

```
remotion compositions src\index.ts
```

- [ ] Both compositions register successfully (the first run auto-downloads Chrome Headless Shell — just wait)

## 8. Render

```
Set-Location <new-project>
node_modules\.bin\remotion.cmd render src\index.ts <comp-landscape> output\landscape.mp4
node_modules\.bin\remotion.cmd render src\index.ts <comp-portrait>  output\portrait.mp4
```

## 9. Extract frames to verify

```
node_modules\.bin\remotion.cmd still src\index.ts <comp> out.jpg --frame=<frame>
```

- [ ] Extract one frame per scene and check it (hook / title / demo / metrics / outro)
- [ ] Note: for `still`, the output path is the 3rd positional argument and the frame number goes in `--frame=`

## 10. Supporting assets

- [ ] Covers (portrait 9:16 + landscape 16:9)
- [ ] Key frames from the video (if you need on-site screenshots, see `keyframe-extraction.en.md`)
- [ ] Publishing copy (WeChat Channels / Bilibili / subtitles / cover / compliance — see section 6 of `remotion-promo-pattern.en.md`)

## Quick pitfall memo

| Pitfall | Fix |
|---|---|
| `still` yields frame 0 | pass the frame as `--frame=` |
| Render can't find the composition | `Set-Location` into the project directory |
| Chinese turns into boxes | put the fonts in `public/fonts/` and reference them in `fonts.ts` |
| Output file locked / write fails | write a temp file, then `Move-Item -Force` |
| Default python has no PIL | use the sandbox python |
