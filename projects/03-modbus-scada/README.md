# 项目③ C# 上位机 Modbus TCP 数据采集监控 🚧 开发中

> 一句话定位：证明你不是只会梯形图——能把 PLC 数据通过 Modbus TCP 采到上位机、存进 SQLite、画成趋势曲线，并支持远程下发配方与断线自动重连。

---

## 1. 技术栈

| 层 | 选型 |
|---|---|
| PLC 侧 | S7-1200（PLCSIM），OB1 调用 `MB_SERVER`，本地端口 502 |
| 上位机 | C# WinForms（.NET 6），Visual Studio 2022 Community（免费） |
| 通信库 | NModbus4（NuGet，开源）或 S7.NET（更对口西门子，推荐 S7.NET） |
| 数据库 | SQLite（Microsoft.Data.Sqlite NuGet，零配置单文件） |
| 趋势曲线 | LiveCharts2（NuGet，开源） |
| 网络分析 | Wireshark（抓 502 端口 Modbus TCP 报文） |

---

## 2. PLC 侧 MB_SERVER 地址映射表

在 OB1 调用 `MB_SERVER`（背景 DB1，端口 502），地址映射如下：

| Modbus 地址 | PLC 侧地址 | 含义 |
|---|---|---|
| 保持寄存器 40001 | MW100 | 本班产量 |
| 保持寄存器 40002 | MW102 | 次品数 |
| 保持寄存器 40003 | DB1.DBD0 | 传送带速度（REAL 转 INT） |
| 线圈 00001 | Q0.0 | 远程启动 |
| 线圈 00002 | Q0.1 | 远程停止 |

---

## 3. C# 核心代码片段

### 3.1 连接与周期轮询

```csharp
using Modbus.Device;
using System.Net.Sockets;

private TcpClient _tcpClient;
private IModbusMaster _master;
private System.Timers.Timer _pollTimer;

private void ConnectPlc()
{
    _tcpClient = new TcpClient("192.168.0.1", 502);
    _master = ModbusIpMaster.CreateIp(_tcpClient);

    _pollTimer = new System.Timers.Timer(1000);   // 1 秒轮询一次
    _pollTimer.Elapsed += async (s, e) => await PollAsync();
    _pollTimer.Start();
}
```

### 3.2 周期读寄存器 + UI 线程更新

```csharp
private async Task PollAsync()
{
    try
    {
        ushort[] regs = _master.ReadHoldingRegisters(0, 4);   // 读 40001~40004
        this.Invoke((Action)(() =>
        {
            lblProd.Text   = regs[0].ToString();
            lblDefect.Text = regs[1].ToString();
            lblSpeed.Text  = regs[2].ToString();
            // LiveCharts2 追加新点
            _speedSeries.Values.Add(DateTime.Now.Second, regs[2]);
        }));
    }
    catch (SocketException ex)
    {
        Log($"断线：{ex.Message}，3 秒后重连...");
        _pollTimer.Stop();
        await Task.Delay(3000);
        Reconnect();
    }
}
```

### 3.3 写线圈 / 写寄存器（远程启停与配方下发）

```csharp
private void btnRemoteStart_Click(object sender, EventArgs e)
{
    _master.WriteSingleCoil(0, true);            // 线圈 00001 = 远程启动
}

private void btnSendRecipe_Click(object sender, EventArgs e)
{
    ushort target = (ushort)numTargetQty.Value;
    _master.WriteMultipleRegisters(2, new ushort[] { target });  // 下发目标产量
}
```

### 3.4 SQLite 存库

```csharp
using var conn = new SqliteConnection("Data Source=plc_data.db");
conn.Open();
using var cmd = conn.CreateCommand();
cmd.CommandText = @"INSERT INTO History (ts, prod, defect, speed)
                    VALUES (@ts, @prod, @defect, @speed);";
cmd.Parameters.AddWithValue("@ts", DateTime.Now.ToString("o"));
cmd.Parameters.AddWithValue("@prod", lblProd.Text);
cmd.Parameters.AddWithValue("@defect", lblDefect.Text);
cmd.Parameters.AddWithValue("@speed", lblSpeed.Text);
cmd.ExecuteNonQuery();
```

> 建表 SQL：
> ```sql
> CREATE TABLE IF NOT EXISTS History (
>     id INTEGER PRIMARY KEY AUTOINCREMENT,
>     ts TEXT, prod INTEGER, defect INTEGER, speed REAL
> );
> ```

---

## 4. 可验证完成标准（验收清单）

- [ ] 上位机 **1 秒刷新一次 PLC 寄存器**，趋势曲线连续滚动 **30 分钟无断点**；
- [ ] 在 C# 界面改配方（目标产量），PLC 监控表里 MW200 同步变化；
- [ ] 网线断开后 **3 秒内自动重连**，日志写入 `log.txt`；
- [ ] Wireshark 抓包截图：（开发完成后补齐：502 端口 Modbus TCP 报文截图）；
- [ ] C# 上位机界面截图：（开发完成后补齐：主界面 + 趋势曲线截图）；
- [ ] SQLite 表结构与查询结果截图。

---

## 5. 无真机替代方案

PLCSIM 当 Modbus Server，C# 当 Client，全部本地回环，0 硬件：

- VS 2022 Community 免费；
- NModbus4 / S7.NET 开源；
- SQLite 免费单文件；
- LiveCharts2 开源；
- 本项目所有通信均发生在 PLCSIM ↔ 本机 C# 进程之间，未连接实体 PLC。

---

## 6. 求职加分逻辑

| JD 关键词 | 本项目对应能力 |
|---|---|
| "上位机开发 / 数据采集" | C# WinForms + Modbus TCP 完整链路 |
| "Modbus 协议" | MB_SERVER + ReadHoldingRegisters + 写线圈/寄存器 |
| "MES / 设备联网 / OEE" | SQLite 历史存库 + 趋势曲线 |
| "断线重连 / 心跳" | catch SocketException + 3 秒重连 |
| "配方下发" | WriteMultipleRegisters 远程改 PLC 参数 |

---

## 7. 投入与成本

- 时间：**40–50 小时**；
- 成本：**0 元**（VS Community 免费、NModbus4 开源、SQLite 免费）。

---

更新日志：2026-09-29 创建项目技术方案文档
