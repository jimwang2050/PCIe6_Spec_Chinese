# PCI Express Base Specification 6.2 — Chapter 4
# Physical Layer Logical Block
# 物理层逻辑子块

> 中英文对照翻译 | Chinese-English Parallel Translation
>
> Source: NCB-PCI_Express_Base_6.2-2024-01-25.pdf, Pages 351–650

---

## Chapter 4. Physical Layer Logical Block
## 第 4 章 物理层逻辑子块

---

## 快速导航 | Quick Navigation

| 章节 | 内容 |
|------|------|
| [4.1](#41-introduction) | 简介：物理层分工、编码与数据流模式 |
| [4.2](#42-logical-sub-block) | 逻辑子块：8b/10b / 128b/130b / 1b/1b 编码、Flit Mode、链路均衡、LTSSM、时钟容差补偿、Compliance 模式、Lane Margining |
| [4.3](#43-retimers) | 重定时器：转发规则、执行模式、延迟、SRIS、配置参数 |

---

## 4.1 Introduction
## 4.1 简介

The Physical Layer isolates the Transaction and Data Link Layers from the signaling technology used for Link data interchange. It is divided into the **logical sub-block** and **electrical sub-block**. Chapter 4 describes the logical sub-block; Chapter 8 describes the electrical sub-block.

> 物理层将事务层和数据链路层与用于链路数据交换的信令技术隔离开来。它分为**逻辑子块**和**电气子块**。第 4 章描述逻辑子块；第 8 章描述电气子块。

---

The logical sub-block has two main sections:

- **Transmit section**: prepares outgoing information passed from the Data Link Layer for transmission by the electrical sub-block
- **Receiver section**: identifies and prepares received information before passing it to the Data Link Layer

> 逻辑子块有两个主要部分：
>
> - **发送部分**：准备从数据链路层传递的传出信息，供电气子块发送
> - **接收部分**：识别并准备接收到的信息，然后传递给数据链路层

---

### Encoding and Data Stream Modes | 编码与数据流模式

PCI Express uses three encoding types and two Data Stream modes:

**Table 4-1 Valid Encoding and Data Stream Mode Combinations | 表 4-1 编码与数据流模式组合**

| Data Rate | Flit Mode | Encoding | Data Stream |
|-----------|-----------|----------|-------------|
| 2.5, 5.0 GT/s | No | 8b/10b | Non-Flit Mode |
| 2.5, 5.0 GT/s | Yes | 8b/10b | Flit Mode |
| 8.0, 16.0, 32.0 GT/s | No | 128b/130b | Non-Flit Mode |
| 8.0, 16.0, 32.0 GT/s | Yes | 128b/130b | Flit Mode |
| 64.0 GT/s | Yes (mandatory) | 1b/1b | Flit Mode |

---

## 4.2 Logical Sub-block
## 4.2 逻辑子块

### 4.2.1 8b/10b Encoding (2.5 & 5.0 GT/s) | 8b/10b 编码

#### Symbol Encoding | 符号编码

8b/10b encoding maps each 8-bit byte to a 10-bit symbol. It ensures DC balance and sufficient transition density for clock recovery. Special symbols (**K-Codes**) are used for framing and Link management:

- **COM (K28.5)**: Used in Training Sequences for symbol alignment
- **SKP (K28.0)**: Clock tolerance compensation (SKiP)
- **STP (K27.7)**: Start of TLP
- **SDP (K28.2)**: Start of DLLP
- **END (K29.7)**: End of TLP/DLLP
- **EDB (K30.7)**: End of Bad TLP (nullification)
- **PAD (K23.7)**: Packet padding
- **IDL (K28.3)**: Logical Idle

> 8b/10b 编码将每个 8 位字节映射为一个 10 位符号。它确保直流平衡和足够的跳变密度以进行时钟恢复。特殊符号（**K 码**）用于帧定界和链路管理。

---

#### Data Scrambling | 数据加扰

Data bytes (D-codes) are scrambled using a 16-bit LFSR to reduce EMI and ensure sufficient transition density. The scrambling polynomial is G(x) = X^16 + X^5 + X^4 + X^3 + 1.

> 数据字节（D 码）使用 16 位 LFSR 进行加扰，以减少 EMI 并确保足够的跳变密度。加扰多项式为 G(x) = X^16 + X^5 + X^4 + X^3 + 1。

---

### 4.2.2 128b/130b Encoding (8.0, 16.0 & 32.0 GT/s) | 128b/130b 编码

Each Block is 130 bits: a 2-bit Sync Header followed by 128-bit Payload. The Sync Header indicates whether the Payload is a Data Block (01b) or an Ordered Set Block (10b).

> 每个 Block 为 130 位：2 位同步头后跟 128 位有效载荷。同步头指示有效载荷是数据块（01b）还是有序集块（10b）。

---

#### Framing Tokens (Non-Flit Mode) | 帧定界令牌

TLPs and DLLPs are framed using Tokens embedded within the 128b/130b Data Blocks:

| Token | Encoding | Purpose |
|-------|----------|---------|
| **STP** | Start of TLP | Marks beginning of a TLP |
| **SDP** | Start of DLLP | Marks beginning of a DLLP |
| **EDB** | End of Bad TLP | Nullification token |
| **IDL** | IDLe | Logical idle |
| **EDS** | End of Data Stream | End of data stream marker |

The STP Token includes an 11-bit **TLP Length** field protected by a 4-bit Frame CRC and an even parity bit (Frame Parity). Together they guarantee 3-bit error detection for the TLP Length field.

> STP Token 包含一个 11 位的 **TLP Length** 字段，由 4 位 Frame CRC 和一个偶校验位（Frame Parity）保护。它们共同保证 TLP Length 字段的 3 位错误检测。

---

#### Scrambling (128b/130b) | 加扰

Uses a 23-bit LFSR with polynomial G(x) = X^23 + X^21 + X^16 + X^8 + X^5 + X^2 + 1. SKP Ordered Sets bypass scrambling to maintain clock compensation functionality.

> 使用 23 位 LFSR，多项式为 G(x) = X^23 + X^21 + X^16 + X^8 + X^5 + X^2 + 1。SKP 有序集绕过加扰以保持时钟补偿功能。

---

### 4.2.3 Flit Mode Operation | Flit 模式操作

#### 4.2.3.1 1b/1b Encoding (64.0 GT/s+) | 1b/1b 编码

At 64.0 GT/s and higher, PCIe uses **1b/1b encoding with PAM4 signaling** (2 bits per symbol). Key characteristics:

- **PAM4 signaling**: 4 voltage levels encoding 2 bits (LSB, MSB) per Unit Interval
- **Gray coding**: adjacent PAM4 levels differ by only 1 bit, minimizing error impact
- **Precoding**: 1/(1+D) mod 4 precoding to mitigate error propagation
- **Scrambling**: Uses 23-bit LFSR, bypasses during SKP OS

> 在 64.0 GT/s 及以上速率，PCIe 使用 **1b/1b 编码配合 PAM4 信令**。关键特性：
>
> - **PAM4 信令**：4 个电平编码每个 UI 2 位（LSB、MSB）
> - **格雷编码**：相邻 PAM4 电平仅相差 1 位，最小化错误影响
> - **预编码**：1/(1+D) mod 4 预编码以减轻错误传播
> - **加扰**：使用 23 位 LFSR，在 SKP OS 期间绕过

---

#### Data Stream in Flit Mode | Flit 模式数据流

In Flit Mode, data is organized into **Flits** — fixed-size transfer units. Each Flit contains:

- **TLP bytes**: Transaction Layer data
- **DLP bytes**: Data Link Layer payload (Ack/Nak, credit, Flit sequence numbers)
- **CRC bytes**: Error detection for the Flit
- **ECC bytes**: Forward Error Correction

> Flit 模式下，数据组织为 **Flit** — 固定大小的传输单元。每个 Flit 包含：
>
> - **TLP 字节**：事务层数据
> - **DLP 字节**：数据链路层有效载荷（Ack/Nak、信用、Flit 序列号）
> - **CRC 字节**：Flit 的错误检测
> - **ECC 字节**：前向纠错

---

#### Flit Sequence Number and Retry | Flit 序列号与重试

Flit Mode uses a three-phase handshake:

1. **IDLE Flit Handshake Phase**: Initial alignment
2. **Sequence Number Handshake Phase**: Establish initial sequence numbers
3. **Normal Flit Exchange Phase**: Regular data transfer with Ack/Nak/Replay

> Flit 模式使用三阶段握手：
>
> 1. **IDLE Flit 握手阶段**：初始对齐
> 2. **序列号握手阶段**：建立初始序列号
> 3. **正常 Flit 交换阶段**：带 Ack/Nak/重放的常规数据传输

---

### 4.2.4 Link Equalization (8.0 GT/s+) | 链路均衡

Link Equalization is a multi-phase procedure that optimizes transmitter settings at both ends of a Link for 8.0 GT/s and higher data rates:

- **Phase 0** (Upstream only): Upstream port adjusts its transmitter
- **Phase 1**: Downstream port adjusts, Upstream provides feedback
- **Phase 2**: Upstream port adjusts, Downstream provides feedback
- **Phase 3**: Fine-tuning and final settings lock

Transmitter settings include **Pre-cursor (C[-1])**, **Cursor (C[0])**, and **Post-cursor (C[+1])** coefficients. Presets provide predefined coefficient combinations for common channel characteristics.

> 链路均衡是一个多阶段过程，优化链路两端的发送器设置：
>
> - **阶段 0**（仅上游）：上游端口调整发送器
> - **阶段 1**：下游端口调整，上游提供反馈
> - **阶段 2**：上游端口调整，下游提供反馈
> - **阶段 3**：微调和最终设置锁定
>
> 发送器设置包括前标（C[-1]）、主标（C[0]）和后标（C[+1]）系数。

---

### 4.2.5–4.2.7 LTSSM (Link Training and Status State Machine)
### 4.2.5–4.2.7 链路训练与状态机

The LTSSM is the heart of the Physical Layer logical sub-block. It manages Link initialization, training, power management, and recovery.

> LTSSM 是物理层逻辑子块的核心。它管理链路初始化、训练、电源管理和恢复。

---

#### LTSSM States | 状态一览

| 状态 | 描述 |
|------|------|
| **Detect** | Detect presence of a receiver at the far end (Quiet → Active) |
| **Polling** | Bit lock, symbol lock, Lane polarity, compliance testing |
| **Configuration** | Link width negotiation, Lane numbering, Lane-to-Lane de-skew |
| **Recovery** | Re-establish bit/symbol lock, equalization, speed change |
| **L0** | Normal operation — TLPs, DLLPs transmitted |
| **L0s** | Short power-saving idle (one direction at a time) |
| **L0p** | Low-power state for Flit Mode with rapid exit |
| **L1** | Higher latency power-saving state |
| **L2** | Deepest power-saving; main power and clock may be removed |
| **Disabled** | Link disabled; transmitter in Electrical Idle |
| **Loopback** | Test mode: transmitted data looped back |
| **Hot Reset** | In-band reset via Training Sequences |

> | 状态 | 描述 |
> |------|------|
> | **Detect** | 检测远端是否有接收器 |
> | **Polling** | 位锁定、符号锁定、通道极性、Compliance 测试 |
> | **Configuration** | 链路宽度协商、通道编号、通道间去偏斜 |
> | **Recovery** | 重新建立位/符号锁定、均衡、速率更改 |
> | **L0** | 正常运行 — 传输 TLP 和 DLLP |
> | **L0s** | 短期省电空闲（单向） |
> | **L0p** | Flit 模式低功耗状态，快速退出 |
> | **L1** | 较高延迟的省电状态 |
> | **L2** | 最深省电；可移除主电源和时钟 |
> | **Disabled** | 链路禁用 |
> | **Loopback** | 测试模式：数据环回 |
> | **Hot Reset** | 通过训练序列的带内复位 |

---

#### Key Training Sequences | 关键训练序列

- **TS1 (Training Sequence 1)**: Used in Polling and Configuration; carries Link/Lane numbers
- **TS2 (Training Sequence 2)**: Used in Configuration; completes Link training
- **FTS (Fast Training Sequence)**: Used for quick L0s recovery (8b/10b only)
- **EIEOS (Electrical Idle Exit Ordered Set)**: Signals exit from Electrical Idle
- **SDS (Start of Data Stream)**: Marks start of data stream after training

> - **TS1**：用于 Polling 和 Configuration；携带链路/通道号
> - **TS2**：用于 Configuration；完成链路训练
> - **FTS**：用于快速 L0s 恢复（仅 8b/10b）
> - **EIEOS**：信号退出电气空闲
> - **SDS**：标记训练后数据流的开始

---

#### Configuration Detailed | 配置子状态

| 子状态 | Downstream | Upstream |
|--------|-----------|----------|
| **Linkwidth.Start** | Transmit TS1 with Link numbers on active Lanes | Respond with TS1 |
| **Linkwidth.Accept** | Accept proposed Link width | Confirm proposed width |
| **Lanenum.Accept** | Assign Lane numbers | Accept Lane numbering |
| **Lanenum.Wait** | Wait for Lane number acceptance | Drive configured Lanes |
| **Configuration.Complete** | Send TS2, prepare for L0 | Respond with TS2 |
| **Configuration.Idle** | Wait for negotiated data rate | Transition to L0 at data rate |

---

#### Recovery Detailed | 恢复子状态

| 子状态 | 功能 |
|--------|------|
| **RcvrLock** | Re-establish bit lock and symbol alignment |
| **Equalization** | Multi-phase transmitter equalization (8.0 GT/s+) |
| **Speed** | Data rate change negotiation |
| **RcvrCfg** | Receiver configuration, Lane-to-Lane de-skew |
| **Recovery.Idle** | Transition to L0 or next state |

Key transitions to Recovery: from L0 on error, from L0s/L1 on exit, for speed change, or for equalization redo.

> Recovery 的关键转换来源：L0 出错时、L0s/L1 退出时、速率更改时、均衡重做时。

---

### 4.2.8 Clock Tolerance Compensation | 时钟容差补偿

PCI Express uses **SKP Ordered Sets** to compensate for clock frequency differences between transmitter and receiver (up to ±300 ppm for SRNS, ±600 ppm for SRIS).

| Encoding | SKP OS Content |
|----------|---------------|
| 8b/10b | COM + 1–5 SKP symbols (added/removed by receiver) |
| 128b/130b | SKP OS with 4 SKP symbols (receivers add/drop 1 SKP) |
| 1b/1b | SKP OS with variable SKP symbols for PAM4 |

Receivers add or drop SKP symbols to match their local clock rate with the incoming data rate.

> 接收器添加或删除 SKP 符号以使其本地时钟速率与输入数据速率匹配。

---

### 4.2.9–4.2.17 Compliance Patterns | 合规性模式

Various compliance test patterns exist for each encoding:

- **8b/10b**: PRBS patterns for 2.5/5.0 GT/s
- **128b/130b**: Compliance patterns CP1–CP8, jitter measurement patterns
- **1b/1b**: Patterns for 64.0 GT/s PAM4 compliance testing
- **Modified Compliance Patterns**: Alternative patterns for specific test scenarios
- **Toggle Patterns**: Used for 1b/1b encoding validation

> 各编码方式都有相应的合规性测试模式，包括 PRBS、CP1–CP8、抖动测量模式和切换模式。

---

### 4.2.18 Lane Margining at Receiver | 接收器通道裕度测试

Lane Margining allows software to command a receiver to sample its incoming data at an offset from the nominal sampling point, measuring the available timing (horizontal) or voltage (vertical) margin:

- **Step Margin**: Commands receiver to step through offset values and report results
- **Usage Models**: Independent (Lane-level) or Push-Out (all Lanes simultaneously)
- **Margin Types**: Timing (UI offset) or Voltage (voltage offset)
- **Receiver Number**: Up to 64 independent receivers per Port

> 通道裕度测试允许软件命令接收器在偏离标称采样点的位置对输入数据进行采样，测量可用的时序（水平）或电压（垂直）裕度。

---

## 4.3 Retimers
## 4.3 重定时器

A **Retimer** is a physical layer component that sits between two PCIe Links, receiving and retransmitting the data stream on a per-Lane basis. It compensates for channel losses without terminating the Link protocol.

> **重定时器**是位于两条 PCIe 链路之间的物理层组件，在每条通道上接收并重新发送数据流。它补偿通道损耗而不终止链路协议。

---

### Key Retimer Functions | 关键重定时器功能

- **Forwarding Rules**: Forward TS1/TS2, EIEOS, SDS, SKP OS, Electrical Idle, data streams
- **Transmitter Settings**: Determine transmitter coefficients independently for each sub-Link
- **Data Rate Determination**: Track Link data rate from training sequences
- **Execution Modes**: CompLoadBoard, Link Equalization, Follower Loopback
- **Latency**: Bounded retimer latency requirements
- **SRIS**: Support for Separate Refclk Independent SSC architectures
- **L1 PM Substates**: Support for L1.1/L1.2 power management substates

> - **转发规则**：转发 TS1/TS2、EIEOS、SDS、SKP OS、电气空闲、数据流
> - **发送器设置**：为每个子链路独立确定发送器系数
> - **数据速率确定**：从训练序列中跟踪链路数据速率
> - **执行模式**：CompLoadBoard、链路均衡、从动环回
> - **延迟**：有界的重定时器延迟要求
> - **SRIS**：支持独立展频参考时钟架构
> - **L1 PM 子状态**：支持 L1.1/L1.2 电源管理子状态

---

## Appendix: Key Acronyms | 附录：关键缩略语

| 缩略语 | 全称 | 中文翻译 |
|--------|------|---------|
| LTSSM | Link Training and Status State Machine | 链路训练与状态机 |
| TLP | Transaction Layer Packet | 事务层数据包 |
| DLLP | Data Link Layer Packet | 数据链路层数据包 |
| TS1/TS2 | Training Sequence 1/2 | 训练序列 1/2 |
| FTS | Fast Training Sequence | 快速训练序列 |
| EIEOS | Electrical Idle Exit Ordered Set | 电气空闲退出有序集 |
| EIOS | Electrical Idle Ordered Set | 电气空闲有序集 |
| SDS | Start of Data Stream | 数据流开始 |
| SKP | Skip | 跳过（时钟补偿） |
| STP/SDP | Start of TLP/DLLP | TLP/DLLP 开始 |
| EDB | End of Bad TLP | 坏 TLP 结束 |
| IDL | IDLe | 逻辑空闲 |
| COM | Comma | 逗号（符号对齐） |
| PAM4 | Pulse Amplitude Modulation 4-level | 4 电平脉冲幅度调制 |
| LFSR | Linear Feedback Shift Register | 线性反馈移位寄存器 |
| CRC | Cyclic Redundancy Check | 循环冗余校验 |
| ECC | Error Correction Code | 纠错码 |
| SRIS | Separate Refclk Independent SSC | 独立参考时钟独立展频 |
| SRNS | Separate Refclk No SSC | 独立参考时钟无展频 |
| SSC | Spread Spectrum Clocking | 展频时钟 |
| UI | Unit Interval | 单位间隔 |

---

> **Translator's Notes | 译者说明**
>
> 本文档翻译自 PCI Express Base Specification Revision 6.2 (January 25, 2024) 第 4 章全文 (Pages 351–650, 300 pages)。
> 第 4 章是规范中篇幅最大的章节之一，涵盖完整的物理层逻辑子块规范。
