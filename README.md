# PLC 电气自动化工程师 · 求职作品集

> **机器人工程专业（2027 届）× PLC 自动化方向**
> 目标岗位：PLC 工程师 / 电气自动化工程师 / 设备工程师
> 一句话定位：**懂工业机器人 IO 握手逻辑的 PLC 工程师** —— 不只会写梯形图，能做产线集成、上位机通信与视觉联动的复合型候选人。

---

## 一、项目组合（开发中）

| # | 项目 | 定位 | 状态 | 核心技术栈 |
|---|---|---|---|---|
| 01 | [S7-1200 + FactoryIO 三工位分拣产线虚拟调试](projects/01-plc-sorting-line/) | 简历主项目：完整产线逻辑 + HMI + 产量统计 | 🚧 开发中 | TIA Portal V18、LAD/SCL、S7-PLCSIM、FactoryIO、WinCC Basic |
| 02 | [PLC + 六轴机械臂取放联动工位](projects/02-robot-cell/) | **机器人专业核心差异化**：PLC↔机器人 IO 握手时序 | 🚧 开发中 | S7-1200、FactoryIO Robot Arm、SCL 步序（寒假升级 URSim + Modbus TCP） |
| 03 | [C# 上位机 Modbus TCP 数据采集与监控](projects/03-modbus-scada/) | 数据上行：PLC → 上位机 → SQLite → 趋势曲线 | 🚧 开发中 | C# WinForms、NModbus4/S7.NET、SQLite、LiveCharts2 |
| 04 | [OpenCV 视觉检测 + PLC 分拣联动](projects/04-vision-sorting/) | 视觉 + 运动控制跨界 | 🚧 开发中 | Python、OpenCV、pyModbusTCP |
| 05 | [威纶通 HMI 组态（报警/配方/趋势）](projects/05-weintek-hmi/) | 设备岗标配人机交互 | 🚧 开发中 | EasyBuilder Pro |

> 每个项目目录内含：技术方案、IO 地址表、步序逻辑、可量化验收标准、演示录屏链接（开发完成后补齐）。

## 二、技能栈地图

| 维度 | 技能 | 熟练度 |
|---|---|---|
| PLC 品牌 | 西门子 S7-1200/1500（主线）、三菱 FX3U/FX5U（副线）、Codesys（了解） | 学习中 |
| 编程软件 | TIA Portal V18、GX Works3、FactoryIO | 学习中 |
| 编程语言 | 梯形图 LAD、结构化文本 SCL/ST、C#（WinForms）、Python（OpenCV） | 学习中 |
| 通信协议 | Modbus RTU/TCP、PROFINET、EtherCAT（了解）、S7 协议 | 学习中 |
| HMI/组态 | WinCC Basic、威纶通 EasyBuilder Pro | 学习中 |
| 电气设计 | 电气识图、EPLAN 绘图（计划） | 学习中 |
| 工业机器人 | 机器人 IO 信号握手、坐标系（基/工具/用户）、URSim（计划） | 专业背景 |

📌 详细见 [docs/skill-map.md](docs/skill-map.md)

## 三、求职路线图

- 秋招窗口：2026.9–12（投递高峰 10–11 月）
- 春招窗口：2027.2–4
- 周级里程碑排期、简历投递节奏、笔试面试准备、低压电工证考试安排
- 📌 详细见 [docs/career-plan.md](docs/career-plan.md)

## 四、学习笔记

- [西门子 TIA 指令速查](docs/study-notes/siemens-tia-notes.md)
- [三菱 FX 指令速查](docs/study-notes/mitsubishi-fx-notes.md)
- [工业通信协议笔记（Modbus/PROFINET/EtherCAT/OPC UA）](docs/study-notes/communication-protocols.md)
- [校招笔面试题库（持续更新）](docs/study-notes/interview-qa.md)

## 五、诚实声明

本仓库所有项目均基于**软件仿真**开发（S7-PLCSIM / FactoryIO / 威纶通离线仿真），开发完成前未连接实体 PLC 与机器人。所有截图、录屏、数据均来自真实运行画面，**不虚构任何演示内容**。硬件计划：11 月底视预算购入二手 S7-200 SMART 或三菱 FX3U 补充真机手感。

---

*更新日志：2026-09-29 仓库初始化，规划 5 个项目组合，启动前两周速成学习路径。*