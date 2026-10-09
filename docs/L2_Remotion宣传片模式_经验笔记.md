# L:\L\000\2 · Remotion 宣传片模式 · 经验笔记

> 来源：`L:\L\000\2`（kirin-ocr，银河麒麟 V11 + RTX 3060 OCR · 60 秒宣传片，横屏 + 竖屏双版本）
> 用途：一套「程序化生成宣传片」的可复用模板——改文案、配色、时长只动一个文件，组件内部不用改。
> 落地示例：装配检查项目已按此模式在 `L:\L\000\装配检查\_remotion\` 建了一套并渲染出横/竖 48s 成片。

---

## 一、一句话理解这套模式

> **单一复用 Promo 组件 + `theme.ts` 集中配置 + Root 双 Composition 自适应横竖屏。**

所有文案、配色、字体、时间轴、素材、署名都集中在 `theme.ts` 一个文件里；组件按 `SCENES` 时间轴用 `Sequence` 排布场景；同一个组件导出 1920×1080 与 1080×1920 两个 composition。

## 二、目录结构（folder2）

```
L:\L\000\2\
├─ src\
│  ├─ index.ts                    # Remotion 注册入口
│  ├─ Root.tsx                    # 定义两个 Composition（横 1920×1080 / 竖 1080×1920）
│  └─ compositions\kirin-ocr\
│     ├─ theme.ts                 # ★ 可编辑常量区：文案/配色/字号/时间轴/素材/署名
│     ├─ Promo.tsx                # 主组件：按 SCENES 用 Sequence 排场景
│     ├─ fonts.ts                 # 字体引用（Deng / Dengb / CascadiaMono）
│     ├─ components\
│     │  ├─ anim.ts               # 动画函数（clamp / fadeSlideUp / fadeSlideX / popIn）
│     │  ├─ Background.tsx        # 科技背景（SceneBackground，光晕+网格）
│     │  ├─ Watermark.tsx         # 全片常驻署名条（不参与淡入淡出）
│     │  └─ DemoScene.tsx         # 实机演示场景（录屏 + 包装，横竖分栏）
│     └─ scenes\
│        ├─ IntroScenes.tsx       # hook 数字冲击 + 标题
│        └─ OutroScenes.tsx       # 性能指标 + 场景收尾
├─ public\
│  ├─ assets\...                  # 实机录屏素材
│  └─ fonts\...                   # 中文字体 ttf
├─ output\                        # 渲染成片
├─ remotion.config.ts / tsconfig.json / package.json
└─ 发布文案.md                    # 发布文案模板（视频号/B站/字幕/封面/合规）
```

## 三、核心设计要点

### 1. `theme.ts` 是"唯一要改的文件"
顶部注释就写明：**改文案 / 配色 / 时长，只改这个文件，不用动组件内部。** 分成几个常量块：

| 常量 | 作用 |
|---|---|
| `VIDEO` | fps / durationInSeconds / durationInFrames |
| `BRAND` | 配色 token（背景、强调色、文字、边框…） |
| `FONTS` / `TYPE` | 字体 + 字号体系（**基于短边 1080 设计，横竖屏通用**） |
| `AUTHOR` / `VIDEO_TITLE` / `WATERMARK` | 署名与顶部常驻条文案 |
| `SCREEN` | 实机录屏素材宽高 |
| `SCENES` | 时间轴：`[{id,label,from,duration}]` |
| `CONTENT` | 各场景文案/数据（hero 数字、标题、demo 片段、指标、日志、金句） |

### 2. 时间轴用 `SCENES` + `Sequence` 驱动
`Promo.tsx` 里用 `Sequence from={at(id).from} durationInFrames={at(id).duration}` 把每个场景按时间轴排进去；demo 场景可循环渲染多个 `DemoScene`。**改时长只需改 SCENES 的 from/duration。**

### 3. `Watermark` 压在所有场景之上，全片常驻
`<AbsoluteFill>…场景…<Watermark/></AbsoluteFill>`，署名条不参与淡入淡出，每帧都在。

### 4. 同一组件自适应横竖屏
- `const isPortrait = height > width`（取自 `useVideoConfig`）
- 布局：横屏左右分栏、竖屏上下分栏，用 `isPortrait ? … : …` 切换
- 字号基于短边设计，横竖屏都成立

### 5. 实机演示用 `OffthreadVideo + staticFile`
`<OffthreadVideo src={staticFile(SCREEN.src)} startFrom={clip.startFrom} muted>`：录屏素材放 `public/assets/`，每个 demo 片段用 `startFrom`（录屏源帧号）定位，配「实机录制」红点角标 + 顶部渐隐遮掉录屏自带标题栏。

### 6. 动画函数集中在 `anim.ts`
`clamp(frame, [0,14],[0,1])`（透明度/位移插值）、`fadeSlideUp / fadeSlideX / popIn`。统一在一个文件里，改缓动不动组件。

## 四、复用实操经验（建新工程）

1. **建工程目录**：复制 folder2 的结构（src/Root + compositions/xxx）。
2. **node_modules 直接复用**（省安装）：用目录联接
   ```
   cmd /c mklink /J <新工程>\node_modules L:\L\000\2\node_modules
   ```
   Remotion CLI 4.0.534 在 `node_modules\.bin\remotion.cmd`，可跨工程调用。
3. **字体复制**：中文字体（Deng.ttf / Dengb.ttf / CascadiaMono.ttf）复制到 `public/fonts/`。
4. **录屏素材复制**：到 `public/assets/`；配 `SCREEN` 宽高。
5. **首次运行自动下载 Chrome Headless Shell**，稍等即可。

### Remotion CLI 命令（实测）
```
remotion compositions src\index.ts                          # 验证 composition 注册
remotion render src\index.ts <comp-id> <输出.mp4>           # 渲染成片
remotion still src\index.ts <comp-id> <输出.jpg> --frame=60 # 抽静帧
```
- `still` 的**输出路径是第 3 个位置参数**；**帧号用 `--frame=`**（把帧号当位置参数传不生效，会渲染出帧 0）。
- 渲染/抽帧前需 `Set-Location` 到工程目录。

## 五、踩坑记录（遇到过、要记住）

| 坑 | 现象 | 解法 |
|---|---|---|
| `remotion still` 帧号传错 | 抽出的静帧是帧 0（只见 Watermark） | 帧号必须 `--frame=`，输出路径是第 3 位置参数 |
| 首次渲染 | 自动下载 Chrome Headless Shell，耗时 | 等它完成即可 |
| 渲染在别的目录 | 报找不到 composition | `Set-Location` 到工程目录再跑 |
| 横屏成片被预览占用 | ffmpeg 写入 Permission denied | 先写临时文件再 `Move-Item -Force` 替换 |
| 视频帧号 vs 采集墙钟 | 录屏源是 30fps，采集播放速度不同 | demo 的 `startFrom` 用**录屏源帧号**，别用墙钟秒数 |
| 中文字体缺失 | 中文渲染成方块 | 把 ttf 复制进 `public/fonts/` 并在 `fonts.ts` 引用 |
| PowerShell 传含冒号路径给 subtitles 滤镜 | Option not found | 先 `Set-Location` 到字幕目录，用相对文件名 |

## 六、发布文案模板（folder2 `发布文案.md`，可直接套用）

分五节，结构很稳，做新成片照抄填空即可：

1. **微信视频号**：配文（前 3 行最关键，超 3 行折叠）+ 话题标签 + 备选标题（≤40 字）
2. **B站**：标题（15~30 字，关键词前置）+ 简介（前 3 行放核心）+ 标签（5~10 个）+ 分区建议
3. **视频内字幕文案**：字幕时间轴表（每行 15~18 字、单行停留 ≥2s、竖屏底部预留安全区）
4. **封面文案**：竖版 + 横版两套
5. **合规自查表**：绝对化用语、数据表述（标注"本机实测"）、无引流话术、平台差异、发布节奏与最佳时段

## 七、在装配检查项目的落地对照

| folder2（OCR） | 装配检查 `_remotion\` |
|---|---|
| SCENES 8 段 / 60s | SCENES 6 段 / 48s：hook→title→demo→**actions(5类动作卡片)**→metrics→outro |
| DemoScene 4 个录屏片段 | DemoScene 1 段整屏录屏 + 4 个动作点 |
| metrics：302张/秒 等 | metrics：5类 / 25FPS / 2.4M / 43% + 原始日志 |
| CONTENT.demos | 新增 ActionsScene（5 张动作卡片，类别 3 标"可选接入"） |
| 发布文案.md 模板 | 产出口播字幕脚本 + 公众号文案沿用同模板 |

> 结论：以后任何新项目要出横/竖宣传片，照 folder2 这套搭工程 → 改 `theme.ts` 文案/时间轴 → `remotion render` 出双版本 → 用 `发布文案.md` 模板出发布物料。**改的是内容，不动的是骨架。**
