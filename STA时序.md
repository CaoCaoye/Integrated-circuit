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

# 三、multicycle path约束注意点：
## 3.1 错误示例1
> 该结构reg1 **每一拍实时采样。但在reg1施加约束multicycle2**，导致reg1.Q会有亚稳态可能。虽然对于后级采样而言，源头是一个，但源头作为亚稳态如果fanout到多个DFF（如冗余比较逻辑），且在多个DFF之间会有耦合关联，有可能main/shadow会采集到不一致数据，功能层面就存在错误风险。
- **处理方法1**：功能是否正常，依赖于reg1的亚稳态多久可以恢复稳态，可以report reg1-->main/shadow reg 的timing slack，确认时序余量，将亚稳态稳定时间与其进行对比。如果slack时间可大于亚稳态稳定时间，则风险相对可控；当前DFF仿真看亚稳态恢复时间估计在500 ps量级；
- **处理方法2**：reg1后面插入1级dff，来减少亚稳态传播概率。因为通常经过两级同步器，亚稳态传播概率极低，可忽略。

<div align="center"><img src=".\pictures/STA_pct/image3-1.png" alt="Alt text"><p>图 3-1</p></div> 

## 3.2 错误示例2
> MCP后级直连多个DFF，各个DFF结果不一致, 类似于冗余设计结构，非冗余设计做后续逻辑汇聚后同样存在逻辑错误风险
- **处理方法**：这类问题，冗余比较大概率会存在异常，需要电路修改保证

<div align="center"><img src=".\pictures/STA_pct/image3-2.png" alt="Alt text"><p>图 3-2</p></div> 

## 3.3 错误示例3
> 该结构reg1 每一拍实时采样，导致会采集到非实际需求数据且为亚稳态，在后级fanout到单一DFF时，如图reg2处使用亚稳态信号取边沿，或者亚稳态信号直接参与逻辑运算（未示意），也认为是有风险需要处理
- **处理方法**：数据从reg0拍出后，在目标时钟域中进行打拍两拍处理（源时钟和目标时钟为同步时钟），避免亚稳态问题

<div align="center"><img src=".\pictures/STA_pct/image3-3.png" alt="Alt text"><p>图 3-3</p></div> 

# 四、clock_gating_check
门控时钟是RTL级进行低功耗设计的最常用方法，能够有效降低动态功耗。在实际使用中，一般用ICG（集成门控时钟单元）来完成clock gating。ICG电路和时序如下：

<div align="center"><img src=".\pictures/STA_pct/image4-1.png" alt="Alt text"><p>图 4-1</p></div> 

通常来说，工艺库已经集成了ICG，在做门控时钟的时候其实不用考虑那么多。如果在实际设计中，工艺库未提供ICG，需要自己搭建一个门控时钟电路，那么在布局布线的时候可能要注意，不能将期间摆的太远，否则线延迟等会让这个结构失去其意义。

当门控信号控制了逻辑单元中时钟信号的路径时，我们会进行Clocl Gating check。我们根据下图了解一些定义，这样比较直观：

<div align="center"><img src=".\pictures/STA_pct/image4-2.png" alt="Alt text"><p>图 4-2</p></div> 

因此，我们需要明确什么样的电路会被工具分析为门控时钟：
1. Gating cell的输出是作为时钟信号进行使用，即连接到了后级逻辑单元的时钟端口。
2. Gating pin的输入非时钟信号；若Gating pin的输入为时钟信号，则其必须不连接后级逻辑单元的时钟端口。
3. 对于上图简单的与门控制，工具可以较好的推断；但对于复杂结构，需要自己添加约束来进行检查。

<div align="center"><img src=".\pictures/STA_pct/image4-3.png" alt="Alt text"><p>图 4-3</p></div> 

运用提到的定义，对于上图来说，CLKB才是clock signal。接下来我们以这个电路为例看看如何进行clock gating check（对于这个与门来说，有些类似于data to data check）：
首先定义两个时钟：
create_clock -name CLKA -period 10 -waveform {0 5}  [get_ports CLKA]
create_clock -name CLKB -period 10 -waveform {0 5}  [get_ports CLKB]

我们要保证时钟切换的时候不能产生一些毛刺，也不能产生对时钟有影响的结果。对于此电路来说，gating cell是一个与门，那么gating pin的信号变化只能发生在clock pin的低电平状态。则setup check要求门控信号在时钟信号变高前变化；hold check要求门控信号只能在时钟下降沿之后进行改变。时序图如图，gating pin的变化要落在5-10ns间，才能满足门控时钟的要求。

<div align="center"><img src=".\pictures/STA_pct/image4-4.png" alt="Alt text"><p>图 4-4</p></div> 

setup check的时序报告：

<div align="center"><img src=".\pictures/STA_pct/image4-5.png" alt="Alt text"><p>图 4-5</p></div> 

同样地，我们可以看看hold check的时序报告：

<div align="center"><img src=".\pictures/STA_pct/image4-6.png" alt="Alt text"><p>图 4-6</p></div> 

可以看到，在5ns时进行hold check是非常严苛的。由于gating signal的变化过于迅速，导致hold check失败。这个问题可以用我们前面提到的半周期进行处理。我们将寄存器的采样边沿改为负沿：

<div align="center"><img src=".\pictures/STA_pct/image4-7.png" alt="Alt text"><p>图 4-7</p></div> 

这样的话，gating signal在负沿launch，留给时序检查的周期就剩下了半个周期，较好满足hold time：

<div align="center"><img src=".\pictures/STA_pct/image4-8.png" alt="Alt text"><p>图 4-8</p></div> 

# 补充概念
## OCV
> OCV(on-chip variation) OCV是指由于工艺、电源电压(V)和温度(T)在芯片不同区域上的差异而导致的性能变化。这些变化会影响芯片的时序特性，使得同一芯片的不同部分可能处于不同的PVT条件下。为了应对这种变化，设计者需要在静态时序分析(STA)中引入OCV分析，通过降额(derate)特定路径的延迟来模拟OCV的影响。

