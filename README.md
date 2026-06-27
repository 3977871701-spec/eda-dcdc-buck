# DCDC Buck Converter — Codex AI 辅助 PCB 设计

基于嘉立创 EDA Pro (EasyEDA Pro) 的 DCDC Buck 降压转换器 PCB 设计项目，由 Codex AI Agent 通过 codex-eda-bridge 桥接服务远程操控完成原理图优化和 PCB 布局。

## 项目简介

本项目是一个完整的 AI 辅助 EDA 设计实践案例。设计目标为一款 DCDC Buck 降压转换器，包含以下核心元件：

- **U1** — Buck 控制器/转换器 IC
- **L1** — 储能电感
- **D1** — 续流二极管
- **C1** (22uF/25V, C0805) — 输入滤波电容
- **C2** (100nF, C0603) — 输入旁路电容
- **C3** (100nF BOOT, C0603) — Bootstrap 电容
- **C4** (47uF/10V, C1206) — 输出滤波电容
- **C5** — 额外滤波电容
- **R1~R6** — 反馈分压及配置电阻
- **J1, J2** — 输入/输出端子

网络拓扑包括 VIN、GND、VOUT、SW（开关节点）、BST（Bootstrap）等典型 Buck 电路信号。

## 版本迭代说明

项目经历了 4 个设计版本的迭代优化，所有 `.epro` 文件均为嘉立创 EDA Pro 工程文件：

### v1 — 修复版 (`DCDC_Buck_Codex_Fixed_20260618.epro`)

初始修复版本。AI 对原始设计进行了基础属性检查和修正，确保元件属性（Designator、Footprint、Supplier Part 等）完整性，为后续优化奠定基础。

### v2 — 优化版 (`DCDC_Buck_Codex_Optimized_20260618.epro`)

在修复版基础上进行第一轮优化。AI 根据 PCB 设计规范调整了元件标注的显示样式，优化了丝印层文本的可读性和布局合理性。

### v3 — 优化版 v2 (`DCDC_Buck_Codex_Optimized_v2_20260618.epro`)

第二轮优化版本，进一步细化设计参数，调整元件位置和铜皮策略。

### v4 — 实时适配版 (`DCDC_Buck_Codex_Optimized_v3_livefit_20260618.epro`)

最终版本，通过 live_pcb_backup 实时监控 + AI 分析的闭环迭代完成。此版本完成了两项关键 AI 操作：

1. **fix_attrs** — 批量修正所有元件的 Designator 文本属性
2. **create_copper** — 创建顶层和底层 GND 铜皮覆铜区域

## AI 辅助功能详解

本项目通过 `codex-eda-bridge` 桥接服务实现 AI Agent 对 EDA 设计的远程操控。以下是三个核心 AI 辅助功能：

### 1. live_pcb_backup — PCB 实时备份

AI Agent 在设计过程中周期性地通过 Bridge 拉取 PCB 全量数据并保存为 JSON 快照，用于：
- 记录设计演变过程，便于回溯和对比
- 为 AI 分析提供当前设计状态的完整数据
- 防止意外操作导致的设计丢失

本项目的 5 个实时备份文件记录了从 `21:32` 到 `21:53` 约 20 分钟内的设计变化：

| 备份文件 | 时间 | 主要状态 |
|----------|------|----------|
| `live_pcb_backup_20260618_213247.json` | 21:32 | 初始状态，Designator fontSize=4.5 |
| `live_pcb_backup_20260618_213838.json` | 21:38 | 同上，无变化 |
| `live_pcb_backup_20260618_214621.json` | 21:46 | 同上，无变化 |
| `live_pcb_backup_20260618_215157.json` | 21:51 | fix_attrs 执行后，fontSize=1.05 |
| `live_pcb_backup_20260618_215344.json` | 21:53 | 最终状态确认 |

每个备份包含所有元件的完整信息：封装、位置、旋转角度、焊盘网络、属性映射（attrsMap）和丝印属性（ATTR）等。

### 2. fix_attrs — 元件属性批量修复

AI 通过分析 PCB 备份数据，发现所有元件的 Designator（位号标注）在丝印层的显示参数不合理，于是批量执行修复操作。

**修复内容（`fix_attrs_result_20260618_215251.json`）：**

对全部 16 个元件（C1~C5, D1, J1, J2, L1, R1~R6, U1）的 Designator ATTR 进行了统一修正：

| 属性 | 修改前 | 修改后 | 说明 |
|------|--------|--------|------|
| `fontSize` | 4.5 | 1.05 | 缩小字号，适配 PCB 丝印尺寸 |
| `rotation` | 3.14~15.7 (各种角度) | 0 | 统一为水平方向，提高可读性 |
| `strokeWidth` | 0.6 | 0.06 | 线宽按比例缩小 |
| `position` | 各种位置 | 重新计算 | 调整到元件附近合理位置 |

该操作确保了 PCB 丝印层的位号标注清晰、规范、不与焊盘或走线冲突。

### 3. create_copper — 铜皮覆铜区域创建

AI 在 PCB 设计完成后，自动创建了完整的 GND 覆铜区域：

**创建内容（`create_copper_result_20260618_215119.json`）：**

| 层 | 名称 | 网络 | 覆铜范围 | 优先级 |
|------|------|------|----------|--------|
| 顶层 (layerId=1) | GND_TOP_POUR | GND | 整板区域 (9,25)-(112,70) | 2 |
| 底层 (layerId=2) | GND_BOTTOM_POUR | GND | 整板区域 (9,25)-(112,70) | 2 |

覆铜参数：
- 填充样式：solid（实心填充）
- 线宽：0.2mm
- 安全间距：2mm
- 圆角半径：2.8mm
- 保留孤岛：是
- 优化值：0.8

## 技术栈

- **EDA 工具** — 嘉立创 EDA Pro (EasyEDA Pro)，国产云端 PCB 设计平台
- **AI Agent** — Codex CLI，通过自然语言驱动 EDA 操作
- **桥接服务** — codex-eda-bridge，基于 WebSocket 的 AI-to-EDA 通信通道
- **电路拓扑** — DCDC Buck 降压转换器，输入 12~24V，输出可调
- **元件库** — 嘉立创/LCSC 元件库，含 3D 模型和供应商信息

## 项目文件

```
EDA_DCDC_20260621/
├── DCDC_Buck_Codex_Fixed_20260618.epro           # v1 修复版
├── DCDC_Buck_Codex_Optimized_20260618.epro       # v2 优化版
├── DCDC_Buck_Codex_Optimized_v2_20260618.epro    # v3 优化版 v2
├── DCDC_Buck_Codex_Optimized_v3_livefit_20260618.epro  # v4 实时适配版（最终版）
├── README.md                                      # 本文件
└── codex_eda_backups/                             # AI 操作记录
    ├── live_pcb_backup_20260618_213247.json       # 实时备份 #1
    ├── live_pcb_backup_20260618_213838.json       # 实时备份 #2
    ├── live_pcb_backup_20260618_214621.json       # 实时备份 #3
    ├── live_pcb_backup_20260618_215157.json       # 实时备份 #4（fix_attrs 后）
    ├── live_pcb_backup_20260618_215344.json       # 实时备份 #5（最终状态）
    ├── fix_attrs_result_20260618_215251.json      # fix_attrs 操作结果
    ├── create_copper_result_20260618_215119.json  # create_copper 操作结果
    ├── DCDC_Buck_2026-06-18T11-57-34-045Z.esch   # 原理图快照
    └── DCDC_Buck_2026-06-18T11-57-34-045Z.epcb   # PCB 快照
```

## 使用方法

### 前置条件

1. 安装嘉立创 EDA Pro（桌面版或网页版）
2. 部署 codex-eda-bridge 桥接服务（参见 `~/codex-eda-bridge/README.md`）
3. 安装 Codex CLI

### 打开设计文件

使用嘉立创 EDA Pro 打开任意 `.epro` 文件即可查看对应版本的设计。推荐打开最终版本 `DCDC_Buck_Codex_Optimized_v3_livefit_20260618.epro`。

### 查看 AI 操作记录

`codex_eda_backups/` 目录下的 JSON 文件记录了 AI 的完整操作过程，可用于：
- 理解 AI 的设计决策逻辑
- 复现或调整 AI 的操作参数
- 作为类似项目的参考模板
