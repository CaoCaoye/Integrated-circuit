# 一、timing report阅读指南

```
【时序报告】示例
Startpoint: I_RISC_CORE/I_INSTRN_LAT/Instrn_1_reg_27_
　　　　　　 (rising edge-triggered flip-flop clocked by SYS_2x_CLK)
Endpoint: I_RISC_CORE/I_ALU/Zro_Flag_reg
　　　　　　 (rising edge-triggered flip-flop clocked by SYS_2x_CLK)
Path Group: SYS_2x_CLK
Path Type: max

Point                                                 Incr                Path
----------------------------------------------------------------------------------
clock SYS_2x_CLK (rise edge)                          0.00                0.00
clock network delay (propagated)                      0.51                0.51
I_RISC_CORE/I_INSTRN_LAT/Instrn_1_reg_27_/CP (senrq1) 0.00                0.51 r
I_RISC_CORE/I_INSTRN_LAT/Instrn_1_reg_27_/Q (senrq1)  0.62                1.13 f
I_RISC_CORE/I_INSTRN_LAT/Instrn_1[27] (INSTRN_LAT)    0.00                1.13 f
I_RISC_CORE/I_ALU/ALU_OP[3] (ALU)                     0.00                1.13 f
I_RISC_CORE/I_ALU/U288/ZN (nr03d0)                    0.36 *              1.49 r
I_RISC_CORE/I_ALU/U261/ZN (nd03d0)                    0.94 *              2.43 f
I_RISC_CORE/I_ALU/U307/ZN (invbd2)                    0.35 *              2.78 r
I_RISC_CORE/I_ALU/U343/Z (an02d1)                     0.16 *              2.93 r
I_RISC_CORE/I_ALU/U344/ZN (nr02d0)                    0.11 *              3.04 f
I_RISC_CORE/I_ALU/U348/ZN (nd03d0)                    0.28 *              3.32 r
I_RISC_CORE/I_ALU/U355/ZN (nr03d0)                    0.29 *              3.60 f
I_RISC_CORE/I_ALU/U38/Z (an02d1)                      0.15 *              3.75 f
I_RISC_CORE/I_ALU/U40/Z (an02d1)                      0.12 *              3.87 f
I_RISC_CORE/I_ALU/U48/ZN (nd02d1)                     0.06 *              3.93 r
I_RISC_CORE/I_ALU/U27/ZN (nd02d1)                     0.06 *              3.99 f
I_RISC_CORE/I_ALU/Zro_Flag_reg/D (secrq4)             0.00 *              3.99 f
data arrival time                                                         3.99


clock SYS_2x_CLK (rise edge)                          4.00                4.00
clock network delay (propagated)                      0.47                4.47
clock uncertainty                                    -0.10                4.37
I_RISC_CORE/I_ALU/Zro_Flag_reg/CP (secrq4)            0.00                4.37 r
library setup time                                   -0.37                4.00
data required time                                                        4.00
--------------------------------------------------------------------------------
data required time                                                        4.00
data arrival time 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　-3.99
-------------------------------------------------------------------------------
slack (MET) 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　0.01
```

**timing report可分为4个部分：路径信息、路径延时信息、路径延时要求、以及时间总结**

## 1.1 路径信息

<div align="center"><img src=".\pictures/STA_pct/image.png" alt="Alt text"><p>图 1-1</p></div> 

如上图所示，第一部分的表头主要包含了：工作条件、使用的工艺库、时序路径的起点和终点、路径所属的时钟组、报告的信息是作建立或保持的检查，以及所用的线负载类型。

> **建立时间的检查**可以确保**移动最慢的数据**也能及时到达并满足建立的标准，因此，它又被称为**最大分析(max analysis)**。由于建立检查考虑了最晚到达的数据，所以也被称为**晚期分析(late analysis)**

> **保持时间的检查**可以确保即使是**移动最快的数据**也不应该干扰其他数据，同时期望数据保持稳定。因此，数据路径计算最小延迟，被称为**最小分析（min analysis）**。由于保持检查考虑了最早到达的数据，所以也称其为**早期分析（early analysis）**

---

## 1.2 路径延迟信息

这个路径延迟部分是DC计算得到的实际延迟信息；命令执行后，对于下图中的路径，得到的一些路径信息，有了单元名称（point），通过该单元的延时（Incr）,经过这个单元后路径的总延时（Path）等信息：

> （一般是6列：point分析点、fanout扇出值、trans传输延时、incr器件延时、path路径总延时、attributes延时类型）

> 星号(*)表示了使用了SDF文件中的延时值，r表示上升延时，f表示下降延时。

> **标准延迟文件SDF**：主要包含了网表中所有器件的延迟信息，用于时序仿真；通常情况下，在仿真过程中会使用由PT报出的sdf，因为PT会结合后端工具，生成延迟更为精确的sdf文件。

<div align="center"><img src=".\pictures/STA_pct/image-1.png" alt="Alt text"><p>图 1-2</p></div> 

图释：
1. 路径的起点是上一级D触发器的的时钟端。
   
2. input external delay:(由于上一级D触发器的翻转(路径的起点也就这里)、芯片外部组合逻辑而经历的)输入延时约束（set_input_delay），也就是数据到达芯片的数据输入管脚的延时建模,这个延时是1ns;”r”表示上升延时，”f”表示下降延时

3. clock network delay(idle):时钟信号从芯片的端口到内部第一个寄存器的延时是0.5ns；

4. Data1(in):芯片输入端口到芯片内部真正数据输入端之间的线延时，是0.04ns。（可以认为是管脚的延时）

5. U2/y : 这里，前面0.12表示u2这个器件的翻转/传输延时，意思是从这个器件的数据输入端（包括连线），到输出端y的延时是0.12ns。后面的1.66的意思是从路径起点到u2的y输出的延时是1.66ns.

···

6. 最后u4/D：这里就是终点了，D触发器的数据输入端；当然终点也可能是芯片的输出端口。

> PS:
> 报告中，小数点后默认的位数是二，如果要增加有效数(字)，在用report_timing命令时，加上命令选项“-significant_digits"。报告中，Inc:是连线延迟和其后面的单元延迟相加的结果。如要分别报告连线延迟和单元延迟，在使用report_timing命令时，加上命令选项"-input_pins"。

---

## 1.3 路径延时要求

<div align="center"><img src=".\pictures/STA_pct/image-2.png" alt="Alt text"><p>图 1-3</p></div> 

这个路径要求部分是我们约束所要求的部分；值-0. 06从库中查出，其绝对值是寄存器的建立时间。值2.17为时间周期加上延时减去时钟偏斜值再减寄存器的建立时间(假设本例中的时钟周期是2 ns)。

## 1.4 时间总结

<div align="center"><img src=".\pictures/STA_pct/image-3.png" alt="Alt text"><p>图 1-4</p></div> 

DC得到实际数据到达的时间和我们要求的时间后，进行比较。数据要求2.17ns前到达（也就是数据延时要求不得大于2.17ns），DC经过计算得到实际到达时间是2.15ns，因此时序满足要求，也就是met，而不是时序违规（violation）。时间冗余(Timing margin)，又称slack，如果为正数或‘0’，表示满足时间目标。如果为负数，表示没有满足时间目标。这时，设计违反了约束(constraint violation)。

---

# 二、时序违例解决方法
## 2.1 setup time违例解决方法
1. 加强约束，重新进行综合，对违规的路径进行进一步的优化（但一般效果可能不是很明显）
2. 降低时钟频率，但时钟频率一般是在项目最初的时候决定的，后期很难再改变
3. 拆分组合逻辑，插入寄存器，增加流水线，这个是常用方法
4. 优化布局布线，减小传输的延时

## 2.2 hold time违例解决方法
1. 增加组合逻辑的处理时间，一般就是在后端的时候插入buffer
2. 减少时钟的延时

