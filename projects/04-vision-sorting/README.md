# 项目④ OpenCV 视觉检测 + PLC 分拣联动 🚧 开发中

> 一句话定位：跨学科证明你懂"视觉 + 运动控制"——Python/OpenCV 做颜色与位置识别，通过 Modbus TCP 把结果回传 S7-1200，PLC 按位置驱动推缸分流。

---

## 1. 技术栈

| 层 | 选型 |
|---|---|
| 视觉侧 | Python 3.10 + OpenCV（cv2）+ NumPy |
| 通信 | pyModbusTCP（pip 安装，Python 端当 Client） |
| PLC 侧 | 复用项目③的 MB_SERVER（端口 502），新增视觉寄存器 |
| 触发 | PLC 输出 Q1.5（拍照触发）→ Python 轮询检测 → 写回结果 |
| 图像源 | 本地图片 / 笔记本摄像头（可选二手 USB 工业相机 500–1500 元，非必须） |

---

## 2. 寄存器映射（在项目③ MB_SERVER 基础上扩展）

| Modbus 地址 | PLC 侧地址 | 含义 |
|---|---|---|
| 保持寄存器 40010 | MW300 | 视觉识别到的工件中心 X 坐标 |
| 保持寄存器 40011 | MW302 | 视觉结果（1=OK 红色件，2=OK 蓝色件，0=NG） |
| 线圈（PLC 侧） | Q1.5 | 拍照触发信号（PLC → Python） |

---

## 3. Python HSV 红色分割核心代码

```python
import cv2
import numpy as np

img = cv2.imread('part.jpg')
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

# 红色在 HSV 色环上跨越 0°，需要两个区间拼接
lower1 = np.array([0,   120, 70])
upper1 = np.array([10,  255, 255])
lower2 = np.array([170, 120, 70])
upper2 = np.array([180, 255, 255])

mask = cv2.inRange(hsv, lower1, upper1) | cv2.inRange(hsv, lower2, upper2)

contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

for c in contours:
    M = cv2.moments(c)
    if M['m00'] == 0:
        continue
    cx = int(M['m10'] / M['m00'])      # 工件中心 X 坐标
    cy = int(M['m01'] / M['m00'])
    # 后续：通过 pyModbusTCP 把 cx 和识别类别写入 MW300 / MW302
```

---

## 4. 联调逻辑（Python ↔ PLC）

1. PLC 光电传感器检测到工件到位 → 置位 Q1.5（拍照信号）；
2. Python 端以 100 ms 周期轮询线圈 Q1.5，检测到上升沿后读图；
3. Python 计算中心 X 坐标与类别（红/蓝/NG），通过 `write_single_register` 写入 MW300 / MW302；
4. PLC 收到 MW302 = 1 后，根据 MW300 判断左 / 中 / 右位置，延时 500 ms 后驱动对应推缸；
5. 推缸到位后复位 Q1.5，等待下一次触发。

> （开发完成后补齐：识别效果图——画 bounding box + 中心点 + 类别文字；Modbus 通信日志截图）

---

## 5. 可验证完成标准（验收清单）

- [ ] 对 **10 张测试图**识别准确率 **≥ 95%**；
- [ ] PLC 根据视觉结果正确分流 **3 个料道**，连续 **20 次分拣无误**；
- [ ] README 包含识别效果图（bounding box + 中心 + 类别）；
- [ ] README 包含 Modbus 通信日志截图（Python 写寄存器 ↔ PLC 接收）。

---

## 6. 无真机替代方案

- 无工业相机也能做：用笔记本摄像头实时采集，或加载本地图片 / 视频序列模拟传送带工件；
- 有条件可买二手 USB 工业相机（500–1500 元，行情推断），但**非必须**，前两版用图片即可；
- PLC 侧全部由 PLCSIM 仿真，未连接实体设备。

---

## 7. 求职加分逻辑

| JD 关键词 | 本项目对应能力 |
|---|---|
| "机器视觉 / OpenCV" | HSV 颜色分割 + findContours + 质心计算 |
| "视觉分选 / 视觉检测" | 颜色分类 + 位置坐标输出 |
| "PLC 与视觉通讯" | pyModbusTCP ↔ MB_SERVER 联调 |
| "视觉坐标系 ↔ PLC 坐标换算" | 中心 X 坐标写入 MW300，PLC 按位置推缸 |

> 3C / 锂电 / 包装行业视觉分选是标准工位；能讲清"视觉坐标系如何映射到 PLC 物理坐标"已超过大部分应届生。

---

## 8. 投入与成本

- 时间：**30–40 小时**；
- 成本：**0 元**（OpenCV、pyModbusTCP 全免费；工业相机为可选项）。

---

更新日志：2026-09-29 创建项目技术方案文档
