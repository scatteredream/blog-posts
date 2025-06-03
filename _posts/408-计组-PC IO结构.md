---
name: pc-io-structure
title: PC IO 结构
date: 2024-09-18
tags:
- 磁盘
categories:
- 计组

---



# Prototypical Architecture

![image-20241216183418923](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241216183418923-1734346411610-1.png)

CPU通过专用的内存总线连接到内存，通过通用I/O总线(如PCI)连接到显卡等高性能设备，通过外围总线连接到低速的设备(USB Flash, 磁盘驱动器 )，因为光速的限制以及各种物理因素，越快的总线越短，造价也贵，因此离CPU越远的设备性能越低。

# Modern Architecture

![image-20241216185046235](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241216185046235.png)

![undefined](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1024px-Motherboard_diagram.svg-1734348489593-5.png)

在现代系统架构中，有更多的点对点连接到 CPU，PCIe 显卡总线和内存专用总线速度相近，以提供更高的显示性能，CPU通过 **DMI** 连接到一颗 I/O 芯片，所有其他外设通过各种总线连接到这颗I/O芯片。

## CPU Socket

- 用于安装中央处理器 (CPU)。
- 支持不同接口类型（如 LGA、PGA 或 BGA）。
- 插槽周围有供电模块（VRM）提供稳定电压。

不同CPU系列使用不同插槽。后期CPU插槽，数字多数与针脚数量相同：

[LGA 1155](https://zh.wikipedia.org/wiki/LGA_1155) [LGA 1366](https://zh.wikipedia.org/wiki/LGA_1366) [LGA 2066](https://zh.wikipedia.org/wiki/LGA_2066)  [LGA 1700](https://zh.wikipedia.org/wiki/LGA_1700) [Socket AM4](https://zh.wikipedia.org/wiki/Socket_AM4) [Socket TR4](https://zh.wikipedia.org/wiki/Socket_TR4) 

## Chipset

管理数据传输和硬件通信，负责连接 CPU、内存、显卡、存储等设备

**芯片组**(Chipset)负责将电脑的处理器和和其它部分连接，以便能互传数据。芯片组在它所诞生的1980年代时是由多颗微芯片组成的，但是随着科技的进步，芯片组先是从2000年代开始简化为南桥和北桥两颗芯片，再于2010年代简化为单独一颗的南桥芯片，北桥芯片内置在CPU中，目前世界上的芯片组均以单南桥芯片为主流。  

- **南桥**(Southbridge): 主要处理低速信号，后来演变成 ICH->PCH
- **北桥**(Northbridge): 主要处理高速信号，同时也负责与南桥的通信，IOH->MCH，集成进CPU
- **DMI**：连接南北桥的串行总线
- **FSB**：前端总线，负责 CPU 和北桥的数据传递。现在的x86 CPU 内部整合了[北桥](https://zh.wikipedia.org/w/index.php?title=記憶體控制器&action=edit&redlink=1)。FSB 已被Intel [QPI](https://zh.wikipedia.org/wiki/QPI)和AMD [HyperTransport](https://zh.wikipedia.org/wiki/HyperTransport)取代

### Memory Controller Hub (Northbridge)

随着技术发展，原来的北桥芯片已直接整合到 **CPU 内部**，减少延迟并提升效率：

| 控制器名称                         | 功能描述                                                    |
| ---------------------------------- | ----------------------------------------------------------- |
| **Memory Controller**              | 控制内存访问和管理，负责与 RAM 通信。                       |
| **Integrated Graphics Controller** | 提供基本图形渲染能力，支持视频输出，无需独立显卡。          |
| **PCIe Controller (部分 CPU)**     | 高端 CPU 内部集成部分 PCIe 通道，用于高性能设备的直接通信。 |
| **Power Management Controller**    | 管理电源状态和功耗调节，提高能效。                          |

#### Memory Slots

- 用于安装内存模块（RAM）。
- 常见类型：DDR3、DDR4、DDR5 等。
- 多通道技术（如双通道、四通道）提高内存带宽。

##### Volatile Memory (RAM)

[易失性存储器](https://zh.wikipedia.org/wiki/%E6%8F%AE%E7%99%BC%E6%80%A7%E8%A8%98%E6%86%B6%E9%AB%94)(Volatile Memory): 电流中断后，所存储的数据便会消失，一般为 **RAM**

1. [静态随机存储器](https://zh.wikipedia.org/wiki/靜態隨機存取記憶體)(**SRAM**, Static RAM): 把信息存储在锁存器中，只要保持通电，存储的数据就可以一直保持，存储密度较低，速度高，功耗低，内部结构也更为复杂。通常作为**cache**(L1, L2, L3)、**寄存器**或 FPGA ASIC 等专用设备的存储器。
2. [动态随机存储器](https://zh.wikipedia.org/wiki/动态随机存取存储器)(**DRAM**, Dynamic RAM): 根据电容中的电荷多寡来分辨0和1，电容会漏电，不仅要保持通电，还需周期性充电。由于这种需要定时刷新的特性，因此被称为“动态”存储器。存储密度高，速度较低，功耗高，但是成本比 SRAM 低，通常作为 主存 使用，用于存储运行中的程序和数据；也用于 显存。
   - [同步 DRAM](https://zh.wikipedia.org/wiki/SDRAM)(**SDRAM**, Synchronous DRAM): 在 DRAM 的架构基础上增加同步和双区域（Dual Bank）的功能，使得[微处理器](https://zh.wikipedia.org/wiki/微處理器)能与 SDRAM 的[时钟](https://zh.wikipedia.org/wiki/時脈)同步，所以 SDRAM 执行命令和传输资料时相较于 DRAM 可以节省更多时间。
   - [双倍数据率 SDRAM](https://zh.wikipedia.org/wiki/DDR_SDRAM)(DDR SDRAM, Double Data Rate): DDR SDRAM 在系统时钟的上升沿和下降沿都可以进行数据传输，发展到 DDR5，数据传输率、总线频率逐渐提高
     - [低功耗 DDR](https://zh.wikipedia.org/wiki/%E7%A7%BB%E5%8A%A8DDR)(**LPDDR**, Low Power DDR): 专门用于移动设备，发展到 LPDDR5X
     - 图形 DDR(**GDDR**, Graphics DDR): 为高性能显卡提供显存支持，发展到 GDDR7
   - [高带宽内存](https://zh.wikipedia.org/wiki/高頻寬記憶體)(HBM, High Bandwidth Memory): 基于3D堆栈工艺的高性能 [DRAM](https://zh.wikipedia.org/wiki/DRAM) 适用于高存储器带宽需求的应用场合，与高性能**显卡**、路由器、交换机、高性能数据中心的AI ASIC结合使用，在 CPU 和 FPGA 中用作包内 RAM 和 Cache

#### PCIe

北桥负责高速传输，因此显卡这种高速设备也需要通过PCIe和北桥(CPU)相连，详见下方PCIe x16

### I/O Controller Hub (Southbridge)

| 控制器名称                    | 功能描述                                                |
| ----------------------------- | ------------------------------------------------------- |
| **Disk Controller**           | 控制 SATA、SAS 等存储设备的数据传输。                   |
| **USB Controller**            | 控制 USB 2.0、3.0、3.1、Type-C 等接口的数据传输。       |
| **PCIe Controller**           | 管理 PCI Express 插槽的通信，用于扩展显卡、网卡等设备。 |
| **Ethernet Controller**       | 控制有线网络通信，通常内置支持千兆或 2.5G 网卡。        |
| **Audio Controller**          | 提供板载音频功能，支持麦克风、扬声器等外设连接。        |
| **SATA Controller**           | 控制 SATA 硬盘和光驱设备的数据传输。                    |
| **RAID Controller**           | 支持磁盘阵列（RAID）的数据管理，提高存储性能和安全性。  |
| **Wireless Controller**       | 控制 Wi-Fi 和蓝牙模块的无线通信功能（部分主板内置）。   |
| **Security Controller (TPM)** | 提供可信平台模块 (TPM) 功能，保障数据安全性和加密管理。 |

#### Storage Interface (SATA/SAS)

- **IDE**(Integrated Device Electronics): IDE 是一项企图把控制器与盘体集成在一起为主要意图的硬盘接口技术。
- **ATA**(Advanced Technology Attachment): ATA 技术是一个关于 IDE 的技术规范族，全球标准化协议将 IDE 接口技术自诞生以来使用的技术规范归纳成为全球硬盘标准。
  - **并行接口 ATA**(Parallel ATA)的电缆属性、连接器和信号协议都表现出了很大的技术瓶颈，而在技术上突破这些瓶颈存在相当大的难度
- ==**SATA**==(Serial ATA): 并行的 PATA 存在固有的瓶颈——并行信号串扰与同步问题，因此改成了**串行**总线。SATA 既是一种**物理接口**（硬件层连接方式），也是一种**逻辑接口**协议（定义数据通信规则）最初是为传统机械硬盘（HDD）设计的，后来也应用于部分 SSD
  - **eSATA**(External SATA): 外置 SATA 接口，方便外设的连接，eSATA 不像 USB 那样能同时传输数据和供电，因此需要额外的电源线为外部设备供电，属于物理接口协议。
  - **mSATA**(mini-SATA): 迷你版本SATA接口，外型和电子接口与mini PCI-E完全相同，但电子信号不同，两者互不兼容。多用于固态硬盘，适用于需要尺寸较小的存储器的场合
  - **AHCI**(Advanced Host Controller Interface): 允许软件与 [SATA](https://zh.wikipedia.org/wiki/SATA) 存储设备沟通的硬件机制，激活 SATA 的高级功能，属于逻辑接口协议。AHCI 系统的[主机适配器](https://zh.wikipedia.org/wiki/主机适配器)将CPU/存储器子系统与相对慢得多的、基于旋转[磁性介质](https://zh.wikipedia.org/wiki/磁儲存)的存储子系统，AHCI 是针对这种悬殊的速度差来进行优化设计的。AHCI 是针对硬盘优化的，不适合固态盘。
- **SCSI**(Small Computer System Interface): 一种连接主机和外围设备的接口，支持包括硬盘、光驱及扫描仪在内的多种设备。SCSI 总线是一种**并行**总线，其优点是适应面广，性能高；缺点是价格昂贵，安装复杂。
- ==**SAS**==(Serial Attached SCSI): **串行** SCSI ，跟 SATA 总线类似，都是采用串行技术以获得更高的传输速度。就接口标准而言，SATA 是 SAS 的一个子标准，因此 SAS 控制器可以直接操控SATA 硬盘，但是 SATA 控制器并不能对 SAS 硬盘进行控制。

SCSI/SAS 需要专用的控制卡，或者主板另外集成控制芯片，性能更好。PATA/SATA 一般由主板南桥/ICH 芯片直接集成控制器，但接口性能一般比同时期的 SCSI/SAS 要低。目前，服务器流行的硬盘接口类型是 SATA 以及 SAS，两者都是采用串行技术，传输速率都更高。

#### I/O Ports(USB, DP, HDMI)

**通用外设接口** 

- **==USB==**(Universal Serial Bus): 串口总线标准，也是一种输入输出接口的技术规范，被广泛地应用于个人电脑和移动设备等信息通讯产品，并扩展至摄影器材、数字电视(机顶盒)、游戏机等其它相关领域。逐渐取代 PS/2 连接键盘与鼠标，取代 COM(串口)与 LPT(并口)
  - 技术标准
    - USB 2.0: 480 Mbps，半双工
    - USB 3.0: 5 Gbps，全双工，向下兼容
    - USB 3.1: 10 Gbps，向下兼容
    - USB 3.2: 20 Gbps，在[USB Type-C](https://zh.wikipedia.org/wiki/USB_Type-C)接口上实现双通道，向下兼容，推荐 Type-C 接口
    - USB 4: 40 Gbps，只支持 Type-C
    - USB PD: USB 充电标准

  - 接口
    - Mini-USB：弃用
    - Type-A： <img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1920px-USB_3.0_Type-A_receptacle_blue.svg.png" alt="undefined" style="zoom:3%;" />  <img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/2560px-USB_3.0_Type-A_blue.svg.png" alt="undefined" style="zoom:3%;" />
    - Type-B：<img src="https://upload.wikimedia.org/wikipedia/commons/d/d5/USB_3.0_Type-B_blue.svg" style="zoom:17%;" /> <img src="https://upload.wikimedia.org/wikipedia/commons/8/8c/USB_3.0_Type-B_receptacle_blue.svg" style="zoom:17%;" />
    - Micro-USB：<img src="https://upload.wikimedia.org/wikipedia/commons/1/1b/USB_Micro-B_receptacle.svg" style="zoom:15%;" /> <img src="https://upload.wikimedia.org/wikipedia/commons/1/15/USB_Micro-B.svg" style="zoom:16%;" />
    - Type-C：<img src="https://upload.wikimedia.org/wikipedia/commons/e/e8/USB_Type-C_receptacle.svg" style="zoom:15%;" /> <img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/512px-USB_Type-C_icon.svg.png" alt="img" style="zoom:13%;" />

- **雷电**(Thunderbolt): 目的在于当作电脑与其他设备之间的通用[总线](https://zh.wikipedia.org/wiki/匯流排_(數據))，第一代与第二代的连接口与[Mini DisplayPort](https://zh.wikipedia.org/wiki/Mini_DisplayPort)集成，第三代至第五代的连接口与[USB Type-C](https://zh.wikipedia.org/wiki/USB_Type-C)结合，并且能用于充电。由于采用了PCI Express和DisplayPort架构，雷电可以**高速连接诸如硬盘、RAID阵列、视频采集设备和网络接口等外设** 

**音视频接口**(HDMI, DP, 雷电)：

- **VGA**(视频图形阵列, **V**ideo **G**raphics **A**rray): 使用[模拟信号](https://zh.wikipedia.org/wiki/類比訊號)的电脑显示标准，在1987年随IBM PS/2系列计算机推出。VGA是大多数PC制造商所遵循的最后一个IBM图形标准，几乎1990年后的所有PC图形硬件都最低支持VGA。当用VGA来表示分辨率时，通常是指640×480。

- **DVI**(**数字**视频接口, **D**igital **V**isual **I**nterface)：一种[视频](https://zh.wikipedia.org/wiki/視訊)接口标准，设计的目的是用来传输未经压缩的数字化影像。目前广泛应用于[LCD](https://zh.wikipedia.org/wiki/LCD)、数字[投影机](https://zh.wikipedia.org/wiki/投影機)等显示设备上。
- **HDMI**(高清媒体接口, **H**igh **D**efinition **M**ultimedia **I**nterface): 是一种全[数字](https://zh.wikipedia.org/wiki/數位)化[影像](https://zh.wikipedia.org/wiki/影像)和[声音](https://zh.wikipedia.org/wiki/聲音)发送接口，可以发送未[压缩](https://zh.wikipedia.org/wiki/數位壓縮)的[音频](https://zh.wikipedia.org/wiki/音频)及[视频](https://zh.wikipedia.org/wiki/視頻)信号。HDMI 可以同时发送音频和视频信号。音频和视频信号采用同一条线材的设计大大简化系统线路的安装难度。
- **DP**(显示端口, **D**isplay **P**ort): 数字式音频/视频接头，此接口的设计是为了取代传统的[VGA](https://zh.wikipedia.org/wiki/VGA)、[DVI](https://zh.wikipedia.org/wiki/DVI)接口。透过主动或被动转接器，该接口可与传统接口（如[HDMI](https://zh.wikipedia.org/wiki/HDMI)和[DVI](https://zh.wikipedia.org/wiki/DVI)）[向下兼容](https://zh.wikipedia.org/wiki/向下相容)。
- **雷电**(Thunderbolt): 有接口也有总线，目的在于当作电脑与其他设备之间的通用 **总线**，第一代与第二代的连接口与[Mini DisplayPort](https://zh.wikipedia.org/wiki/Mini_DisplayPort)集成，第三代至第五代的连接口与[USB Type-C](https://zh.wikipedia.org/wiki/USB_Type-C)结合，并且能用于充电。由于采用了PCI Express和DisplayPort架构，雷电可以**高速连接诸如硬盘、RAID阵列、视频采集设备和网络接口等外设**。

#### Peripheral Card Slots

外围扩展设备插槽：

- 用于连接显卡、声卡、网卡、固态硬盘 (NVMe SSD) 等扩展设备。
- 常见标准：PCI Express (PCIe) 3.0、4.0、5.0  M.2 2280

##### PCIe slots

PCIe 可以连到南桥或北桥

- **PCI**(Peripheral Component Interconnect): 常见于现代的个人电脑中，并已取代了 ISA 和 VESA 局部总线，成为了标准扩展总线，是并行的总线。
- **==PCIe==**(PCI Express): 沿用既有的 PCI 编程概念及信号标准，并且构建了更加高速的串行通信系统标准。只需修改**物理层**而无须修改软件就可将现有 PCI 系统转换为 PCIe，几乎取代了以往所有的内部总线(包括 PCI)。现在英特尔和 AMD 已采用单芯片组(IO Chip)技术，取代原有的南桥和北桥方案。PCIe仅应用于内部互连，没有对外开放的接口。可以连接**显卡**、**网卡**，高性能的持久化存储设备如 NVMe 设备。根据传输通道数量分为 PCIe ×1, PCIe ×4, PCIe ×8, PCIe ×16
  - **SATA Express**: 物理、逻辑接口标准，使用 PCIe 总线，向下兼容 SATA，支持 PCIe(AHCI 和 NVMe) 设备。后来物理接口被 M.2 和 U.2 取代。
  - **NVMe**(NVM Express, NVMHCIS): 非易失性存储器的接口标准，它是基于设备**逻辑接口**的总线传输协议规范，用于访问通过 PCIe 总线附加的[非易失性存储器](https://zh.wikipedia.org/wiki/非揮發性記憶體)介质（比如 SSD）降低了I/O操作等待时间、提升同一时间内的操作数、更大容量的操作队列等，取代 AHCI 作为新的逻辑接口协议。

###### NVM(Flash/EEPROM)

**[非易失性存储器](https://zh.wikipedia.org/wiki/%E9%9D%9E%E6%8F%AE%E7%99%BC%E6%80%A7%E8%A8%98%E6%86%B6%E9%AB%94)**(**NVM**, Non-Volatile Memory): 电流关掉后，所存储的信息不会消失的存储设备。依存储器内的资料是否能在使用系统时随时改写为标准，可分为三大类产品：

1. [只读存储器](https://zh.wikipedia.org/wiki/ROM)(ROM, Read-only Memory)
   - MROM(Masked ROM): 使用掩模工艺，出厂后不可改变，永久固化
   - PROM(Programmable ROM): 用户可编程的 ROM，烧断熔丝来改变比特，不可逆
   - [EPROM](https://zh.wikipedia.org/wiki/%E5%8F%AF%E6%93%A6%E9%99%A4%E5%8F%AF%E8%A6%8F%E5%8A%83%E5%BC%8F%E5%94%AF%E8%AE%80%E8%A8%98%E6%86%B6%E9%AB%94)(Erasable PROM): FGMOS 代替熔丝，专用编程器编程，用紫外线擦除，可逆
   - [EEPROM](https://zh.wikipedia.org/wiki/%E9%9B%BB%E5%AD%90%E6%8A%B9%E9%99%A4%E5%BC%8F%E5%8F%AF%E8%A4%87%E5%AF%AB%E5%94%AF%E8%AE%80%E8%A8%98%E6%86%B6%E9%AB%94)(Electrically EPROM): 只需要特定电压就可以擦除

2. [闪存](https://zh.wikipedia.org/wiki/Flash_memory)(Flash Memory): 与传统的硬盘相比，闪存有更佳的动态抗震性，不会因为剧烈晃动而造成资料丢失。与 SRAM 相比不需要供电，造价相对 EEPROM 较低，使用块抹除。闪存在分类上属于 [EEPROM](https://zh.wikipedia.org/wiki/EEPROM) 的一种，但普通的 EEPROM 使用的是字节抹除。 
   - 按照 FGMOS 的门电路结构分为：

     - NOR Flash: 抹写时间长，提供完整的寻址总线，可按字节**随机存取**，类似 RAM
     - NAND Flash: 抹写时间短，次数高，存储密度高，只能按页访问，类似硬盘，应用有 SSD 固态硬盘、SD 卡、U盘等。

   - 按照 每存储单元存储 bit 数分为：

     - SLC(1) MLC(2) TLC(3) QLC(4) 成本、寿命、写入性能依次降低

     - ![undefined](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/2560px-Nand_flash_structure.svg.png)

3. [非易失性随机存储器](https://zh.wikipedia.org/w/index.php?title=非揮發性隨機存取記憶體&action=edit&redlink=1)(NVRAM, Non-volatile RAM)
   - [FeRAM](https://zh.wikipedia.org/wiki/FRAM): 替换介电质为铁电材料
   - [MRAM](https://zh.wikipedia.org/wiki/磁阻式隨機存取記憶體): 使用磁存储器件存储数据，应用巨磁阻效应

##### M.2 slots

- **==M.2==**(NGFF): 采用全新物理布局与接口，取代 [PCIe](https://zh.wikipedia.org/wiki/PCI_Express) 及 [mSATA](https://zh.wikipedia.org/wiki/SATA#mSATA) 的物理插槽
  - U.2: 2015 年SATA Express发布了一种兼容SAS、PCI Express x4、SATA总线的的U.2连接器界面，U.2更多用于[服务器](https://zh.wikipedia.org/wiki/伺服器)等企业应用场合。U.2支持[热插拔](https://zh.wikipedia.org/wiki/熱插拔)而M.2不支持；U.2可使用3.3V电源和12V电源，M.2只能使用3.3V电源；M.2可应用于[固态硬盘](https://zh.wikipedia.org/wiki/固态硬盘)、[无线网卡](https://zh.wikipedia.org/wiki/无线网卡)等设备，而U.2仅用于2.5英寸固态盘。
  - 支持 SATA、AHCI+PCIe、NVMe+PCIe，同时可以连接网卡等

#### Flash ROM (BIOS)

大部分的主板的BIOS存储在Flash ROM芯片内，用于对主板作启动的初始化；在启动的过程中包含存储器、周边设备都会被测试以及做初始设置，这个过程称为 加电自检 POST，若是在 POST 的过程中出现错误，则主机会发出"哔"声或是出现错误消息在屏幕上。从2011年起，大部分的零售主板已采用UEFI BIOS，一些厂商（尤其是微星科技、华硕）还率先导入图形界面的UEFI BIOS等技术。

#### CMOS Memory

CMOS 纽扣电池供电给 CMOS 芯片(RAM)，CMOS 芯片保存系统时间和 BIOS 设置。

## Other Modules

### Power Interface

- 24 针主电源接口：为主板供电。
- 8 针或 4+4 针 CPU 辅助电源接口：为 CPU 供电。

### Cooling Module

- CPU 散热器接口：供电及控制风扇速度。
- 芯片组和供电模块上的散热片和风扇，降低温度。

![undefined](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1024px-IO_stack_of_the_Linux_kernel.svg.png)
