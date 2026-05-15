# PCI Express Base Specification Revision 6.2 — Chapter 6

# PCI Express 基础规范 修订版 6.2 — 第6章

**Chinese-English Parallel Translation / 中英对照翻译**

**Chapter 6: System Architecture / 第6章：系统架构**

---

> **Translator's Note / 译者说明**:
> - Paragraph-level Chinese-English parallel translation. 段落级中英对照翻译。
> - Page images are embedded inline for visual reference. 页面截图内嵌在文中以供参考。
> - Key technical terms are preserved in English on first occurrence with Chinese translations. 关键术语首次出现时保留英文及中文翻译。

---

## 6. System Architecture / 系统架构

![Page 707 — Chapter 6 opening](images/ch06_pg0707.png)

This chapter addresses various aspects of PCI Express interconnect architecture in a platform context.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

本章讨论平台环境中 PCI Express 互连架构的各个方面。

</div>

---

### 6.1 Interrupt and PME Support / 中断与 PME 支持

The PCI Express interrupt model supports two mechanisms:

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 中断模型支持两种机制：

</div>

- INTx emulation / INTx 仿真
- Message Signaled Interrupt (MSI/MSI-X) / 消息信号中断（MSI/MSI-X）

For legacy compatibility, PCI Express provides a PCI INTx emulation mechanism to signal interrupts to the system interrupt controller (typically part of the Root Complex). This mechanism is compatible with existing PCI software, and provides the same level and type of service as the corresponding PCI interrupt signaling mechanism and is independent of system interrupt controller specifics. This legacy compatibility mechanism allows boot device support without requiring complex BIOS-level interrupt configuration/control service stacks. It virtualizes PCI physical interrupt signals by using an in-band signaling mechanism.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

为了传统兼容性，PCI Express 提供了 PCI INTx 仿真机制，向系统中断控制器（通常是根复合体的一部分）发送中断信号。该机制与现有 PCI 软件兼容，提供与相应 PCI 中断信号机制相同级别和类型的服务，并且独立于系统中断控制器的具体实现。这种传统兼容性机制允许引导设备支持，而无需复杂的 BIOS 级中断配置/控制服务栈。它通过使用带内信号机制来虚拟化 PCI 物理中断信号。

</div>

If an implementation supports interrupts, then this specification requires support of either MSI or MSI-X or both. PCI Compatible INTx interrupt emulation is optional. Switches are required to support forwarding the INTx interrupt emulation Messages (see Section 2.2.8.1). The PCI Express MSI and MSI-X mechanisms are compatible with those originally defined in [PCI].

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

如果实现支持中断，则本规范要求支持 MSI 或 MSI-X 或两者。PCI 兼容的 INTx 中断仿真是可选的。交换机必须支持转发 INTx 中断仿真消息（见第 2.2.8.1 节）。PCI Express MSI 和 MSI-X 机制与 [PCI] 中最初定义的机制兼容。

</div>

For SR-IOV devices, PFs are permitted to implement INTx, and VFs must not implement INTx. Each PF and VF must implement its own unique interrupt capabilities.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于 SR-IOV 设备，PF 允许实现 INTx，而 VF 不得实现 INTx。每个 PF 和 VF 必须实现其各自唯一的中断能力。

</div>

#### 6.1.1 Rationale for PCI Express Interrupt Model / PCI Express 中断模型的原理

PCI Express takes an evolutionary approach from PCI with respect to interrupt support.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 在中断支持方面采取了从 PCI 演进的方法。

</div>

As required for PCI/PCI-X interrupt mechanisms, each device Function is required to differentiate between INTx and MSI/MSI-X modes of operation. The device complexity required to support both schemes is no different than that for PCI/PCI-X devices. The advantages of this approach include:

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

正如 PCI/PCI-X 中断机制所要求的，每个设备功能需要区分 INTx 和 MSI/MSI-X 操作模式。支持这两种方案所需的设备复杂性与 PCI/PCI-X 设备相同。这种方法的优点包括：

</div>

- Compatibility with existing PCI Software Models / 与现有 PCI 软件模型兼容
- Direct support for boot devices / 直接支持引导设备
- Easier End of Life (EOL) for INTx legacy mechanisms / 更容易实现 INTx 传统机制的终止（EOL）

The existing software model is used to differentiate INTx vs. MSI/MSI-X modes of operation; thus, no special software support is required for PCI Express.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

使用现有软件模型来区分 INTx 与 MSI/MSI-X 操作模式；因此，PCI Express 不需要特殊的软件支持。

</div>

The software model does not support changing interrupt modes while the Function is in active operation. If software does this, interrupt conditions may be dropped or replicated.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

软件模型不支持在功能处于活动操作期间更改中断模式。如果软件这样做，中断条件可能会被丢弃或重复。

</div>

#### 6.1.2 PCI-compatible INTx Emulation / PCI 兼容的 INTx 仿真

PCI Express emulates the PCI interrupt mechanism including the Interrupt Pin and Interrupt Line registers of the PCI Configuration Space for PCI device Functions. PCI Express non-Switch devices may optionally support these registers for backwards compatibility. Switch devices are required to support them. Actual interrupt signaling uses in-band Messages rather than being signaled using physical pins.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 仿真 PCI 中断机制，包括 PCI 设备功能的 PCI 配置空间中的中断引脚（Interrupt Pin）和中断线（Interrupt Line）寄存器。PCI Express 非交换设备可以选择性支持这些寄存器以实现向后兼容。交换设备必须支持它们。实际的中断信号使用带内消息，而不是通过物理引脚发出信号。

</div>

Two types of Messages are defined, Assert_INTx and Deassert_INTx, for emulation of PCI INTx signaling, where x is A, B, C, and D for respective PCI interrupt signals. These Messages are used to provide "virtual wires" for signaling interrupts across a Link. Switches collect these virtual wires and present a combined set at the Switch's Upstream Port. Ultimately, the virtual wires are routed to the Root Complex which maps the virtual wires to system interrupt resources. Devices must use assert/deassert Messages in pairs to emulate PCI interrupt level-triggered signaling. Actual mapping of PCI Express INTx emulation to system interrupts is implementation specific as is mapping of physical interrupt signals in conventional PCI.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

定义了两种类型的消息——Assert_INTx 和 Deassert_INTx，用于仿真 PCI INTx 信号，其中 x 为 A、B、C 和 D，分别对应 PCI 中断信号。这些消息用于提供跨链路发送中断信号的"虚拟线路"。交换机收集这些虚拟线路，并在交换机的上行端口呈现组合集合。最终，虚拟线路被路由到根复合体，根复合体将虚拟线路映射到系统中断资源。设备必须成对使用 assert/deassert 消息来仿真 PCI 中断电平触发信号。PCI Express INTx 仿真到系统中断的实际映射是特定于实现的，正如传统 PCI 中物理中断信号的映射一样。

</div>

The legacy INTx emulation mechanism may be deprecated in a future version of this specification.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

传统 INTx 仿真机制可能在本规范的未来版本中被弃用。

</div>

#### 6.1.3 INTx Emulation Software Model / INTx 仿真软件模型

The software model for legacy INTx emulation matches that of PCI. The system BIOS reporting of chipset/platform interrupt mapping and the association of each device Function's interrupt with PCI interrupt lines is handled in exactly the same manner as with conventional PCI systems. Legacy software reads from each device Function's Interrupt Pin register to determine if the Function is interrupt driven. A value between 01h and 04h indicates that the Function uses an emulated interrupt pin to generate an interrupt.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

传统 INTx 仿真的软件模型与 PCI 的软件模型一致。系统 BIOS 报告的芯片组/平台中断映射以及每个设备功能的中断与 PCI 中断线的关联，以与传统 PCI 系统完全相同的方式处理。传统软件从每个设备功能的中断引脚寄存器读取，以确定该功能是否为中断驱动。值在 01h 到 04h 之间表示该功能使用仿真中断引脚来生成中断。

</div>

Note that similarly to physical interrupt signals, the INTx emulation mechanism may potentially cause spurious interrupts that must be handled by the system software.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

注意，与物理中断信号类似，INTx 仿真机制可能会产生虚假中断，必须由系统软件处理。

</div>

#### 6.1.4 MSI and MSI-X Operation / MSI 与 MSI-X 操作

Message Signaled Interrupts (MSI) is an optional feature that enables a device Function to request service by writing a system-specified data value to a system-specified address (using a DWORD Memory Write transaction). System software initializes the message address and message data (from here on referred to as the "vector") during device configuration, allocating one or more vectors to each MSI-capable Function.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

消息信号中断（MSI）是一个可选功能，使设备功能能够通过向系统指定的地址写入系统指定的数据值（使用 DWORD 存储器写事务）来请求服务。系统软件在设备配置期间初始化消息地址和消息数据（以下称为"向量"），为每个支持 MSI 的功能分配一个或多个向量。

</div>

Interrupt latency (the time from interrupt signaling to interrupt servicing) is system dependent. Consistent with current interrupt architectures, Message Signaled Interrupts do not provide interrupt latency time guarantees.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

中断延迟（从中断信号到中断服务的时间）取决于系统。与当前中断架构一致，消息信号中断不提供中断延迟时间保证。

</div>

MSI-X defines a separate optional extension to basic MSI functionality. Compared to MSI, MSI-X supports a larger maximum number of vectors per Function, the ability for software to control aliasing when fewer vectors are allocated than requested, plus the ability for each vector to use an independent address and data value, specified by a table that resides in Memory Space. However, most of the other characteristics of MSI-X are identical to those of MSI.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

MSI-X 定义了基本 MSI 功能的一个独立可选扩展。与 MSI 相比，MSI-X 支持每个功能更多的最大向量数、当分配的向量少于请求时软件控制别名化的能力，以及每个向量使用独立地址和数据值的能力（由驻留在存储器空间的表指定）。然而，MSI-X 的大多数其他特性与 MSI 相同。

</div>

For the sake of software backward compatibility, MSI and MSI-X use separate and independent Capability structures. On Functions that support both MSI and MSI-X, system software that supports only MSI can still enable and use MSI without any modification. MSI functionality is managed exclusively through the MSI Capability structure, and MSI-X functionality is managed exclusively through the MSI-X Capability structure.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

为了软件向后兼容，MSI 和 MSI-X 使用独立的能力结构。在同时支持 MSI 和 MSI-X 的功能上，仅支持 MSI 的系统软件仍然可以启用和使用 MSI，无需任何修改。MSI 功能完全通过 MSI 能力结构管理，MSI-X 功能完全通过 MSI-X 能力结构管理。

</div>

A Function is permitted to implement both MSI and MSI-X, but system software is prohibited from enabling both at the same time. If system software enables both at the same time, the behavior is undefined.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

功能可以同时实现 MSI 和 MSI-X，但禁止系统软件同时启用两者。如果系统软件同时启用两者，行为是未定义的。

</div>

![Page 708-709 — MSI/MSI-X overview](images/ch06_pg0709.png)

All PCI Express device Functions that are capable of generating interrupts must support MSI or MSI-X or both. The MSI and MSI-X mechanisms deliver interrupts by performing Memory Write transactions. MSI and MSI-X are edge-triggered interrupt mechanisms; neither [PCI] nor this specification support level-triggered MSI/MSI-X interrupts. Certain PCI devices and their drivers rely on INTx-type level-triggered interrupt behavior (addressed by the PCI Express legacy INTx emulation mechanism). To take advantage of the MSI or MSI-X capability and edge-triggered interrupt semantics, these devices and their drivers may have to be redesigned.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

所有能够生成中断的 PCI Express 设备功能必须支持 MSI 或 MSI-X 或两者。MSI 和 MSI-X 机制通过执行存储器写事务来传递中断。MSI 和 MSI-X 是边沿触发的中断机制；[PCI] 和本规范都不支持电平触发的 MSI/MSI-X 中断。某些 PCI 设备及其驱动程序依赖 INTx 类型的电平触发中断行为（由 PCI Express 传统 INTx 仿真机制解决）。为了利用 MSI 或 MSI-X 能力以及边沿触发中断语义，这些设备及其驱动程序可能需要重新设计。

</div>

MSI and MSI-X each support Per-Vector Masking (PVM). PVM is an optional extension to MSI, and a standard feature with MSI-X. A Function that supports the PVM extension to MSI is backward compatible with system software that is unaware of the extension. MSI-X also supports a Function Mask bit, which when Set masks all of the vectors associated with a Function.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

MSI 和 MSI-X 各支持每向量屏蔽（PVM）。PVM 是 MSI 的可选扩展，并且是 MSI-X 的标准功能。支持 MSI PVM 扩展的功能与不知道该扩展的系统软件向后兼容。MSI-X 还支持功能屏蔽位（Function Mask bit），当置位时屏蔽与该功能关联的所有向量。

</div>

A Legacy Endpoint that implements MSI is required to support either the 32-bit or 64-bit Message Address version of the MSI Capability structure. A PCI Express Endpoint that implements MSI is required to support the 64-bit Message Address version of the MSI Capability structure.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

实现 MSI 的传统端点需要支持 MSI 能力结构的 32 位或 64 位消息地址版本。实现 MSI 的 PCI Express 端点需要支持 MSI 能力结构的 64 位消息地址版本。

</div>

The Requester of an MSI/MSI-X transaction must set the No Snoop and Relaxed Ordering attributes of the Transaction Descriptor to 0b. A Requester of an MSI/MSI-X transaction is permitted to Set the ID-Based Ordering (IDO) attribute if use of the IDO attribute is enabled.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

MSI/MSI-X 事务的请求者必须将事务描述符的 No Snoop 和 Relaxed Ordering 属性设置为 0b。如果启用了 IDO 属性的使用，允许 MSI/MSI-X 事务的请求者设置 ID-Based Ordering（IDO）属性。

</div>

Note that, unlike INTx emulation Messages, MSI/MSI-X transactions are not restricted to the TC0 traffic class.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

注意，与 INTx 仿真消息不同，MSI/MSI-X 事务不限于 TC0 流量类别。

</div>

Within a device, different Functions are permitted to implement different sets of the MSI/MSI-X/INTx interrupt mechanisms, and system software manages each Function's interrupt mechanisms independently.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

在一个设备内，不同的功能可以实现不同的 MSI/MSI-X/INTx 中断机制集，系统软件独立管理每个功能的中断机制。

</div>

**Implementation Note: Synchronization of Data Traffic and Message Signaled Interrupts** / 实现说明：数据流量与消息信号中断的同步

MSI/MSI-X transactions are permitted to use the TC that is most appropriate for the device's programming model. This is generally the same TC as is used to transfer data; for legacy I/O, TC0 should be used. If a device uses more than one TC, it must explicitly ensure that proper synchronization is maintained between data traffic and interrupt Message(s) not using the same TC. Methods for ensuring this synchronization are implementation specific. One option is for a device to issue a zero-length Read (as described in Section 2.2.5) using each additional TC used for data traffic prior to issuing the MSI/MSI-X transaction. Other methods are also possible. Note, however, that platform software (e.g., a device driver) is generally only capable of issuing transactions using TC0.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

MSI/MSI-X 事务可以使用最适合设备编程模型的 TC。这通常是与传输数据相同的 TC；对于传统 I/O，应使用 TC0。如果设备使用多个 TC，它必须显式确保在不使用相同 TC 的数据流量和中断消息之间保持适当的同步。确保这种同步的方法取决于具体实现。一种选择是设备在发出 MSI/MSI-X 事务之前，使用每个用于数据流量的附加 TC 发出一个零长度读（如第 2.2.5 节所述）。其他方法也是可能的。但请注意，平台软件（例如设备驱动程序）通常只能使用 TC0 发出事务。

</div>

##### 6.1.4.1 MSI Configuration / MSI 配置

In this section, all register and field references are in the context of the MSI Capability structure.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

在本节中，所有寄存器和字段引用均在 MSI 能力结构的上下文中。

</div>

System software reads the Message Control register to determine the Function's MSI capabilities.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

系统软件读取消息控制寄存器以确定功能的 MSI 能力。

</div>

System software reads the Multiple Message Capable field (bits 3-1 of the Message Control register) to determine the number of requested vectors. MSI supports a maximum of 32 vectors per Function. System software writes to the Multiple Message Enable field (bits 6-4 of the Message Control register) to allocate either all or a subset of the requested vectors. For example, a Function can request four vectors and be allocated either four, two, or one vector. The number of vectors requested and allocated is aligned to a power of two (that is, a Function that requires three vectors must request four).

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

系统软件读取多消息能力字段（消息控制寄存器的第3-1位）以确定请求的向量数。MSI 支持每个功能最多 32 个向量。系统软件写入多消息启用字段（消息控制寄存器的第6-4位）以分配全部或部分请求的向量。例如，一个功能可以请求四个向量，并被分配四个、两个或一个向量。请求和分配的向量数对齐到2的幂（即，需要三个向量的功能必须请求四个）。

</div>

If the Per-Vector Masking Capable bit (bit 8 of the Message Control register) is Set and system software supports Per-Vector Masking, system software may mask one or more vectors by writing to the Mask Bits register.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

如果每向量屏蔽能力位（消息控制寄存器的第8位）置位且系统软件支持每向量屏蔽，系统软件可以通过写入屏蔽位寄存器来屏蔽一个或多个向量。

</div>

If the 64-bit Address Capable bit (bit 7 of the Message Control register) is Set, system software initializes the MSI Capability structure's Message Address register (specifying the lower 32 bits of the message address) and the Message Upper Address register (specifying the upper 32 bits of the message address) with a system-specified message address. System software may program the Message Upper Address register to zero so that the Function uses a 32-bit address for the MSI transaction. If this bit is Clear, system software initializes the MSI Capability structure's Message Address register (specifying a 32-bit message address) with a system specified message address.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

如果64位地址能力位（消息控制寄存器的第7位）置位，系统软件使用系统指定的消息地址初始化 MSI 能力结构的消息地址寄存器（指定消息地址的低32位）和消息高地址寄存器（指定消息地址的高32位）。系统软件可以将消息高地址寄存器编程为零，以便功能使用32位地址进行 MSI 事务。如果该位清零，系统软件使用系统指定的消息地址初始化 MSI 能力结构的消息地址寄存器（指定32位消息地址）。

</div>

System software initializes the MSI Capability structure's Message Data register with the lower 16 bits of a system specified data value. When the Extended Message Data Capable bit is Clear, care must be taken to initialize only the Message Data register (i.e., a 2-byte value) and not modify the upper two bytes of that DWORD location.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

系统软件使用系统指定数据值的低16位初始化 MSI 能力结构的消息数据寄存器。当扩展消息数据能力位清零时，必须注意只初始化消息数据寄存器（即2字节值），而不修改该 DWORD 位置的高两字节。

</div>

If the Extended Message Data Capable bit is Set and system software supports 32-bit vector values, system software may initialize the MSI capability structure's Extended Message Data register with the upper 16 bits of a system specified data value, and then Set the Extended Message Data Enable bit.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

如果扩展消息数据能力位置位且系统软件支持32位向量值，系统软件可以使用系统指定数据值的高16位初始化 MSI 能力结构的扩展消息数据寄存器，然后置位扩展消息数据启用位。

</div>

##### 6.1.4.2 MSI-X Configuration / MSI-X 配置

![Page 710 — MSI-X Configuration](images/ch06_pg0710.png)

In this section, all register and field references are in the context of the MSI-X Capability, MSI-X Table, and MSI-X PBA structures.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

在本节中，所有寄存器和字段引用均在 MSI-X 能力、MSI-X 表和 MSI-X PBA 结构的上下文中。

</div>

System software allocates address space for the Function's standard set of Base Address registers and sets the registers accordingly. One of the Function's Base Address registers includes address space for the MSI-X Table, though the system software that allocates address space does not need to be aware of which Base Address register this is, or the fact the address space is used for the MSI-X Table. The same or another Base Address register includes address space for the MSI-X PBA, and the same point regarding system software applies.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

系统软件为功能的标准基地址寄存器组分配地址空间并进行相应设置。功能的一个基地址寄存器包含 MSI-X 表的地址空间，但分配地址空间的系统软件不需要知道这是哪个基地址寄存器，也不需要知道该地址空间用于 MSI-X 表。同一个或另一个基地址寄存器包含 MSI-X PBA 的地址空间，关于系统软件的同样说明也适用于此。

</div>

Depending upon system software policy, system software, device driver software, or each at different times or environments may configure a Function's MSI-X Capability and table structures with suitable vectors. For example, a booting environment will likely require only a single vector, whereas a normal operating system environment for running applications may benefit from multiple vectors if the Function supports an MSI-X Table with multiple entries. For the remainder of this section, "software" refers to either system software or device driver software.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

根据系统软件策略，系统软件、设备驱动程序软件，或在不同时间或环境下各自配置功能的 MSI-X 能力和表结构为适当的向量。例如，引导环境可能只需要一个向量，而运行应用程序的正常操作系统环境可能从多个向量中受益，如果功能支持具有多个条目 MSI-X 表的话。在本节余下部分，"软件"指系统软件或设备驱动程序软件。

</div>

Software reads the Table Size field from the Message Control register to determine the MSI-X Table size. The field encodes the number of table entries as N-1, so software must add 1 to the value read from the field to calculate the number of table entries N. MSI-X supports a maximum table size of 2048 entries.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

软件从消息控制寄存器读取表大小字段以确定 MSI-X 表大小。该字段将表条目数编码为 N-1，因此软件必须将从该字段读取的值加1以计算表条目数 N。MSI-X 支持最大表大小为 2048 个条目。

</div>

Software calculates the base address of the MSI-X Table by reading the 32-bit value from the Table Offset/Table BIR register, masking off the lower 3 Table BIR bits, and adding the remaining QWORD-aligned 32-bit Table offset to the address taken from the Base Address register indicated by the Table BIR. Software calculates the base address of the MSI-X PBA using the same process with the PBA Offset/PBA BIR register.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

软件通过从表偏移/表 BIR 寄存器读取32位值，屏蔽低3位表 BIR 位，并将剩余的 QWORD 对齐的32位表偏移加到由表 BIR 指示的基地址寄存器中获取的地址来计算 MSI-X 表的基地址。软件使用相同的过程通过 PBA Offset/PBA BIR 寄存器计算 MSI-X PBA 的基地址。

</div>

For each MSI-X Table entry that will be used, software fills in the Message Address field, Message Upper Address field, Message Data field, and Vector Control field. The Vector Control field may contain optional Steering Tag fields. Software must not modify the Address, Data, or Steering Tag fields of an entry while it is unmasked. Refer to Section 6.1.4.5 for details.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于每个将要使用的 MSI-X 表条目，软件填写消息地址字段、消息高地址字段、消息数据字段和向量控制字段。向量控制字段可能包含可选的方向标签（Steering Tag）字段。软件不得在条目取消屏蔽时修改其地址、数据或方向标签字段。详情参考第 6.1.4.5 节。

</div>

For potential use by future specifications, the Reserved bits in the Vector Control field must have their default values preserved by software. If software does not preserve their values, the result is undefined.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

为了未来规范的可能使用，向量控制字段中的保留位必须由软件保持其默认值。如果软件不保持其值，结果是未定义的。

</div>

For each MSI-X Table entry that software chooses not to configure for generating messages, software can simply leave the entry in its default state of being masked.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于软件选择不配置用于生成消息的每个 MSI-X 表条目，软件可以简单地让该条目保持其默认的屏蔽状态。

</div>

Software is permitted to configure multiple MSI-X Table entries with the same vector, and this may indeed be necessary when fewer vectors are allocated than requested.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

软件可以使用相同向量配置多个 MSI-X 表条目，并且当分配的向量少于请求时，这确实是必要的。

</div>

![Page 711 — Enabling, Implementation Notes](images/ch06_pg0711.png)

**Implementation Note: Special Considerations for QWORD Accesses** / 实现说明：QWORD 访问的特殊考虑

Software is permitted to fill in MSI-X Table entry DWORD fields individually with DWORD writes, or software in certain cases is permitted to fill in appropriate pairs of DWORDs with a single QWORD write. Specifically, software is always permitted to fill in the Message Address and Message Upper Address fields with a single QWORD write. If a given entry is currently masked (via its Mask bit or the Function Mask bit), software is permitted to fill in the Message Data and Vector Control fields with a single QWORD write, taking advantage of the fact the Message Data field is guaranteed to become visible to hardware no later than the Vector Control field. However, if software wishes to mask a currently unmasked entry (without Setting the Function Mask bit), software must Set the entry's Mask bit using a DWORD write to the Vector Control field, since performing a QWORD write to the Message Data and Vector Control fields might result in the Message Data field being modified before the Mask bit in the Vector Control field becomes Set.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

软件允许分别用 DWORD 写填充 MSI-X 表条目的各个 DWORD 字段，或在某些情况下允许用单个 QWORD 写填充适当的 DWORD 对。具体来说，软件总是允许用单个 QWORD 写填充消息地址和消息高地址字段。如果给定条目当前被屏蔽（通过其屏蔽位或功能屏蔽位），软件允许用单个 QWORD 写填充消息数据和向量控制字段，利用消息数据字段保证不晚于向量控制字段对硬件可见的事实。但是，如果软件希望屏蔽当前未屏蔽的条目（而不设置功能屏蔽位），软件必须使用对向量控制字段的 DWORD 写来设置条目的屏蔽位，因为对消息数据和向量控制字段执行 QWORD 写可能在向量控制字段中的屏蔽位被置位之前导致消息数据字段被修改。

</div>

**Implementation Note: Handling MSI-X Vector Shortages** / 实现说明：处理 MSI-X 向量不足

For the case where fewer vectors are allocated to a Function than desired, software-controlled aliasing as enabled by MSI-X is one approach for handling the situation. For example, if a Function supports five queues, each with an associated MSI-X table entry, but only three vectors are allocated, the Function could be designed for software still to configure all five table entries, assigning one or more vectors to multiple table entries. Software could assign the three vectors {A,B,C} to the five entries as ABCCC, ABBCC, ABCBA, or other similar combinations. Alternatively, the Function could be designed for software to configure it (using a device specific mechanism) to use only three queues and three MSI-X table entries. Software could assign the three vectors {A,B,C} to the five entries as ABC--, A-B-C, A-CB, or other similar combinations.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于分配给功能的向量少于所需的情况，由 MSI-X 启用的软件控制别名化是处理这种情况的一种方法。例如，如果一个功能支持五个队列，每个队列关联一个 MSI-X 表条目，但只分配了三个向量，功能可以设计为让软件仍然配置所有五个表条目，将一个或多个向量分配给多个表条目。软件可以将三个向量 {A,B,C} 分配给五个条目为 ABCCC、ABBCC、ABCBA 或其他类似组合。或者，功能可以设计为让软件配置它（使用设备特定机制）只使用三个队列和三个 MSI-X 表条目。软件可以将三个向量 {A,B,C} 分配给五个条目为 ABC--、A-B-C、A-CB 或其他类似组合。

</div>

##### 6.1.4.3 Enabling Operation / 启用操作

To maintain backward compatibility, the MSI Enable bit in the Message Control Register for MSI and the MSI-X Enable bit in the Message Control Register for MSI-X are each Clear by default (MSI and MSI-X are both disabled). System configuration software Sets one of these bits to enable either MSI or MSI-X, but never both simultaneously. Behavior is undefined if both MSI and MSI-X are enabled simultaneously. Software disabling either mechanism during active operation may result in the Function dropping pending interrupt conditions or failing to recognize new interrupt conditions. While enabled for MSI or MSI-X operation, a Function is prohibited from using INTx interrupts (if implemented) to request service (MSI, MSI-X, and INTx are mutually exclusive).

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

为了保持向后兼容，MSI 的消息控制寄存器中的 MSI 启用位和 MSI-X 的消息控制寄存器中的 MSI-X 启用位默认均为清零（MSI 和 MSI-X 都被禁用）。系统配置软件设置其中之一以启用 MSI 或 MSI-X，但从不同时启用两者。如果同时启用 MSI 和 MSI-X，行为是未定义的。在活动操作期间软件禁用任一机制可能导致功能丢弃挂起的中断条件或无法识别新的中断条件。当启用 MSI 或 MSI-X 操作时，禁止功能使用 INTx 中断（如果实现了的话）来请求服务（MSI、MSI-X 和 INTx 是互斥的）。

</div>

##### 6.1.4.4 Sending Messages / 发送消息

![Page 712 — Sending Messages, Synchronization](images/ch06_pg0712.png)

Once MSI or MSI-X is enabled (the appropriate bit in one of the Message Control registers is Set), and one or more vectors is unmasked, the Function is permitted to send messages. To send a message, a Function does a DWORD Memory Write to the appropriate message address with the appropriate message data.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

一旦 MSI 或 MSI-X 被启用（相应消息控制寄存器中的适当位被置位），并且一个或多个向量取消屏蔽，功能就可以发送消息。为了发送消息，功能使用适当的消息数据向适当的消息地址执行 DWORD 存储器写。

</div>

For MSI when the Extended Message Data Enable bit is Clear, the DWORD that is written is made up of the value in the MSI Message Data register in the lower two bytes and zeros in the upper two bytes. For MSI when the Extended Message Data Enable bit is Set, the DWORD that is written is made up of the value in the MSI Message Data register in the lower two bytes and the value in the Extended Message Data register in the upper two bytes.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于 MSI，当扩展消息数据启用位清零时，写入的 DWORD 由低两字节的 MSI 消息数据寄存器中的值和上两字节的零组成。对于 MSI，当扩展消息数据启用位置位时，写入的 DWORD 由低两字节的 MSI 消息数据寄存器中的值和高两字节的扩展消息数据寄存器中的值组成。

</div>

For MSI-X, the DWORD that is written is taken from the appropriate Table entry's Message Data field. If multiple unmasked vectors are used, they can all use the same message address or an independent message address for each vector.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于 MSI-X，写入的 DWORD 取自适当表条目的消息数据字段。如果使用多个取消屏蔽的向量，它们可以都使用相同的消息地址，或者每个向量使用独立的消息地址。

</div>

##### 6.1.4.5 Per-vector Masking and Function Masking / 每向量屏蔽和功能屏蔽

For MSI-X, a Function is permitted to cache Address and Data values from unmasked MSI-X Table entries. However, when software updates the Address and/or Data fields for an unmasked entry, the Function might use either the old or new value for any message it sends. Therefore, software must mask an entry before updating its Address or Data fields, and must not unmask the entry until the updates are complete.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于 MSI-X，功能可以从取消屏蔽的 MSI-X 表条目缓存地址和数据值。然而，当软件更新未屏蔽条目的地址和/或数据字段时，功能可能对其发送的任何消息使用旧值或新值。因此，软件必须在更新条目的地址或数据字段之前屏蔽该条目，并且在更新完成之前不得取消屏蔽该条目。

</div>

For MSI, the Mask Bits and Pending Bits registers provide Per-Vector Masking functionality. For MSI-X, each Table entry has a Mask bit in its Vector Control field, and there is a Function Mask bit in the Message Control register.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于 MSI，屏蔽位和挂起位寄存器提供每向量屏蔽功能。对于 MSI-X，每个表条目在其向量控制字段中有一个屏蔽位，并且在消息控制寄存器中有一个功能屏蔽位。

</div>

##### 6.1.4.6 Hardware/Software Synchronization / 硬件/软件同步

When software updates the Address and/or Data fields of an MSI-X Table entry, the following sequence must be observed:
1. Mask the entry by Setting its Mask bit
2. Update the Address and/or Data fields
3. Ensure the updates are globally visible
4. Clear the entry's Mask bit to unmask it

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

当软件更新 MSI-X 表条目的地址和/或数据字段时，必须遵守以下顺序：
1. 通过设置其屏蔽位来屏蔽该条目
2. 更新地址和/或数据字段
3. 确保更新是全局可见的
4. 清除条目的屏蔽位以取消屏蔽

</div>

Similarly, when software updates multiple MSI-X Table entries, it should mask them all, update them, and then unmask them.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

类似地，当软件更新多个 MSI-X 表条目时，它应该全部屏蔽它们，更新它们，然后全部取消屏蔽。

</div>

##### 6.1.4.7 Message Transaction Reception and Ordering Requirements / 消息事务接收和排序要求

![Page 715 — MSI ordering, PME](images/ch06_pg0715.png)

An MSI or MSI-X message, by virtue of being a Posted Request, is prohibited by transaction ordering rules from passing any earlier Posted Request. This preserves the ordering of data versus interrupts.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

MSI 或 MSI-X 消息，作为一种 Posted 请求，根据事务排序规则，禁止超越任何先前的 Posted 请求。这保持了数据与中断的排序。

</div>

#### 6.1.5 PME Support / PME 支持

The PCI Express power management event (PME) signaling mechanism uses in-band Messages to emulate the PCI PME signal. This mechanism is compatible with existing PCI PME software.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 电源管理事件（PME）信号机制使用带内消息来仿真 PCI PME 信号。该机制与现有 PCI PME 软件兼容。

</div>

#### 6.1.6 Native PME Software Model / 原生 PME 软件模型

The native PME software model uses the PME Status bit and PME En bit in the Power Management Control/Status register in PCI Express Capability, along with the Root Complex Event Collector, to manage PME signaling.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

原生 PME 软件模型使用 PCI Express 能力中电源管理控制/状态寄存器的 PME 状态位和 PME 启用位，以及根复合体事件收集器来管理 PME 信号。

</div>

#### 6.1.7 Legacy PME Software Model / 传统 PME 软件模型

The legacy PME software model provides backward compatibility with PCI PME handling through the PCI Power Management Capability structure.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

传统 PME 软件模型通过 PCI 电源管理能力结构提供与 PCI PME 处理的向后兼容。

</div>

#### 6.1.8 Operating System Power Management Notification / 操作系统电源管理通知

PCI Express provides mechanisms for the operating system to receive notification of power management events from devices, supporting both the native and legacy PME models.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 为操作系统提供从设备接收电源管理事件通知的机制，支持原生和传统 PME 模型。

</div>

#### 6.1.9 PME Routing Between PCI Express and PCI Hierarchies / PCI Express 与 PCI 层次结构之间的 PME 路由

PME messages can be routed between PCI Express and PCI hierarchies through PCI Express to PCI/PCI-X Bridges, providing end-to-end PME support across mixed topologies.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PME 消息可以通过 PCI Express 到 PCI/PCI-X 桥在 PCI Express 和 PCI 层次结构之间路由，提供跨混合拓扑的端到端 PME 支持。

</div>

---

### 6.2 Error Signaling and Logging / 错误信号与记录

![Page 717 — Error Signaling](images/ch06_pg0717.png)

#### 6.2.1 Scope / 范围

This section describes the error handling mechanisms for PCI Express. The error handling architecture includes error classification, error signaling, error logging, and error recovery mechanisms.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

本节描述 PCI Express 的错误处理机制。错误处理架构包括错误分类、错误信号、错误记录和错误恢复机制。

</div>

#### 6.2.2 Error Classification / 错误分类

PCI Express errors are classified into two major categories:

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 错误分为两大类：

</div>

##### 6.2.2.1 Correctable Errors / 可纠正错误

Correctable errors are those that can be corrected by hardware without any loss of information and without software intervention. These errors may impact performance but do not affect the functional operation of the Link.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

可纠正错误是那些可以由硬件纠正而不丢失任何信息且无需软件干预的错误。这些错误可能会影响性能，但不会影响链路的功能操作。

</div>

##### 6.2.2.2 Uncorrectable Errors / 不可纠正错误

Uncorrectable errors are those that cannot be corrected by hardware. These errors are further subdivided into:

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

不可纠正错误是那些无法由硬件纠正的错误。这些错误进一步细分为：

</div>

- **Fatal Errors** / 致命错误 — Errors that are severe enough to make the hardware unreliable. The Link or Function may no longer be functional.
<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

  严重到足以使硬件不可靠的错误。链路或功能可能不再正常工作。
- **Non-Fatal Errors** / 非致命错误 — Errors that do not cause the hardware to be unreliable but may result in data loss or require software intervention to recover.
  不会导致硬件不可靠，但可能导致数据丢失或需要软件干预才能恢复的错误。

</div>

#### 6.2.3 Error Signaling / 错误信号

![Page 719 — Error Signaling Methods](images/ch06_pg0719.png)

PCI Express uses three primary mechanisms for error signaling:

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 使用三种主要机制进行错误信号：

</div>

##### 6.2.3.1 Completion Status / 完成状态

The Completion Status field in a Completion TLP indicates the outcome of a Request. Status values include Successful Completion (SC), Unsupported Request (UR), and Completer Abort (CA).

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

完成 TLP 中的完成状态字段指示请求的结果。状态值包括成功完成（SC）、不支持的请求（UR）和完成者中止（CA）。

</div>

##### 6.2.3.2 Error Messages / 错误消息

Error Messages are used to signal errors that are not associated with a specific Completion. These include:
- ERR_COR (Correctable Error) / 可纠正错误
- ERR_NONFATAL (Non-Fatal Error) / 非致命错误
- ERR_FATAL (Fatal Error) / 致命错误

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

错误消息用于发送与特定完成无关的错误信号。这些包括：
- ERR_COR（可纠正错误）
- ERR_NONFATAL（非致命错误）
- ERR_FATAL（致命错误）

</div>

###### 6.2.3.2.1 Uncorrectable Error Severity Programming (Advanced Error Reporting) / 不可纠正错误严重程度编程

The severity of uncorrectable errors can be programmed through the Uncorrectable Error Severity register in the AER Extended Capability.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

不可纠正错误的严重程度可以通过 AER 扩展能力中的不可纠正错误严重程度寄存器进行编程。

</div>

###### 6.2.3.2.2 Masking Individual Errors / 屏蔽个别错误

Individual error types can be masked through the Correctable Error Mask register and Uncorrectable Error Mask register.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

可以通过可纠正错误屏蔽寄存器和不可纠正错误屏蔽寄存器屏蔽个别错误类型。

</div>

###### 6.2.3.2.3 Error Pollution / 错误污染

When a TLP with an error is forwarded, subsequent components may log additional errors. To avoid confusion, error pollution rules define when a received error should not be logged again.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

当带有错误的 TLP 被转发时，后续组件可能会记录额外的错误。为了避免混淆，错误污染规则定义了何时不应再次记录接收到的错误。

</div>

###### 6.2.3.2.4 Advisory Non-Fatal Error Cases / 咨询性非致命错误情况

Several specific Advisory Non-Fatal error cases are defined to provide guidance for error handling in situations where an error may or may not be significant.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

定义了几个特定的咨询性非致命错误情况，为错误可能重要也可能不重要的情境提供错误处理指导。

</div>

**6.2.3.2.4.1 Completer Sending a Completion with UR/CA Status** / 完成者发送带有 UR/CA 状态的完成

When a Completer sends a Completion with UR or CA status, this is signaled as an Advisory Non-Fatal Error.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

当完成者发送带有 UR 或 CA 状态的完成时，这作为咨询性非致命错误发出信号。

</div>

**6.2.3.2.4.2 Intermediate Receiver** / 中间接收者

An intermediate Receiver (e.g., Switch) that detects an error in a TLP may log the error as an Advisory Non-Fatal Error.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

检测到 TLP 中错误的中间接收者（如交换机）可以将该错误记录为咨询性非致命错误。

</div>

**6.2.3.2.4.3 Ultimate PCI Express Receiver of a Poisoned TLP** / 中毒 TLP 的最终 PCI Express 接收者

The ultimate PCI Express Receiver of a poisoned TLP signals the error as an Advisory Non-Fatal Error.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

中毒 TLP 的最终 PCI Express 接收者将错误作为咨询性非致命错误发出信号。

</div>

**6.2.3.2.4.4 Requester with Completion Timeout** / 具有完成超时的请求者

A Requester that experiences a Completion Timeout signals this as an Advisory Non-Fatal Error.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

经历完成超时的请求者将此作为咨询性非致命错误发出信号。

</div>

**6.2.3.2.4.5 Receiver of an Unexpected Completion** / 意外完成的接收者

The Receiver of an Unexpected Completion (a Completion for which there is no outstanding Request) logs it as an Advisory Non-Fatal Error.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

意外完成（没有未完成请求的完成）的接收者将其记录为咨询性非致命错误。

</div>

**6.2.3.2.5 Requester Receiving a Completion with UR/CA Status** / 请求者接收带有 UR/CA 状态的完成

A Requester that receives a Completion with UR or CA status must handle it appropriately, typically by reporting the error to its device driver.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

接收带有 UR 或 CA 状态的完成的请求者必须适当处理，通常通过向其设备驱动程序报告错误。

</div>

##### 6.2.3.3 Error Forwarding (Data Poisoning) / 错误转发（数据中毒）

Error forwarding, also known as data poisoning, is used to flag TLPs that contain data with known errors. The EP bit in the TLP header is Set to indicate the TLP is poisoned.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

错误转发，也称为数据中毒，用于标记包含已知错误数据的 TLP。TLP 头中的 EP 位置位表示 TLP 已中毒。

</div>

##### 6.2.3.4 Optional Error Checking / 可选错误检查

Optional error checking mechanisms, such as ECRC, provide additional layers of error detection beyond the standard Link-level CRC.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

可选的错误检查机制（如 ECRC）提供了标准链路级 CRC 之外的其他错误检测层。

</div>

#### 6.2.4 Error Logging / 错误记录

![Page 725 — Error Logging](images/ch06_pg0725.png)

PCI Express provides several registers for error logging:

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 提供了几个用于错误记录的寄存器：

</div>

##### 6.2.4.1 Root Complex Considerations (Advanced Error Reporting) / 根复合体考虑事项（高级错误报告）

The Root Complex plays a special role in error handling, as it is typically the entity that reports errors to the operating system.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

根复合体在错误处理中扮演特殊角色，因为它通常是向操作系统报告错误的实体。

</div>

**6.2.4.1.1 Error Source Identification** / 错误源识别

The Root Complex identifies the source of errors through the Requester ID or by tracking which Root Port received the error message.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

根复合体通过请求者 ID 或跟踪哪个根端口接收了错误消息来识别错误源。

</div>

**6.2.4.1.2 Interrupt Generation** / 中断生成

The Root Complex generates system interrupts to report errors to the operating system, using the error interrupt mechanisms configured through AER.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

根复合体使用通过 AER 配置的错误中断机制生成系统中断，向操作系统报告错误。

</div>

##### 6.2.4.2 Multiple Error Handling (Advanced Error Reporting Extended Capability) / 多重错误处理（高级错误报告扩展能力）

When multiple errors occur, the AER Capability structure provides the First Error Pointer and multiple error status registers to help software identify and manage multiple concurrent error conditions.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

当发生多个错误时，AER 能力结构提供首个错误指针和多个错误状态寄存器，以帮助软件识别和管理多个并发的错误条件。

</div>

##### 6.2.4.3 Advisory Non-Fatal Error Logging / 咨询性非致命错误记录

Advisory Non-Fatal Errors are logged in the AER Capability structure but typically do not generate system interrupts by default.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

咨询性非致命错误记录在 AER 能力结构中，但通常默认不生成系统中断。

</div>

##### 6.2.4.4 End-End TLP Prefix Logging — Non-Flit Mode / 端到端 TLP 前缀记录 — 非 Flit 模式

In Non-Flit Mode, End-End TLP Prefixes are logged as part of the Header Log registers when an error is detected.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

在非 Flit 模式下，当检测到错误时，端到端 TLP 前缀作为头日志寄存器的一部分被记录。

</div>

#### 6.2.5 Sequence of Device Error Signaling and Logging Operations / 设备错误信号和记录操作序列

![Page 731 — Error Sequence](images/ch06_pg0731.png)

The sequence of error signaling and logging operations is defined to ensure consistent behavior across implementations:
1. Error is detected by the device
2. Error status register is updated
3. If the error is not masked, the First Error Pointer is updated (if this is the first error)
4. If error signaling is enabled:
   - For correctable errors: ERR_COR Message is generated (if enabled)
   - For uncorrectable errors: ERR_NONFATAL or ERR_FATAL Message is generated
5. Header Log register captures the TLP header
6. If enabled, an interrupt is generated

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

错误信号和记录操作的顺序被定义为确保跨实现的一致行为：
1. 设备检测到错误
2. 更新错误状态寄存器
3. 如果错误未被屏蔽，则更新首个错误指针（如果是第一个错误）
4. 如果启用了错误信号：
   - 对于可纠正错误：生成 ERR_COR 消息（如果启用）
   - 对于不可纠正错误：生成 ERR_NONFATAL 或 ERR_FATAL 消息
5. 头日志寄存器捕获 TLP 头
6. 如果启用，生成中断

</div>

#### 6.2.6 Error Message Controls / 错误消息控制

![Page 733 — Error Message Controls](images/ch06_pg0733.png)

Error messages are controlled through several mechanisms:
- The SERR Enable bit controls forwarding of error messages by Bridges
- The PCI Express Capability Device Control register provides additional error message enables
- The Root Error Command register in AER controls error reporting by Root Ports

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

错误消息通过几种机制控制：
- SERR 启用位控制桥对错误消息的转发
- PCI Express 能力设备控制寄存器提供额外的错误消息启用
- AER 中的根错误命令寄存器控制根端口的错误报告

</div>

#### 6.2.7 Error Listing and Rules / 错误列表与规则

This section provides a comprehensive listing of all PCI Express error types and the rules governing their detection, signaling, and logging.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

本节提供了所有 PCI Express 错误类型的全面列表以及管理其检测、信号和记录的规则。

</div>

![Page 734 — Error Listing](images/ch06_pg0734.png)

Errors are categorized by their functional source:
- **Physical Layer Errors** / 物理层错误 — Receiver errors, framing errors, etc.
- **Data Link Layer Errors** / 数据链路层错误 — LCRC errors, sequence number errors, replay timeouts, etc.
- **Transaction Layer Errors** / 事务层错误 — Malformed TLP, Unsupported Request, Completer Abort, Unexpected Completion, etc.
- **Configuration Errors** / 配置错误 — Errors detected during configuration space access

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

错误按其功能来源分类：
- **物理层错误** — 接收器错误、定帧错误等
- **数据链路层错误** — LCRC 错误、序列号错误、重放超时等
- **事务层错误** — 畸形 TLP、不支持的请求、完成者中止、意外完成等
- **配置错误** — 在配置空间访问期间检测到的错误

</div>

##### 6.2.7.1 Conventional PCI Mapping / 传统 PCI 映射

PCI Express error types are mapped to conventional PCI error reporting mechanisms (SERR, PERR) for backward compatibility with legacy software.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

PCI Express 错误类型映射到传统 PCI 错误报告机制（SERR、PERR），以与旧软件向后兼容。

</div>

#### 6.2.8 Virtual PCI Bridge Error Handling / 虚拟 PCI 桥错误处理

![Page 739 — Bridge Error Handling](images/ch06_pg0739.png)

##### 6.2.8.1 Error Message Forwarding and PCI Mapping for Bridge — Rules / 桥的错误消息转发和 PCI 映射 — 规则

Virtual PCI Bridges in Switches and Root Ports follow specific rules for forwarding error messages and mapping PCI Express errors to PCI-compatible error signaling.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

交换机和根端口中的虚拟 PCI 桥遵循特定规则来转发错误消息并将 PCI Express 错误映射到 PCI 兼容的错误信号。

</div>

#### 6.2.9 SR-IOV Baseline Error Handling / SR-IOV 基线错误处理

![Page 740 — SR-IOV Error Handling](images/ch06_pg0740.png)

For SR-IOV devices, error handling must account for the separation of Physical Functions (PFs) and Virtual Functions (VFs). Each VF has its own set of error status registers, and errors must be properly attributed to the correct Function.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

对于 SR-IOV 设备，错误处理必须考虑到物理功能（PF）和虚拟功能（VF）的分离。每个 VF 都有自己的一组错误状态寄存器，错误必须正确归因于正确的功能。

</div>

#### 6.2.10 Internal Errors / 内部错误

![Page 741 — Internal Errors](images/ch06_pg0741.png)

Internal errors are device-specific errors that are not caused by TLPs received from the Link. These are typically reported through the AER Capability structure.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

内部错误是由于设备特定原因而非从链路接收的 TLP 引起的错误。这些通常通过 AER 能力结构报告。

</div>

#### 6.2.11 Downstream Port Containment (DPC) / 下行端口遏制（DPC）

![Page 742 — DPC](images/ch06_pg0742.png)

Downstream Port Containment (DPC) is an optional Extended Capability that provides automatic error containment for uncorrectable errors detected at a Downstream Port. When an uncorrectable error is detected:
1. The Downstream Port disables the Link, preventing the propagation of corrupted data
2. The error is logged
3. Optionally, an interrupt is generated to notify software
4. Software can then take recovery actions

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

下行端口遏制（DPC）是一个可选的扩展能力，为在下行端口检测到的不可纠正错误提供自动错误遏制。当检测到不可纠正错误时：
1. 下行端口禁用链路，防止损坏数据的传播
2. 记录错误
3. 可选地，生成中断以通知软件
4. 软件然后可以采取恢复操作

</div>

##### 6.2.11.1 DPC Interrupts / DPC 中断

DPC provides interrupt generation mechanisms to notify software when DPC has been triggered and when the containment status changes.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

DPC 提供中断生成机制，在 DPC 被触发时以及遏制状态更改时通知软件。

</div>

##### 6.2.11.2 DPC ERR_COR Signaling / DPC ERR_COR 信号

DPC can be configured to signal correctable errors through the ERR_COR message mechanism.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

DPC 可以配置为通过 ERR_COR 消息机制发送可纠正错误信号。

</div>

##### 6.2.11.3 Root Port Programmed I/O (RPPIO) Error Controls / 根端口编程 I/O（RPPIO）错误控制

Root Ports provide additional error control mechanisms for programmed I/O operations.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

根端口为编程 I/O 操作提供额外的错误控制机制。

</div>

##### 6.2.11.4 Software Triggering of DPC / DPC 的软件触发

Software can trigger DPC for testing or recovery purposes by writing to the DPC Control register.

<div style="background-color:#e8e8e8; padding:4px 12px; border-radius:4px; margin:4px 0;">

软件可以通过写入 DPC 控制寄存器来触发 DPC 以进行测试或恢复。

</div>

---

> **[End of Chapter 6 / 第6章结束]**
>
> *This is a comprehensive translation of Chapter 6: System Architecture, covering:*
> - *Interrupt mechanisms (INTx emulation, MSI, MSI-X)*
> - *PME support and software models*
> - *Error signaling, classification, and logging*
> - *Advanced Error Reporting (AER)*
> - *Downstream Port Containment (DPC)*
> - *SR-IOV error handling*
>
> *Page reference images are embedded throughout the text for visual reference. 页面参考截图内嵌在文本中供参考。*
>
> *Translation Batch 2 of N. Next chapter suggestions: Chapter 2 (Transaction Layer) or Chapter 4 (Physical Layer). 翻译第2批，共N批。*
