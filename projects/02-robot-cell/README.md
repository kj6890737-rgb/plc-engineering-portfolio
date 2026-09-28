# 项目② PLC + 六轴机械臂取放联动工位 🚧 开发中

> 一句话定位：把机器人工程专业背景与 PLC 控制焊在一起——普通自动化学生不懂机器人 IO 握手，机器人专业学生只会用示教器，本项目证明你两边都能讲清楚。

---

## 1. 为什么这个项目最值钱

- 中车系 / 汽车零部件 / 3C 行业的"机器人集成工程师"岗位，JD 直接写"PLC + 机器人联调经验"；
- 普通候选人只会说"我会用示教器"，讲不清 Pick / Place / Busy / InPosition 的握手时序；
- 本项目要求能在面试时画出 **PLC ↔ 机器人 DI/DO 信号交互表**，并口述坐标系与协议区别。

---

## 2. 技术栈与两阶段方案

| 阶段 | 时间目标 | 软件 | 内容 |
|---|---|---|---|
| 阶段 A | 2026 年 10 月底前（秋招前出 demo） | S7-1200 PLCSIM + FactoryIO Robot Arm 对象 | PLC 通过 IO 信号控制虚拟六轴完成 Pick/Place |
| 阶段 B | 2027 年寒假升级 | URSim（UR5e/UR10e 官方免费仿真）+ MB_CLIENT | PLC 通过 Modbus TCP 读写 UR 寄存器，做完整协议握手 |

**阶段 A 技术栈：**
- PLC 侧：S7-1200（PLCSIM）+ SCL；
- 机器人侧：FactoryIO 自带 Robot Arm 对象；
- HMI：WinCC Basic，显示节拍时间。

**阶段 B 升级要点（寒假）：**
- URSim 中 Setup → Modbus Server，端口 502；
- TIA 侧调用 `MB_CLIENT` 功能块（背景 DB），连接 UR 默认 IP 192.168.0.1；
  - 写保持寄存器 40001 = 目标点位号（1=Home, 2=Pick, 3=Place）；
  - 写线圈 00001 = 触发运动；
  - 读输入寄存器 30001 = 到位状态；
- README 补一张完整的"PLC ↔ UR 信号握手时序图"。

---

## 3. PLC ↔ 机器人信号交互表（阶段 A）

| PLC 输出 → 机器人输入 | 含义 | 机器人输出 → PLC 输入 | 含义 |
|---|---|---|---|
| Q1.0 = Start | 启动一个取放循环 | I1.0 = Busy | 机器人运动中 |
| Q1.1 = Pick | 机器人移动到取料点 | I1.1 = InPosition | 到位 |
| Q1.2 = Place | 机器人移动到放料点 | I1.2 = GripperClosed | 夹爪已闭合 |
| Q1.3 = Gripper | 夹爪开合控制 | — | — |

> （开发完成后补齐：信号交互表截图 + 步序流程图）

---

## 4. SCL CASE 步序

```pascal
CASE #Step OF
  0:  IF #Start THEN #Step := 10; END_IF;          // 待机
  10: "Conveyor" := TRUE;                          // 传送带送料
  20: IF "PartAtPickPos" THEN                      // 料到位
        "Conveyor" := FALSE;
        "Pick_Cmd" := TRUE;                        // 发 Pick 命令
        #Step := 30;
      END_IF;
  30: IF "InPosition" AND "GripperClosed" THEN
        "Pick_Cmd" := FALSE;
        "Place_Cmd" := TRUE;                       // 发 Place 命令
        #Step := 40;
      END_IF;
  40: IF "PlaceDone" THEN
        "Place_Cmd" := FALSE;
        "Gripper" := FALSE;                        // 松爪
        #Step := 0;                                // 一个循环结束
      END_IF;
END_CASE;
```

---

## 5. HMI 节拍时间设计

- 在 HMI 上用 TON 测每个取放循环的实际耗时；
- **目标节拍 ≤ 8 秒 / 循环**；
- 实时显示当前节拍、平均节拍、循环次数。

> （开发完成后补齐：HMI 节拍时间显示截图）

---

## 6. 可验证完成标准（验收清单）

- [ ] 录屏：PLC 控制机械臂连续完成 **10 个取放循环无错位**；
- [ ] README 包含信号交互表 + 步序流程图；
- [ ] 能口述：基坐标 / 工具坐标 / 用户坐标的区别；DI/DO 信号含义；Profinet / EtherNet/IP / Modbus TCP 的区别；
- [ ] HMI 节拍时间可见并记录在 README（目标 ≤ 8 s/循环）；
- [ ] B 站 demo 录屏链接：（开发完成后补齐：B 站视频链接）

---

## 7. 无真机替代方案

全软件，0 硬件：

- FactoryIO Robot Arm 对象免费随 FactoryIO 提供；
- URSim 为 Universal Robots 官方免费仿真软件；
- PLCSIM 完成 PLC 侧仿真；
- 本项目所有画面均来自软件仿真，未连接实体机器人或实体 PLC。

---

## 8. 求职加分逻辑

- 面试中车系 / 汽车零部件厂，这一个项目即可把候选人从"会写梯形图的应届生"升级为"能做机器人工作站集成的人"；
- 面试官大概率追问"机器人和 PLC 怎么握手"——本项目的信号交互表与 CASE 步序就是标准答案。

---

## 9. 投入与成本

- 阶段 A：**30–40 小时**；
- 阶段 B（寒假升级）：额外 **20 小时**；
- 成本：**0 元**（FactoryIO Robot Arm + URSim + PLCSIM 均免费）。

---

更新日志：2026-09-29 创建项目技术方案文档
