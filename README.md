# 凝逸Ai · 装配检查系统 YOLOv26n

装配检查的下一站：从「检出一个动作」到「维护一个装配工序状态」

[English README](README.en.md) · [英文文章页](https://nyav.github.io/assembly-check/index.en.html) · [中文文章页](https://nyav.github.io/assembly-check/)

## 简介

凝逸Ai_装配检查_YOLOv26n 是一套面向工业装配流水线的实时视觉检测方案。基于 **YOLOv26n** 端到端目标检测模型，对装配工位 7×24 在线巡检，自动识别工件外观状态与装配动作，实时判断「白色堵头是否装配到位、说明书是否放入」，将人工目检升级为可追溯的机器判定。

## 检测类别（5 类）

| 类别 | 名称 | 环节 |
| --- | --- | --- |
| 0 | 外观检查-顶端 | 检查 |
| 1 | 外观检查-铜体 | 检查 |
| 2 | 安装白色堵头 | 装配 |
| 3 | 放置说明书 | 装配 |
| 4 | 白色堵头装配完成 | 确认 |

## 技术参数

- 模型：YOLOv26n（端到端无 NMS · 参数量约 2.4 M · CPU 推理提速 43%）
- 检测类别：5 类
- 帧率：25 FPS
- 分辨率：1920 × 1080
- 处理帧数：6701 帧

## 页面

- 中文文章页 [index.html](index.html)（已部署 GitHub Pages）：https://nyav.github.io/assembly-check/
- 英文文章页 [index.en.html](index.en.html)（已部署）：https://nyav.github.io/assembly-check/index.en.html

## 经验文档

中英双语，各行末尾为英文版链接：

- [L2 Remotion 宣传片模式 · 经验笔记](docs/L2_Remotion宣传片模式_经验笔记.md) —— 程序化生成横/竖宣传片（单一 Promo 组件 + theme.ts 集中配置 + 双 Composition）· [EN](docs/remotion-promo-pattern.en.md)
- [ocr-kylin 拆视频取关键帧 · 经验笔记](docs/ocr-kylin_拆视频取关键帧_经验笔记.md) —— 拆帧定位各类动作、抽实机截图并编号配图 · [EN](docs/keyframe-extraction.en.md)
- [Remotion 宣传片 · 从零新建工程步骤清单](docs/Remotion宣传片_从零新建工程_步骤清单.md) —— 从零搭新工程并渲染成片的逐步清单 · [EN](docs/remotion-project-setup-checklist.en.md)

## 实机运行截图

「外观检查-顶端 / 铜体 / 装白色堵头 / 装配完成」四类实机关键帧见 [images/](images/)（已嵌入文章页）。

## 相关公开材料

- GPU OCR · 国产银河麒麟 V11 + RTX3060 实测（C++ 部署，14 线程 × 100 次 = 1400 张 / 0 失败 / 峰值 302 张每秒）：https://www.bilibili.com/video/BV16EpW6oEdd/
- 铁路货车车号识别 + 车体表面缺陷检测：8192×2048 线阵小目标召回 98.5%，雨雪逆光 OCR 97%+

## 合作与联系

独立计算机视觉工程师 · 8 年经验 · 20+ 商业项目全程远程交付（UTC+8，距日本 1 小时）。

典型交付形态为独立模块：**1~2 周**从数据到部署，交付 C++ DLL/SO + HTTP 服务，完整私有化部署在你自己服务器上。**发我 50 张样本图，免费返准确率报告 + 演示视频**，零风险评估。

- 邮箱：**ccc7@foxmail.com**　微信：**lxnyai**

© 凝逸Ai · nyav
