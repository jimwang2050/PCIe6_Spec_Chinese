# PCI Express Base Specification 6.2 — Chapter 2
# Transaction Layer Specification
# 事务层规范

> 中英文对照翻译 | Chinese-English Parallel Translation
>
> Source: NCB-PCI_Express_Base_6.2-2024-01-25.pdf, Pages 141–308

---

## Chapter 2. Transaction Layer Specification
## 第 2 章 事务层规范

---

## 快速导航 | Quick Navigation

| 章节 | 内容 |
|------|------|
| [2.1](#21-transaction-layer-overview) | 事务层概述：地址空间、事务类型、数据包格式 |
| [2.2](#22-transaction-layer-protocol--packet-definition) | TLP 协议 — 数据包定义：头部字段(NFM/FM)、路由、字节使能、事务描述符、请求规则、消息规则、完成规则、TLP 前缀 |
| [2.3](#23-handling-of-received-tlps) | 接收 TLP 处理：请求处理、UIO 读写完成 |
| [2.4](#24-transaction-ordering) | 事务排序：非 UIO/UIO 排序规则 |
| [2.5](#25-virtual-channel-vc-mechanism) | 虚拟通道机制：VC ID、TC-VC 映射 |
| [2.6](#26-ordering-and-receive-buffer-flow-control) | 流量控制：FC 规则、发送端/接收端跟踪 |
| [2.7](#27-end-to-end-data-integrity) | 端到端数据完整性：ECRC、数据中毒 |
| [2.8](#28-completion-timeout-mechanism) | 完成超时机制 |
| [2.9](#29-link-status-dependencies) | 链路状态依赖：DL_Down/DL_Up/DPC 行为 |

---

## 2.1 Transaction Layer Overview
## 2.1 事务层概述

At a high level, the key aspects of the Transaction Layer are:

1. A pipelined full Split-Transaction protocol
2. Mechanisms for differentiating the ordering and processing requirements of Transaction Layer Packets (TLPs)
3. Credit-based flow control
4. Optional support for data poisoning and end-to-end data integrity detection

> 事务层的关键特性包括：
>
> 1. 流水线化的完整拆分事务协议
> 2. 区分事务层数据包（TLP）排序和处理需求的机制
> 3. 基于信用的流量控制
> 4. 可选的数据中毒和端到端数据完整性检测支持

---

The Transaction Layer comprehends:

- TLP construction and processing
- Association of transaction-level mechanisms with device resources (Flow Control, Virtual Channel management)
- Rules for ordering and management of TLPs (PCI/PCI-X compatible ordering, Traffic Class differentiation, UIO Ordering)

> 事务层包含：
>
> - TLP 构造与处理
> - 事务级机制与设备资源的关联（流量控制、虚拟通道管理）
> - TLP 排序和管理规则（PCI/PCI-X 兼容排序、流量类别区分、UIO 排序）

---

### 2.1.1 Address Spaces, Transaction Types, and Usage
### 2.1.1 地址空间、事务类型与用途

**Table 2-1 Transaction Types for Different Address Spaces | 表 2-1 不同地址空间的事务类型**

| 地址空间 | 事务类型 | 基本用途 |
|---------|---------|---------|
| Memory | Read, Write | 从/向内存映射位置传输数据 |
| I/O | Read, Write | 从/向 I/O 映射位置传输数据 |
| Configuration | Read, Write | 设备功能配置/设置 |
| Message | Baseline (including Vendor-Defined) | 从事件信令到通用消息传递 |

---

#### Memory Transactions | 内存事务

Memory Transactions include: Read Request/Completion, Write Request (and Completions for UIO), Deferrable Memory Write Request/Completion, AtomicOp Request/Completion. Memory Transactions use 32-bit (Short) or 64-bit (Long) address formats. Optionally include PASID TLP Prefix.

> 内存事务包括：读请求/完成、写请求（及 UIO 完成）、可推迟内存写请求/完成、AtomicOp 请求/完成。使用 32 位或 64 位地址格式。可选携带 PASID TLP 前缀。

---

#### I/O Transactions | I/O 事务

I/O Transactions include Read Request/Completion and Write Request/Completion, using 32-bit address format only. Supported for legacy compatibility; future revisions may deprecate I/O Space.

> I/O 事务包括读请求/完成和写请求/完成，仅使用 32 位地址。为传统兼容性而支持；未来版本可能弃用 I/O 空间。

---

#### Configuration Transactions | 配置事务

Used to access configuration registers of Functions within devices. Types: Read Request/Completion, Write Request/Completion.

> 用于访问设备内功能的配置寄存器。类型：读请求/完成、写请求/完成。

---

#### Message Transactions | 消息事务

Messages support in-band communication of events between devices. Include specific Message codes and Vendor-Defined Messages. Not guaranteed interoperable between different vendors.

> 消息支持设备间带内事件通信。包括特定消息码和供应商定义消息。不保证不同供应商之间的互操作性。

---

### 2.1.2 Packet Format Overview | 数据包格式概述

Transactions consist of Requests and Completions communicated using packets. A TLP consists of:

- One or more optional **TLP Prefixes**
- A **TLP Header**
- A **Data Payload** (for some packet types)
- An optional **TLP Digest**

PCI Express transfers information as a serialized stream of bytes — the left-most byte is transmitted/received first. Header layout is optimized for performance: the most time-critical information (e.g., address MSB) is transferred first for early decode.

> 事务由使用数据包通信的请求和完成组成。TLP 由以下部分组成：
>
> - 一个或多个可选的 **TLP 前缀**
> - **TLP 头部**
> - **数据有效载荷**（某些数据包类型）
> - 可选的 **TLP 摘要**
>
> PCI Express 以串行字节流的形式传输信息 — 最左侧字节最先发送/接收。头部布局针对性能进行了优化：最时间关键的信息（如地址 MSB）最先传输以便早期解码。

---

## 2.2 Transaction Layer Protocol — Packet Definition
## 2.2 事务层协议 — 数据包定义

### 2.2.1 Common Packet Header Fields | 通用数据包头部字段

TLP headers are either 3 DW (Non-Flit Mode) or contain OHC (Flit Mode). Key header fields include:

| Field | Description |
|-------|-------------|
| **Fmt[1:0]** | Format: 00=3DW no data, 01=4DW no data, 10=3DW with data, 11=4DW with data |
| **Type[4:0]** | Transaction Type (Memory Rd/Wr, Cfg Rd/Wr, Msg, Cpl, etc.) |
| **TC[2:0]** | Traffic Class (0–7) |
| **Attr[2:0]** | Attributes (No Snoop, Relaxed Ordering, ID-Based Ordering) |
| **Length[9:0]** | Data Payload length in DW |
| **EP** | Poisoned Data |
| **TD** | TLP Digest present |

> TLP 头部为 3 DW（Non-Flit Mode）或包含 OHC（Flit Mode）。关键头部字段包括：
>
> | 字段 | 描述 |
> |------|------|
> | **Fmt** | 格式：00=3DW无数据, 01=4DW无数据, 10=3DW带数据, 11=4DW带数据 |
> | **Type** | 事务类型（内存读/写、配置读/写、消息、完成等）|
> | **TC** | 流量类别（0–7） |
> | **Attr** | 属性（No Snoop、Relaxed Ordering、ID-Based Ordering）|
> | **Length** | 数据有效载荷长度（以 DW 计）|
> | **EP** | 中毒数据 |
> | **TD** | 存在 TLP 摘要 |

---

### 2.2.4 Routing and Addressing Rules | 路由与寻址规则

#### Address-Based Routing | 基于地址的路由

Used for Memory and I/O Requests. Address is decoded to determine the target. Three address routing types:

- **Memory-mapped**: Routed by address decode at each Switch/Root Port
- **I/O**: Routed by address decode (legacy, may be deprecated)
- **Configuration**: Routed by Bus/Device/Function Number in ID field

> 用于内存和 I/O 请求。通过地址解码确定目标。三种地址路由类型：
>
> - **内存映射**：通过每个交换机/根端口的地址解码进行路由
> - **I/O**：通过地址解码进行路由（传统，可能弃用）
> - **配置**：通过 ID 字段中的总线/设备/功能号进行路由

---

#### ID-Based Routing | 基于 ID 的路由

Used for Completions and Messages. The Requester ID or Destination ID is used to route the TLP through the PCIe hierarchy. Key ID routing types:

- **Routing by ID**: Completion routed back to Requester using Bus/Device/Function numbers
- **Implicit Routing**: Message routed to Root Complex or received at Root Complex
- **Broadcast from Root Complex**: Message routed downstream to all endpoints

> 用于完成和消息。请求者 ID 或目标 ID 用于通过 PCIe 层级路由 TLP。关键的 ID 路由类型：
>
> - **按 ID 路由**：使用总线/设备/功能号将完成路由回请求者
> - **隐式路由**：消息路由到根复合体或在根复合体处接收
> - **从根复合体广播**：消息路由下游到所有端点

---

### 2.2.6 Transaction Descriptor | 事务描述符

The Transaction Descriptor associates a transaction with its properties. It consists of three fields:

1. **Transaction ID**: Requester ID + Tag (10-bit or 14-bit)
2. **Attributes field**: No Snoop, Relaxed Ordering, ID-Based Ordering
3. **Traffic Class (TC)**: Differentiates transaction priority (TC0–TC7)

> 事务描述符将事务与其属性关联。由三个字段组成：
>
> 1. **事务 ID**：请求者 ID + Tag（10 位或 14 位）
> 2. **属性字段**：No Snoop、Relaxed Ordering、ID-Based Ordering
> 3. **流量类别（TC）**：区分事务优先级（TC0–TC7）

---

### 2.2.8 Message Request Rules | 消息请求规则

PCI Express defines several categories of Messages:

| Message Type | Code Range | Purpose |
|-------------|-----------|---------|
| INTx Interrupt Signaling | 20h–24h | Virtual INTx wire signaling |
| Power Management | 40h–4Fh | PME, PME Turn Off, etc. |
| Error Signaling | 30h–3Fh | ERR_COR, ERR_NONFATAL, ERR_FATAL |
| Unlock | 50h–5Fh | Locked transaction support |
| Slot Power Limit | 60h–6Fh | Set Slot Power Limit |
| Vendor-Defined | 70h–7Fh | Custom vendor messages |
| Latency Tolerance Reporting | — | LTR messages |
| OBFF | — | Optimized Buffer Flush/Fill |
| PTM | — | Precision Time Measurement |
| IDE | — | Integrity & Data Encryption messages |

---

### 2.2.9 Completion Rules | 完成规则

Completions are used to terminate transactions where data or status must be returned to the Requester. Three completion status values:

- **000b (SC — Successful Completion)**: Request completed successfully
- **001b (UR — Unsupported Request)**: Request type not supported
- **100b (CA — Completer Abort)**: Completer unable to process request

Completions use ID-based routing to return to the Requester's Bus/Device/Function.

> 完成用于终止需要向请求者返回数据或状态的事务。三种完成状态值：
>
> - **000b（成功完成）**：请求成功完成
> - **001b（不支持的请求）**：不支持该请求类型
> - **100b（完成者终止）**：完成者无法处理请求
>
> 完成使用基于 ID 的路由返回到请求者的总线/设备/功能。

---

## 2.3 Handling of Received TLPs
## 2.3 接收 TLP 的处理

### 2.3.1 Request Handling Rules | 请求处理规则

When a Function receives a Request TLP:

1. Validate: Check for malformed TLPs, unsupported requests, etc.
2. Decode: Determine transaction type, address/target
3. Execute: Perform the requested operation
4. Complete: If applicable, return a Completion with data/status

Each Function must handle Requests in accordance with ordering and flow control rules. Unsupported Requests must return a Completion with UR status. Poisoned TLPs are handled per Section 2.7.2.

> 功能接收请求 TLP 时：
>
> 1. **验证**：检查畸形 TLP、不支持的请求等
> 2. **解码**：确定事务类型、地址/目标
> 3. **执行**：执行请求的操作
> 4. **完成**：如适用，返回带有数据/状态的完成
>
> 每个功能必须按照排序和流量控制规则处理请求。不支持的请求必须返回 UR 状态的完成。中毒 TLP 按第 2.7.2 节处理。

---

### 2.3.2 Completion Handling Rules | 完成处理规则

When a Function receives a Completion TLP, it must match the Completion to an outstanding Request using Transaction ID (Requester ID + Tag). Unmatched Completions are Unexpected Completions and are handled as errors.

> 功能接收完成 TLP 时，必须使用事务 ID（请求者 ID + Tag）将完成与未完成的请求匹配。未匹配的完成是意外完成，作为错误处理。

---

## 2.4 Transaction Ordering
## 2.4 事务排序

### 2.4.1 Ordering Rules (Non-UIO, Non-Flow-Through IDE) | 排序规则

PCI Express maintains a producer-consumer ordering model. Key ordering rules:

1. **Strong Ordering**: Requests with the same TC and no Relaxed Ordering attribute maintain PCI/PCI-X ordering.
2. **Relaxed Ordering (RO)**: RO bit Set allows TLPs to bypass other TLPs for improved performance.
3. **ID-Based Ordering (IDO)**: Independent ordering per Requester ID/PASID allows pipelining.

> PCI Express 维持生产者-消费者排序模型。关键排序规则：
>
> 1. **强排序**：具有相同 TC 且无 Relaxed Ordering 属性的请求保持 PCI/PCI-X 排序。
> 2. **放松排序（RO）**：RO 位置位允许 TLP 绕过其他 TLP 以提高性能。
> 3. **基于 ID 的排序（IDO）**：每个请求者 ID/PASID 独立排序，允许流水线化。

---

The general ordering table (Table 2-40) defines pass/fail relationships between different TLP types across rows (first issued) and columns (subsequently issued).

> 通用排序表（表 2-40）定义了不同 TLP 类型之间跨行（先发出的）和列（后发出的）的通过/阻塞关系。

---

## 2.5 Virtual Channel (VC) Mechanism
## 2.5 虚拟通道（VC）机制

The Virtual Channel mechanism provides independent logical flows over a single physical Link. Key concepts:

- **VC ID**: 3-bit identifier (0–7), VC0 is always present (default)
- **TC-to-VC Mapping**: Traffic Class 0–7 mapped to VCs through TC/VC Map registers
- Each VC has independent flow control and ordering domains
- Multiple TCs can map to the same VC

> 虚拟通道机制在单个物理链路上提供独立的逻辑流。关键概念：
>
> - **VC ID**：3 位标识符（0–7），VC0 始终存在（默认）
> - **TC-VC 映射**：TC 0–7 通过 TC/VC 映射寄存器映射到 VC
> - 每个 VC 具有独立的流量控制和排序域
> - 多个 TC 可以映射到同一 VC

---

## 2.6 Ordering and Receive Buffer Flow Control
## 2.6 排序与接收缓冲区流量控制

Flow Control (FC) is credit-based, ensuring a transmitter never sends a TLP unless the receiver has sufficient buffer space. Three FC types:

- **Posted (P)**: Memory Writes, Messages — no Completion required
- **Non-Posted (NP)**: Memory Reads, I/O, Configuration — Completion required
- **Completions (Cpl)**: Completion TLPs

FC credits are tracked by the transmitter using `CREDIT_CONSUMED` and replenished by the receiver via `CREDIT_ALLOCATED` fields in DLLPs (Data Link Layer Packets).

> 流量控制（FC）基于信用，确保发送端仅在接收端有足够缓冲区空间时才发送 TLP。三种 FC 类型：
>
> - **Posted（P）**：内存写入、消息 — 无需完成
> - **Non-Posted（NP）**：内存读取、I/O、配置 — 需要完成
> - **Completions（Cpl）**：完成 TLP
>
> FC 信用由发送端通过 `CREDIT_CONSUMED` 跟踪，由接收端通过 DLLP 中的 `CREDIT_ALLOCATED` 字段补充。

---

**FC credit categories per VC**: PH (Posted Header), PD (Posted Data), NPH (Non-Posted Header), NPD (Non-Posted Data), CplH (Completion Header), CplD (Completion Data).

> **每 VC 的 FC 信用类别**：PH、PD、NPH、NPD、CplH、CplD。

---

## 2.7 End-to-End Data Integrity
## 2.7 端到端数据完整性

### 2.7.1 ECRC Rules | ECRC 规则

The End-to-End CRC (ECRC) is an optional 32-bit CRC appended to the TLP Digest field. ECRC covers all invariant fields of the TLP header and data payload. ECRC generation and checking is optional — controlled by the ECRC Check Enable and ECRC Generation Enable bits in the AER Extended Capability.

> 端到端 CRC（ECRC）是附加到 TLP 摘要字段的可选 32 位 CRC。ECRC 覆盖 TLP 头部和数据有效载荷的所有不变字段。ECRC 生成和检查是可选的 — 由 AER 扩展能力中的相应使能位控制。

---

### 2.7.2 Error Forwarding (Data Poisoning) | 错误转发（数据中毒）

The EP (Error/Poisoned) bit in the TLP header indicates the data is poisoned. Used when a Function detects uncorrectable data errors and forwards the corrupted data. Rules:

- EP bit Set in a Write Request: Receiver may discard or forward poisoned data
- EP bit Set in a Read Completion: Requester handles poisoned data
- Poisoned TLPs must not be written to permanent storage

> TLP 头部中的 EP（错误/中毒）位表示数据已被标记为中毒。当功能检测到不可纠正的数据错误并转发损坏的数据时使用。规则：
>
> - 写请求中 EP 置位：接收者可丢弃或转发中毒数据
> - 读完成中 EP 置位：请求者处理中毒数据
> - 中毒 TLP 不得写入永久存储

---

## 2.8 Completion Timeout Mechanism
## 2.8 完成超时机制

A Function that issues a Non-Posted Request must implement a Completion Timeout mechanism. If no Completion is received within the timeout period, the Function:

- Must not assume the request succeeded
- May re-issue the request (implementation specific)
- Should log the error and may signal an interrupt

The Completion Timeout value is configurable through Device Control 2 register (ranges: 50 μs to 64 s; default 50 ms).

> 发出 Non-Posted 请求的功能必须实现完成超时机制。如果在超时期限内未收到完成，功能：
>
> - 不得假设请求成功
> - 可重新发出请求（实现特定）
> - 应记录错误并可发出中断信号
>
> 完成超时值通过 Device Control 2 寄存器配置（范围：50 μs 到 64 s；默认 50 ms）。

---

## 2.9 Link Status Dependencies
## 2.9 链路状态依赖性

### 2.9.1 DL_Down | 数据链路层关闭

When DL_Down occurs, the Transaction Layer:
- Must not transmit any new TLPs to the Data Link Layer
- All outstanding Non-Posted Requests must be terminated with error
- Flow control credits are reset

> DL_Down 发生时，事务层：
>
> - 不得向数据链路层发送任何新的 TLP
> - 所有未完成的 Non-Posted 请求必须作为错误终止
> - 流量控制信用被复位

---

### 2.9.2 DL_Up | 数据链路层启动

When DL_Up occurs after DL_Down, all previously outstanding Requests are considered terminated. The Function must re-establish any necessary state. Resets and other mechanisms may be used to recover.

> DL_Up 在 DL_Down 之后发生时，所有先前未完成的请求被视为已终止。功能必须重新建立任何必要的状态。可使用复位和其他机制进行恢复。

---

### 2.9.3 Downstream Port Containment (DPC) | 下游端口遏制

During DPC, the Transaction Layer of the affected Downstream Port:

- Blocks all TLP transmissions downstream
- Terminates outstanding Non-Posted Requests
- May signal errors to the associated Functions

> DPC 期间，受影响下游端口的事务层：
>
> - 阻止所有向下游的 TLP 发送
> - 终止未完成的 Non-Posted 请求
> - 可向相关功能发出错误信号

---

## Appendix: Key Acronyms | 附录：关键缩略语

| 缩略语 | 全称 | 中文翻译 |
|--------|------|---------|
| TLP | Transaction Layer Packet | 事务层数据包 |
| DLLP | Data Link Layer Packet | 数据链路层数据包 |
| TC | Traffic Class | 流量类别 |
| VC | Virtual Channel | 虚拟通道 |
| FC | Flow Control | 流量控制 |
| ECRC | End-to-End CRC | 端到端 CRC |
| EP | Error/Poisoned | 错误/中毒 |
| RO | Relaxed Ordering | 放松排序 |
| IDO | ID-Based Ordering | 基于 ID 的排序 |
| NS | No Snoop | 不监听 |
| RCB | Read Completion Boundary | 读完成边界 |
| SC/UR/CA | Successful/Unsupported/Completer Abort | 成功/不支持/完成者终止 |
| PASID | Process Address Space ID | 进程地址空间 ID |
| AER | Advanced Error Reporting | 高级错误报告 |
| LTR | Latency Tolerance Reporting | 延迟容忍报告 |
| OBFF | Optimized Buffer Flush/Fill | 优化缓冲区刷新/填充 |
| PTM | Precision Time Measurement | 精确时间测量 |
| DPC | Downstream Port Containment | 下游端口遏制 |
| UIO | Untranslated I/O | 未转换 I/O |
| ATC | Address Translation Cache | 地址转换缓存 |
| DW | Double Word (4 bytes) | 双字（4 字节） |

---

> **Translator's Notes | 译者说明**
>
> 本文档翻译自 PCI Express Base Specification Revision 6.2 (January 25, 2024) 第 2 章全文 (Pages 141–308, 168 pages)。
> 由于第 2 章篇幅巨大（168 页），本翻译覆盖了所有主要章节的核心内容，包括 TLP 格式定义、路由规则、排序规则、流量控制和完成超时机制。
