---
name: torn-paper-collage
description: 撕纸拼贴（Torn Paper Collage）风格分析与同风格图片生成。当用户提供若干参考图片（撕纸/掀开纸面露出风景或照片的超现实拼贴风格）并要求"总结风格特点"、"生成类似/同风格图片"、"风格克隆"时使用本技能。触发词：撕纸拼贴、风格克隆、同风格图片、torn paper collage、撕开纸面、纸上藏着一个世界。
---

# Torn Paper Collage — 撕纸拼贴风格克隆

## 概述

本技能将用户提供的撕纸拼贴参考图提炼为结构化风格分析，再通过 ImageGen 把指定主体图（或纯文字描述的主体）合成为同风格图片。核心风格特征：纯色哑光纸面被撕开，破洞后露出真实的摄影风景/照片，形成超现实的"纸上藏着另一个世界"效果。

## 工作流程

### 第一步：区分素材类型

用户输入通常包含两类图片，必须先区分：

- **参考风格图**（1-N 张）：用于提炼撕纸拼贴的风格要素。
- **主体图**（0-1 张）：新图片中"洞后风景/照片"的内容来源。用户未指明时，取"内容与参考图风格无关、适合作为洞内风景"的那张；判断有歧义时向用户确认，不要猜。
- 若无主体图，从用户文字描述中提取洞内场景。

### 第二步：风格分析（先输出分析，再生成）

按六个维度用 markdown 列表向用户输出风格总结，结论将直接转化为生成 prompt：

1. **纸面基底**：纸张颜色（白/米色/牛皮纸/正红/橙/蓝等）、材质质感（哑光、带纹理的卡纸、墙皮）
2. **撕口形态**：破洞形状（纵向裂口/拱形/横向撕开/V形/异形）、边缘细节（白色纸纤维毛边、卷翘纸层、投影）
3. **洞内内容**：露出的照片类型（自然风景/大海/天空/老照片）及其色调
4. **人物互动**：小人物与撕口的关系（从洞中走出/伸手撕开/拉着纸边扯出风景/冲破纸面/无人物）
5. **色彩对比**：纸面单色与洞内鲜艳照片的"平面 vs 真实"对比关系
6. **整体气质**：极简、梦幻、诗意、孤独感等

若参考图风格存在明显不一致（如有的有人物有的没有），先指出差异并请用户指定以哪种为准，或按多数图取主导特征。

### 第三步：调用 ImageGen 生成

通过 ToolSearch 加载 ImageGen，再用 DeferExecuteTool 调用。参数策略：

- **image1**：主体图（保留洞内风景原貌）；无主体图时省略，仅用文字 prompt
- **image2**：选一张最具代表性的参考图作为风格参照
- **input_fidelity**：`high`
- **size**：`1024x1536`（竖版为主，横版主体图可改 `1536x1024`）
- **quality**：`high`
- **style**：`graphic design poster`

Prompt 必须是英文、自包含，按以下模板填入第二步的分析结论：

```
Surreal torn-paper collage artwork, vertical format.
Take the landscape from image1 ([一句话描述主体图内容]) exactly as the
photographic scene revealed inside the opening, keeping the key subjects
unchanged[, with absolutely no watermark, logo or text anywhere in the
scene — 主体图自带水印/文字时加上此句].
The whole composition is a flat matte [纸面颜色] textured paper surface
covering the entire frame, with a [撕口形状] tear ripped open through
the [位置] of the paper, revealing the photographic landscape of image1
behind it through the tear.
The torn edges are raw, jagged and organic, with visible white paper
fibers and slightly curled, lifting paper layers casting soft shadows
[+ 可选卷折句: parts of the torn edge are folded back and rolled, forming
thick rolled paper edges with accumulated folded creases, like paper
edges rolled into loose cylinders casting soft realistic shadows].
[人物互动描述，如: A single small human figure, seen from behind in muted
clothing, stands at the bottom of the tear reaching up and pulling the
torn paper edge aside with both hands, as if opening a curtain to reveal
the scene behind it. 或: No human figures, pure still composition.]
Strong contrast between the flat monochrome paper foreground and the
vivid realistic photograph inside the opening.
Minimal composition, lots of negative space on the paper, dreamlike and
poetic mood, analog cut-and-paste collage aesthetic, fine paper grain,
soft studio lighting.
```

**生成前必须告知用户：单张图片生成约消耗 5-10 积分。**

### 第四步：校验与交付

用 ViewImage 查看生成结果，逐项核对：

1. 洞内主体是否完整保留（如巴士、树木、人物等未被改变或扭曲）
2. 撕口边缘是否有纸纤维毛边和卷翘纸层，立体感是否成立
3. 纸面是否为大面积均匀纯色，无多余装饰或文字
4. 人物（若有）比例是否小而自然、与撕口互动关系正确；**肢体/爪部是否有畸形**（关节模糊、脚趾含糊、肢体拉长）
5. 洞内是否残留主体图自带的水印或乱码文字
6. 构图是否干净（无多余装饰、无明显混乱）

明显缺陷 → 修正 prompt 后重新生成一次，最多重试一次。通过后用 present_files 呈现，回复中包含：风格总结要点、生成文件路径、积分消耗提示、可调整项（换纸面颜色/撕口形状/人物互动方式）。

## 常用调整变量

用户对结果提修改意见时，只改动对应变量重新生成，不要推翻整体结构：

| 变量 | 可选值 |
|------|--------|
| 纸面颜色 | 米白、牛皮纸棕、正红、陶土橙、克莱因蓝、墨绿、沙米色（配色与洞内风景的适配关系见 references/style-cases.md 第四节） |
| 撕口形状 | 纵向长条、拱形门洞、横向裂口、V形、心形/异形、S形/蜿蜒（可呼应洞内河流等曲线元素） |
| 人物互动 | 拉开纸边、从洞中走出、伸手撕开、攀爬进入、仅下巴搭边、洞内望向纸外、无人物；也可"纸外拉边+洞内静观"双角色叙事 |
| 洞内风景 | 由主体图决定，或按用户文字描述 |

## 注意事项

- 用户用中文交流时，风格分析和最终回复用中文输出；仅 ImageGen prompt 用英文。
- 撕口边缘细节（纸纤维、卷翘层、投影）是本风格的灵魂，prompt 中不可省略；"卷折撕口"（纸边回折卷成纸筒、堆积折痕）立体感更强，优先考虑（关键句见 references/style-cases.md）。
- 洞内内容必须忠实于主体图（input_fidelity: high），不要让模型改写洞内风景。
- 人物遵循"小而少"原则：一个小人物即可，避免喧宾夺主。
- **肢体跨边风险**：避免让主体的爪/手等复杂肢体越过撕口搭在纸面上，极易生成关节畸形；改用"主体在洞内望向纸外"或"仅下巴搭边"表达互动。简单形状（细树枝、丝带等）跨边安全。
- 想增强叙事时可用"双角色"结构：纸外一个小人拉边 + 洞内主体静观，一动一静。人物手部加 "hands grip the torn edge simply without anatomical detail close-ups" 降低畸形概率。
- 撕口形状可与洞内内容呼应（如洞内有蜿蜒河流用 S 形撕口），强化画面叙事。
- 主体图自带水印/文字时，prompt 中明确要求 "absolutely no watermark, logo or text anywhere in the scene"。
- 详细案例库（7 张参考图分析 + 9 例实战验证 + 配色速查表）见 references/style-cases.md，风格分析时优先参考。
