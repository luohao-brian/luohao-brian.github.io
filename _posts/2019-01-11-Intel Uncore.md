---
layout: post
title: Intel Core & Uncore架构简介
tags: [硬件平台]
---

### 背景

uncore一词，是英特尔用来描述微处理器中，功能上为非处理器核心（Core）所负担，但是对处理器性能的发挥和维持有必不可少的作用的组成部分。处理器核心（Core）包含的处理器组件都涉及处理器指令的运行，包括算术逻辑单元（ALU）、浮点运算单元（FPU）、一级缓存（L1 Cache）、二级缓存（L2 Cache）。Uncore的功能包括QPI控制器、三级缓存（L3 Cache）、内存一致性监测（snoop agent pipeline）、内存控制器，以及Thunderbolt控制器。至于其余的总线控制器，像是PCI-E、SPI等，则是属于芯片组的一部分。

英特尔Uncore设计根源，来自于北桥芯片。Uncroe的设计是，将对于处理器核心有关键作用的功能重新组合编排，从物理上使它们更靠近核心（集成至处理器芯片上，而它们有部分原来是位于北桥上），以降低它们的访问延时。而北桥上余下的和处理器核心并无太大关键作用的功能，像是PCI-E控制器或者是电源控制单元（PCU），并没有集成至Uncore部分，而是继续作为芯片组的一部分。

具体而言，微架构中的uncore是被细分为数个模块单元的。uncore连接至处理器核心是通过一个叫Cache Box（CBox）的界面实现的，CBox也是末级缓存（Last Level Cache，LLC）的连接界面，同时负责管理缓存一致性。复合的内部与外部QPI链接由物理层单元（Physical Layer units）管理，称为PBox。PBox、CBox以及一个或更多的内置存储器控制器（iMC，作MBox）的连接由系统配置控制器（System Config Controller，作UBox）和路由器（Router，作RBox）负责管理。

从uncore部分移出串列总线控制器，可以更好地促进性能的提升，通过允许uncore的时钟频率（UCLK）运作于基准的2.66GHz，提升至超过超频限制值的3.44GHz，实现性能提升。这种时脉提升使得核心访问关键功能部件（像是存储器控制器）时的延时值更低（典型情况下处理器核心访问DRAM的时间可降低10纳秒或更多）。

### 架构

下面是较早的Intel® Xeon® Processor E7-8800/4800/2800系列CPU Uncore架构图以及各组件说明：

![Intel® Xeon® Processor E7-8800/4800/2800 Product Families Block Diagram](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/core-uncore.png)


Name | Function
---|---
Core|Intel Xeon Processor E7-8800/4800/2800 Product Families core architecture processing unit
Bbox|Home Agent
Cbox|Last Level Cache Coherency Engine
Mbox|Integrated Memory Controller
Pbox|Physical Layer (PHY) for the Intel® QPI Interconnect and Intel® SMI Interconnect memory controller
Rbox|Crossbar Router
Sbox|Caching Agent
Ubox|System Configuration Agent
Wbox|Power Controller
Intel® SMI|Intel® Scalable Memory Interconnect (formerly “FBD2” or “Fully Buffered DIMM 2 interface”)
Intel® QPI|Intel® QuickPath Interconnect
LLC|Last Level Cache (Level 3)

这是较新的Intel® Xeon® E5系列Uncore架构图，看起来HA(Home Agent)取代了Bbox, iMC取代了Mbox, PCU取代了WBox。

![Intel® Xeon® E5 series block diagram](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/core-uncore-pcm.png)

### Linux Uncore Driver

Linux会把识别出的Uncore设备列在sysfs中：

```
# ls -d /sys/devices/uncore_*
/sys/devices/uncore_cha_0   /sys/devices/uncore_imc_1
/sys/devices/uncore_cha_1   /sys/devices/uncore_imc_2
/sys/devices/uncore_cha_10  /sys/devices/uncore_imc_3
/sys/devices/uncore_cha_11  /sys/devices/uncore_imc_4
/sys/devices/uncore_cha_2   /sys/devices/uncore_imc_5
/sys/devices/uncore_cha_3   /sys/devices/uncore_irp_0
/sys/devices/uncore_cha_4   /sys/devices/uncore_irp_1
/sys/devices/uncore_cha_5   /sys/devices/uncore_irp_2
/sys/devices/uncore_cha_6   /sys/devices/uncore_irp_3
/sys/devices/uncore_cha_7   /sys/devices/uncore_irp_4
/sys/devices/uncore_cha_8   /sys/devices/uncore_irp_5
/sys/devices/uncore_cha_9   /sys/devices/uncore_m2m_0
/sys/devices/uncore_iio_0   /sys/devices/uncore_m2m_1
/sys/devices/uncore_iio_1   /sys/devices/uncore_m3upi_0
/sys/devices/uncore_iio_2   /sys/devices/uncore_m3upi_1
/sys/devices/uncore_iio_3   /sys/devices/uncore_pcu
/sys/devices/uncore_iio_4   /sys/devices/uncore_ubox
/sys/devices/uncore_iio_5   /sys/devices/uncore_upi_0
/sys/devices/uncore_imc_0   /sys/devices/uncore_upi_1

```
### PMU Tools

开源项目[pmu-tools](https://github.com/andikleen/pmu-tools)集成了ucevent(uncore event)的工具，可以查看并跟踪uncore事件。

首先需要克隆项目：

```
# Clone pmu-tools项目
git clone https://github.com/andikleen/pmu-tools
```

使用ucevnent.py列出支持的uncore events：
```
cd pmu-tools/ucevent
./ucevent.py

CHA (Home Agent) CACHE Events
  CHA.LLC_DRD_MISS_PCT           LLC DRd Miss Percentage

CHA (Home Agent) HA (Home Agent) REQUEST Events
  CHA.PCT_RD_REQUESTS            Percent Read Requests
  CHA.PCT_WR_REQUESTS            Percent Write Requests

CHA (Home Agent) INGRESS Events
  CHA.AVG_INGRESS_DEPTH          Average Ingress (from CMS) Depth
  CHA.AVG_INGRESS_LATENCY        Average Ingress (from CMS) Latency
  CHA.AVG_INGRESS_LATENCY_WHEN_NE Average Latency in Non-Empty Ingress (from CMS)
  CHA.CYC_INGRESS_BLOCKED        Cycles Ingress (from CMS) Blocked
  CHA.INGRESS_REJ_V_INS          Ingress (from CMS) Rejects vs. Inserts

CHA (Home Agent) TOR (Table of Requests, pending transactions) Events
  CHA.AVG_CRD_MISS_LATENCY       Average Code Read Latency
  CHA.AVG_DEMAND_RD_HIT_LATENCY  Average Data Read Hit Latency
  CHA.AVG_DEMAND_RD_MISS_LOCAL_LATENCY Average Data Read Local Miss Latency
  CHA.AVG_DRD_MISS_LATENCY       Average Data Read Miss Latency
  CHA.AVG_IA_CRD_LLC_HIT_LATENCY Average Code Read Latency
  CHA.AVG_RFO_MISS_LATENCY       Average RFO Latency
  CHA.AVG_TOR_DRDS_MISS_WHEN_NE  Average Data Read Misses in Non-Empty TOR
  CHA.AVG_TOR_DRDS_WHEN_NE       Average Data Reads in Non-Empty TOR
  CHA.FAST_STR_LLC_HIT           Fast String operations
  CHA.FAST_STR_LLC_MISS          Fast String misses
  CHA.LLC_CRD_MISS_TO_LOC_MEM    LLC Code Read Misses to Local Memory
  CHA.LLC_CRD_MISS_TO_REM_MEM    LLC Code Read Misses to Remote Memory
  CHA.LLC_DRD_MISS_TO_LOC_MEM    LLC Data Read Misses to Local Memory
  CHA.LLC_DRD_MISS_TO_REM_MEM    LLC Data Read Misses to Remote Memory
  CHA.LLC_DRD_PREFETCH_HITS      DRd Prefetches that Hit the LLC
  ...
```

比如跟踪10秒内不同Socket上的PCI带宽：

```
# ucevent.py -I 2000  CBO.PCIE_DATA_BYTES sleep 10
S0-CBO.PCIE_DATA_BYTES
|      S1-CBO.PCIE_DATA_BYTES
384.00 256.00
0.00   256.00
0.00   0.00
0.00   0.00
```

再比如跟踪某个应用在内存控制器(iMC)上统计获得的缺页次数，一般情况下，这个指标可以用来衡量内存延时:

```
# ucevent.py iMC.PCT_REQUESTS_PAGE_MISS my-workload
```

### PCM (Processor Counter Monitor)

[PCM](https://github.com/opcm/pcm)提供了一些监控LLC, PCI，QPI的工具:

其中，pcm.x可以监控core, cache miss/hit和各个socket的upi带宽：

```
./pcm.x
...

Detected Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz "Intel(r) microarchitecture codename Skylake-SP" stepping 4 microcode level 0x2000043

 EXEC  : instructions per nominal CPU cycle
 IPC   : instructions per CPU cycle
 FREQ  : relation to nominal CPU frequency='unhalted clock ticks'/'invariant timer ticks' (includes Intel Turbo Boost)
 AFREQ : relation to nominal CPU frequency while in active state (not in power-saving C state)='unhalted clock ticks'/'invariant timer ticks while in C0-state'  (includes Intel Turbo Boost)
 L3MISS: L3 (read) cache misses
 L2MISS: L2 (read) cache misses (including other core's L2 cache *hits*)
 L3HIT : L3 (read) cache hit ratio (0.00-1.00)
 L2HIT : L2 cache hit ratio (0.00-1.00)
 L3MPI : number of L3 (read) cache misses per instruction
 L2MPI : number of L2 (read) cache misses per instruction
 READ  : bytes read from main memory controller (in GBytes)
 WRITE : bytes written to main memory controller (in GBytes)
 L3OCC : L3 occupancy (in KBytes)
 TEMP  : Temperature reading in 1 degree Celsius relative to the TjMax temperature (thermal headroom): 0 corresponds to the max temperature
 energy: Energy in Joules


 Core (SKT) | EXEC | IPC  | FREQ  | AFREQ | L3MISS | L2MISS | L3HIT | L2HIT | L3MPI | L2MPI |   L3OCC | TEMP

   0    0     0.01   0.37   0.04    1.17      36 K    103 K    0.54    0.63    0.00    0.00      576     47
   1    1     0.00   0.88   0.00    1.17    4602       13 K    0.64    0.68    0.00    0.00      432     56
   2    0     0.01   0.82   0.01    1.17      20 K     61 K    0.57    0.70    0.00    0.00      384     48
   3    1     0.01   0.87   0.01    1.17      16 K     41 K    0.56    0.72    0.00    0.00     2016     57
   4    0     0.01   0.78   0.01    1.17      18 K     66 K    0.67    0.66    0.00    0.00      672     49
   5    1     0.00   0.51   0.01    1.17      14 K     31 K    0.51    0.62    0.00    0.00     1152     58
   6    0     0.01   0.65   0.01    1.17      21 K     72 K    0.65    0.61    0.00    0.00     1968     50
   7    1     0.00   0.82   0.00    1.17    6567       13 K    0.45    0.60    0.00    0.00      480     58
...

Intel(r) UPI data traffic estimation in bytes (data traffic coming to CPU/socket through UPI links):

               UPI0     UPI1    |  UPI0   UPI1
---------------------------------------------------------------------------------------------------------------
 SKT    0       43 M     43 M   |    0%     0%
 SKT    1       32 M     31 M   |    0%     0%
---------------------------------------------------------------------------------------------------------------
...
```

pcm.memory.x可以用来监控各个内存通道的读写带宽：

```
# ./pcm-memory.x

 Processor Counter Monitor: Memory Bandwidth Monitoring Utility  ($Format:%ci ID=%h$)

 This utility measures memory bandwidth per channel or per DIMM rank in real-time

Number of physical cores: 24
Number of logical cores: 48
Number of online logical cores: 48
Threads (logical cores) per physical core: 2
Num sockets: 2
Physical cores per socket: 12
Core PMU (perfmon) version: 4
Number of core PMU generic (programmable) counters: 4
Width of generic (programmable) counters: 48 bits
Number of core PMU fixed counters: 3
Width of fixed counters: 48 bits
Nominal core frequency: 2300000000 Hz
Package thermal spec power: 105 Watt; Package minimum power: 64 Watt; Package maximum power: 231 Watt;
Socket 0: 2 memory controllers detected with total number of 6 channels. 2 QPI ports detected.
Socket 1: 2 memory controllers detected with total number of 6 channels. 2 QPI ports detected.

Detected Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz "Intel(r) microarchitecture codename Skylake-SP" stepping 4 microcode level 0x2000043
Update every 1 seconds
|---------------------------------------||---------------------------------------|
|--             Socket  0             --||--             Socket  1             --|
|---------------------------------------||---------------------------------------|
|--     Memory Channel Monitoring     --||--     Memory Channel Monitoring     --|
|---------------------------------------||---------------------------------------|
|-- Mem Ch  0: Reads (MB/s):    18.91 --||-- Mem Ch  0: Reads (MB/s):    16.60 --|
|--            Writes(MB/s):    23.22 --||--            Writes(MB/s):    18.96 --|
|-- Mem Ch  1: Reads (MB/s):    18.98 --||-- Mem Ch  1: Reads (MB/s):    16.80 --|
|--            Writes(MB/s):    23.39 --||--            Writes(MB/s):    19.15 --|
|-- Mem Ch  2: Reads (MB/s):    18.77 --||-- Mem Ch  2: Reads (MB/s):    16.35 --|
|--            Writes(MB/s):    23.05 --||--            Writes(MB/s):    18.71 --|
|-- Mem Ch  3: Reads (MB/s):    17.89 --||-- Mem Ch  3: Reads (MB/s):    15.75 --|
|--            Writes(MB/s):    22.94 --||--            Writes(MB/s):    18.44 --|
|-- Mem Ch  4: Reads (MB/s):    17.87 --||-- Mem Ch  4: Reads (MB/s):    15.88 --|
|--            Writes(MB/s):    22.78 --||--            Writes(MB/s):    18.55 --|
|-- Mem Ch  5: Reads (MB/s):    18.04 --||-- Mem Ch  5: Reads (MB/s):    16.80 --|
|--            Writes(MB/s):    22.93 --||--            Writes(MB/s):    19.64 --|
|-- NODE 0 Mem Read (MB/s) :   110.47 --||-- NODE 1 Mem Read (MB/s) :    98.18 --|
|-- NODE 0 Mem Write(MB/s) :   138.31 --||-- NODE 1 Mem Write(MB/s) :   113.46 --|
|-- NODE 0 P. Write (T/s):      18728 --||-- NODE 1 P. Write (T/s):      18732 --|
|-- NODE 0 Memory (MB/s):      248.78 --||-- NODE 1 Memory (MB/s):      211.63 --|
|---------------------------------------||---------------------------------------|
|---------------------------------------||---------------------------------------|
|--                 System Read Throughput(MB/s):        208.65                --|
|--                System Write Throughput(MB/s):        251.77                --|
|--               System Memory Throughput(MB/s):        460.42                --|
|---------------------------------------||---------------------------------------|
```
numa带宽监控

```
# ./pcm-numa.x

...
Detected Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz "Intel(r) microarchitecture codename Skylake-SP" stepping 4 microcode level 0x2000043
Update every 1.0 seconds
Time elapsed: 1000 ms
Core | IPC  | Instructions | Cycles  |  Local DRAM accesses | Remote DRAM Accesses
   0   0.37         30 M       81 M        41 K                90 K
   1   0.71         10 M       14 M        90 K                20 K
   2   0.64         14 M       22 M        20 K                59 K
   3   1.10         79 M       71 M      7264                  21 K
   4   0.82         26 M       31 M        33 K                48 K
   5   0.95         12 M       13 M      8131                  27 K
   6   0.37         13 M       35 M       306 K               111 K
   7   1.00       7417 K     7432 K      9991                9136
   8   0.76         12 M       15 M        13 K              6833
   9   1.03         14 M       13 M        10 K                19 K
  10   0.56       6269 K       11 M        42 K                50 K
  11   0.92         26 M       29 M        60 K                85 K
  12   0.87         14 M       16 M        17 K                10 K
  13   0.64       4130 K     6504 K      5615                  40 K
  14   0.88         32 M       36 M        57 K                51 K
  15   0.88         10 M       11 M      8689                  16 K
...

```

### 参考

 * [Intel® Performance Counter Monitor - A better way to measure CPU utilization](https://software.intel.com/en-us/articles/intel-performance-counter-monitor)
 * [Intel Uncore Wiki](https://zh.wikipedia.org/wiki/Uncore)
 * [Intel® Xeon® Processor E7-8800/4800/2800 Product Families Datasheet Volume 2 of 2](https://manualsbrain.com/en/manuals/312844/pdf/7d577a522369147a0d54421c1f92ec9d6de9c175e86b6a35ba26ee2dd0476857/cisco-intel-xeon-e7-4860-ucs-cpu-e74860-user-manual.pdf)
 * [Processor Counter Monitor (PCM)](https://github.com/opcm/pcm)
 * [PMU Tools](https://github.com/andikleen/pmu-tools)
 * [Intel Processor Events Reference](https://software.intel.com/en-us/vtune-amplifier-help-intel-processor-events-reference)
 * [Linux Perf PMU Events List](https://elixir.bootlin.com/linux/latest/source/tools/perf/pmu-events/arch/x86)
