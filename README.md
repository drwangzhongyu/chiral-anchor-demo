<div align="center">

# chiral-anchor-demo

**空间坐标系锚定 · 杯 2D 转 3D 手性校验 · 缪刺法坐标系校准**

用三维几何解释「手性」：旋转不改变手性，镜像翻转手性；并以此推演中医缪刺法「左病右取、右病左取」的左右映射机理。

An interactive Three.js demo about **chirality**: rotation preserves chirality, mirroring reverses it — used to derive the left/right mapping of the TCM *Miao acupuncture* principle (treat the left for a right-side disorder and vice versa).

</div>

---

## 目录 · Table of Contents

- [中文](#中文)
  - [项目简介](#项目简介)
  - [运行方式](#运行方式)
  - [页面说明](#页面说明)
  - [核心原理](#核心原理)
  - [目录结构](#目录结构)
  - [技术栈](#技术栈)
- [English](#english)
  - [Overview](#overview)
  - [Getting Started](#getting-started)
  - [Pages](#pages)
  - [Core Principles](#core-principles)
  - [Project Structure](#project-structure)
  - [Tech Stack](#tech-stack)

---

# 中文

## 项目简介

本仓库用可交互的三维可视化解释两个相互关联的问题：

1. **手性（Chirality）是什么** —— 一个物体与它的镜像无法通过旋转重合时，就说它具有手性。绕任意轴旋转不改变手性，而沿某一个轴做镜像翻转则会反转手性。
2. **如何用坐标系的奇偶性判断手性** —— 同时翻转偶数个坐标轴，等价于一次旋转（手性保持）；翻转奇数个坐标轴，等价于一次镜像（手性反转）。
3. **缪刺法为什么是「左病右取」** —— 把人体气机循环看作一个有方向的本体循环，医者观察体位不同会造成左右视角翻转，从而决定是「同侧取穴」还是「交叉取穴（缪刺）」。

## 运行方式

页面是纯静态 HTML，但使用了 ES Module + `importmap` 从 CDN 加载 Three.js，请通过本地 HTTP 服务器打开（直接双击 `file://` 可能被浏览器的模块加载策略拦截）：

```bash
# 在仓库根目录
python3 -m http.server 8000
# 然后浏览器访问
# http://localhost:8000/chirality_demo.html
```

> 需要联网以从 `unpkg.com` 加载 Three.js 0.160.0。

## 页面说明

### 1. `chirality_demo.html` —— 合并版（推荐入口）

顶部标签栏切换两个页面：

| 标签 | 内容 |
| --- | --- |
| **三维重建 · 奇偶校验** | 杯子 2D→3D 重建的旋转 / 镜像与手性奇偶校验 |
| **缪刺法 · 坐标系校准** | 医患体位、气机循环与取穴侧的三维映射 |

### 2. `cup_chirality_check.html` —— 独立杯子页

只包含「杯子三维重建奇偶校验」内容，便于单独演示手性概念。页面左下角图例标注模型的局部坐标系：**X 轴（杯子左侧，红）· Y 轴（杯子上方，绿）· Z 轴（可见面 / 穿出屏幕，蓝）**。

- **绕轴旋转（不改变手性）**：X / Y / Z 三个滑块，范围 0–360°。
- **轴镜像翻转（改变手性）**：X 翻转 / Y 翻转 / Z 翻转三个按钮。
- **奇偶校验结果**：实时显示 PASS / ALARM 与已翻转轴数。
- **模拟重建**：随机生成一个重建姿态并自动校验；另可重置为规范姿态。

### 3. `miaoci_coordinate_calibration.html` —— 独立缪刺页

只包含「缪刺法坐标系校准」内容。三种演示体位：

- **面对面（视角相反）**：医生与患者相对而立。
- **面对背（同向）**：医生与患者同向，观察同一侧。
- **旋转演示（视角翻转）**：患者绕 Y 轴从「背对医生」匀速旋转到「面对医生」，实时展示医者视角下循环方向的翻转。

底部可点击 **左升通路障碍 / 右降通路障碍 / 重置**，触发病机、症状与针刺侧的三维映射推演。

## 核心原理

### 一、手性的奇偶校验

对一个 3D 模型同时施加若干次坐标轴镜像（`scale = -1`）：

- 翻转轴数为**偶数** → 等价于一次旋转 → **手性保持**（PASS）。
- 翻转轴数为**奇数** → 等价于一次镜像 → **手性反转**（ALARM）。

页面中的 `flippedCount()` 统计翻转轴数，`updateParity()` 据此切换 PASS / ALARM 状态。因此「连续翻转两次 X 轴」会回到原点状态，而「翻转 X 轴一次」会让杯子变成它的镜像（例如杯把从左侧跳到右侧、浮雕文字左右颠倒）。

### 二、缪刺法的坐标映射

把患者本体的气机循环记为**「左升右降」**（从其自身视角为顺时针）。医者观察时，因体位不同，左右视角可能发生翻转：

| 体位 | X 轴（左右） | Z 轴（前后） | 翻转轴数 | 手性 | 结果 |
| --- | --- | --- | --- | --- | --- |
| 面对面（视角相反） | 翻转 | 翻转 | **2（偶数）** | 不变 | 视角相反 → 症状出现在病机**对侧** → **交叉取穴（缪刺）** |
| 面对背（同向） | 不变 | 不变 | **0（偶数）** | 不变 | 坐标系重合 → 症状与病机**同侧** → **同侧取穴** |

- **面对面**：病机在患者左侧时，医者（视角相反）观测到的信号落在其自身视角的另一侧，交叉映射后症状呈现于患者对侧，于是「左病右取、右病左取」。
- **面对背**：医者与患者同向，观测与本体循环同向，症状与病机同侧，故「同侧取穴」。

> 关键点：无论体位如何，患者**本体循环方向始终不变**（左升右降）；改变的只是医者**观测视角**下左右轴是否翻转。缪刺的左右交叉，本质是「偶数次轴翻转 → 手性不变 → 只是视角反转」的几何结果。

### 三、气机循环的可视化

- 每个角色（患者 / 医生）胸前放置一个环形粒子圈，粒子沿环**顺时针匀速**流动，代表气机循环的方向。
- **红色圈** = 患者本体循环；**蓝色圈** = 医生观测循环。医生的圈独立于医生自身的缩放，始终与患者圈等大，便于对比。
- 场景中的标记：**红球** = 病机（本体循环障碍点）、**蓝球** = 症状 / 观测点、**绿环** = 针刺侧；**橙色虚线** = 病机→观测，**绿色虚线** = 观测→症状。

## 目录结构

```
chiral-anchor-demo/
├── chirality_demo.html                 # 合并版：双标签页（推荐入口）
├── cup_chirality_check.html            # 独立：杯子三维重建奇偶校验
├── miaoci_coordinate_calibration.html  # 独立：缪刺法 3D 坐标系校准
├── README.md
└── .gitignore
```

## 技术栈

- [Three.js](https://threejs.org/) `0.160.0`（通过 `unpkg` CDN + `importmap` 以 ES Module 方式引入）
- `OrbitControls`（拖拽旋转 / 缩放视角）
- 原生 HTML / CSS / JavaScript，无构建步骤、无第三方依赖安装

---

# English

## Overview

This repository uses interactive 3D visualizations to explain two connected ideas:

1. **What chirality is** — an object is *chiral* when it cannot be superimposed on its mirror image by rotation. Rotating around any axis preserves chirality; mirroring along one axis reverses it.
2. **How axis parity determines chirality** — flipping an even number of coordinate axes is equivalent to a rotation (chirality preserved); flipping an odd number is equivalent to a mirror (chirality reversed).
3. **Why Miao acupuncture treats "left for right"** — treat the body's *qi* circulation as an oriented intrinsic cycle. Different doctor–patient orientations flip the left/right viewpoint, deciding between *same-side* and *cross-side (Miao)* needling.

## Getting Started

The pages are plain static HTML, but they load Three.js from a CDN as an ES Module via `importmap`. Serve them over a local HTTP server (opening via `file://` may be blocked by browser module policies):

```bash
# from the repository root
python3 -m http.server 8000
# then open in a browser
# http://localhost:8000/chirality_demo.html
```

> An internet connection is required to load Three.js 0.160.0 from `unpkg.com`.

## Pages

### 1. `chirality_demo.html` — merged (recommended entry)

A tab bar at the top switches between two pages:

| Tab | Content |
| --- | --- |
| **3D Reconstruction · Parity Check** | Cup 2D→3D rotation / mirroring and chirality parity check |
| **Miao Acupuncture · Coordinate Calibration** | 3D mapping of doctor–patient orientation, qi circulation and needling side |

### 2. `cup_chirality_check.html` — standalone cup page

Contains only the cup reconstruction parity check for focused demos. The on-screen legend marks the model's local coordinate system: **X (cup's left, red) · Y (up, green) · Z (facing viewer, blue)**.

- **Rotation (chirality-preserving)**: X / Y / Z sliders, 0–360°.
- **Axis mirror flip (chirality-reversing)**: X / Y / Z flip buttons.
- **Parity result**: live PASS / ALARM status and the count of flipped axes.
- **Simulated reconstruction**: generate a random pose and auto-verify, or reset to the canonical pose.

### 3. `miaoci_coordinate_calibration.html` — standalone Miao page

Contains only the Miao acupuncture coordinate calibration. Three demo orientations:

- **Face-to-face (opposite viewpoint)**: doctor and patient stand facing each other.
- **Face-to-back (same direction)**: doctor and patient face the same way.
- **Rotation demo (viewpoint flip)**: the patient rotates around the Y axis from "back to the doctor" to "facing the doctor", showing the direction flip in the doctor's viewpoint.

Buttons **Left ascending-path obstruction / Right descending-path obstruction / Reset** trigger the 3D mapping of pathogenesis, symptom and needling side.

## Core Principles

### 1. Parity check of chirality

Apply several axis mirrors (`scale = -1`) to a 3D model:

- **Even** number of flipped axes → equivalent to a rotation → **chirality preserved** (PASS).
- **Odd** number of flipped axes → equivalent to a mirror → **chirality reversed** (ALARM).

In code, `flippedCount()` tallies the flipped axes and `updateParity()` switches between PASS / ALARM. Flipping X twice returns to the original state; flipping X once turns the cup into its mirror (e.g. the handle jumps to the other side and the relief text is mirrored).

### 2. Coordinate mapping of Miao acupuncture

Take the patient's **intrinsic** qi circulation as "left ascends, right descends" (clockwise from the patient's own viewpoint). Depending on the doctor–patient orientation, the left/right viewpoint may flip:

| Orientation | X axis (left/right) | Z axis (front/back) | Flipped axes | Chirality | Result |
| --- | --- | --- | --- | --- | --- |
| Face-to-face (opposite) | Flipped | Flipped | **2 (even)** | Preserved | Opposite viewpoint → symptom appears on the **opposite** side of the pathogenesis → **cross-side needling (Miao)** |
| Face-to-back (same direction) | Same | Same | **0 (even)** | Preserved | Coordinate systems coincide → symptom on the **same** side as the pathogenesis → **same-side needling** |

- **Face-to-face**: with the pathogenesis on the patient's left, the doctor's (opposite) viewpoint places the observed signal on the other side; after cross-mapping the symptom appears on the patient's opposite side — hence "treat left for right, right for left".
- **Face-to-back**: doctor and patient share the same direction, so symptom and pathogenesis fall on the same side — hence "same-side needling".

> Key insight: regardless of orientation, the patient's **intrinsic** circulation never changes (left ascends, right descends). Only the doctor's **viewpoint** may or may not flip the left/right axis. The left–right crossing of Miao acupuncture is simply the geometric consequence of "even axis flips → chirality preserved → viewpoint reversal only".

### 3. Visualizing qi circulation

- Each figure (patient / doctor) carries a ring of particles at the chest; particles flow **clockwise at a constant speed**, representing the direction of qi circulation.
- **Red ring** = patient's intrinsic cycle; **blue ring** = doctor's observed cycle. The doctor's ring is independent of the doctor's scale and always matches the patient's ring in size for easy comparison.
- Scene markers: **red sphere** = pathogenesis (intrinsic cycle obstruction), **blue sphere** = symptom / observation, **green ring** = needling side; **orange dashed line** = pathogenesis → observation, **green dashed line** = observation → symptom.

## Project Structure

```
chiral-anchor-demo/
├── chirality_demo.html                 # merged: two tabs (recommended entry)
├── cup_chirality_check.html            # standalone: cup reconstruction parity check
├── miaoci_coordinate_calibration.html  # standalone: Miao acupuncture coordinate calibration
├── README.md
└── .gitignore
```

## Tech Stack

- [Three.js](https://threejs.org/) `0.160.0` (loaded as an ES Module from the `unpkg` CDN via `importmap`)
- `OrbitControls` (drag to rotate / zoom the view)
- Vanilla HTML / CSS / JavaScript — no build step, no dependencies to install
