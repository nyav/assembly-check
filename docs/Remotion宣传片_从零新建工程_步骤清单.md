# 从零新建 Remotion 宣传片工程 · 步骤清单

> 目标：照 `L:\L\000\2`（kirin-ocr）模式，从零搭一个新项目并渲染横/竖宣传片。
> 前置：`L:\L\000\2` 的 node_modules 可作复用源；本机有 ffmpeg 与"沙箱 python(带 PIL)"。
> 原则：**改的是内容，不动的是骨架**——所有文案/配色/时长集中在 `theme.ts`。

---

## 0. 准备

- [ ] 确认本机：ffmpeg（`C:\Program Files (x86)\FormatFactory\ffmpeg.exe`）
- [ ] 确认带 PIL 的 python：`C:\Users\Administrator\AppData\Local\Doubao\User Data\sandbox_runtime\bases\…\python\python.exe`
- [ ] 确认素材：录屏视频、封面图、字体（Deng.ttf / Dengb.ttf / CascadiaMono.ttf，可自 `L:\L\000\2\public\fonts\` 复制）

## 1. 建目录骨架

```
<新工程>\
├─ src\
│  ├─ index.ts                    # Remotion 注册入口
│  ├─ Root.tsx                    # 两个 Composition（横 1920×1080 / 竖 1080×1920）
│  └─ compositions\<名字>\
│     ├─ theme.ts                 # ★ 唯一要改的文件
│     ├─ Promo.tsx
│     ├─ fonts.ts
│     ├─ components\{anim.ts, Background.tsx, Watermark.tsx, DemoScene.tsx, ...}
│     └─ scenes\{IntroScenes.tsx, OutroScenes.tsx, ...}
├─ public\
│  ├─ assets\...                  # 录屏素材
│  └─ fonts\...                   # 中文字体
├─ output\
├─ package.json / tsconfig.json / remotion.config.ts
```

## 2. 复用 node_modules（省安装）

```
cmd /c mklink /J <新工程>\node_modules L:\L\000\2\node_modules
```

## 3. 拷素材与字体

- [ ] 录屏 → `public\assets\`
- [ ] 字体 → `public\fonts\`（Deng.ttf / Dengb.ttf / CascadiaMono.ttf）

## 4. 写配置文件

- [ ] `package.json`：`"dependencies": { "remotion": "^4.0.534", "react", "react-dom" }`，`"scripts": { "render": "remotion render src/index.ts", "still": "remotion still src/index.ts" }`
- [ ] `tsconfig.json`：`"jsx": "react-jsx"`、`"moduleResolution": "node"`
- [ ] `remotion.config.ts`：`Config.setVideoImageFormat("jpeg")` / 分辨率设置（按需）

## 5. 写 src（照 folder2 抄骨架，改名字）

- [ ] `fonts.ts`：引用 `public/fonts/` 三个字体（staticFile + loadFont）
- [ ] `anim.ts`：`clamp / fadeSlideUp / fadeSlideX / popIn`
- [ ] `Background.tsx`：SceneBackground（光晕 + 网格）
- [ ] `Watermark.tsx`：全片常驻署名条
- [ ] `Promo.tsx`：`<AbsoluteFill>` + 按 SCENES 的 `Sequence` 排场景 + `<Watermark/>` 压底
- [ ] `scenes/IntroScenes.tsx`：hook 数字冲击 + 标题
- [ ] `scenes/OutroScenes.tsx`：指标 + 收尾金句
- [ ] `components/DemoScene.tsx`：录屏 + 包装（横竖分栏）
- [ ] `Root.tsx`：横 1920×1080 + 竖 1080×1920 两个 Composition
- [ ] `index.ts`：注册 `RemotionRoot`

## 6. 填 theme.ts（核心工作）

- [ ] `VIDEO`：fps / 秒数 / durationInFrames
- [ ] `BRAND`：配色 token（沿用深海军蓝 #070B14 + 青 #22D3EE 科技风）
- [ ] `TYPE`：字号基于短边 1080 设计
- [ ] `AUTHOR / VIDEO_TITLE / WATERMARK`：署名文案
- [ ] `SCREEN`：录屏宽高
- [ ] `SCENES`：时间轴 `[{id,label,from,duration}]`
- [ ] `CONTENT`：各场景文案/数据（hero 数字、标题、demo、指标、日志、金句）

## 7. 验证

```
remotion compositions src\index.ts
```
- [ ] 两个 composition 都注册成功（首次运行自动下载 Chrome Headless Shell，稍等）

## 8. 渲染

```
Set-Location <新工程>
node_modules\.bin\remotion.cmd render src\index.ts <comp-landscape> output\横屏.mp4
node_modules\.bin\remotion.cmd render src\index.ts <comp-portrait>  output\竖屏.mp4
```

## 9. 抽帧验证

```
node_modules\.bin\remotion.cmd still src\index.ts <comp> out.jpg --frame=<帧号>
```
- [ ] 每个场景抽一帧核对（hook/标题/demo/指标/收尾）
- [ ] 注意：`still` 输出路径是第 3 位置参数，帧号用 `--frame=`

## 10. 配套物料

- [ ] 封面（竖版 9:16 + 横版 16:9）
- [ ] 拆视频取关键帧（如需实机截图，见 `ocr-kylin_拆视频取关键帧_经验笔记.md`）
- [ ] 发布文案（视频号/B站/字幕/封面/合规，见 `L2_Remotion宣传片模式_经验笔记.md` 第六节模板）

## 快速踩坑备忘

| 坑 | 解法 |
|---|---|
| `still` 抽到帧 0 | 帧号用 `--frame=` |
| 渲染找不到 composition | `Set-Location` 到工程目录 |
| 中文变方块 | 字体进 `public/fonts/` 并在 fonts.ts 引用 |
| 成片被占用写不进 | 先写临时文件再 Move-Item -Force |
| 默认 python 无 PIL | 用 sandbox python |
