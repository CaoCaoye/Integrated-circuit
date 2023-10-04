# 1.亚稳态
* 产生原因：
    * 跨时钟域数据不满足目标时钟的建立保持时间
    ![Alt text](pictures/CDC_pct/image.png)
* 解决方案：
    * 两级或多级同步器隔离亚稳态
>* 亚稳态无法消除
>* 亚稳态只能被隔离，降低发生概率
>* 两级同步器之间不要有组合逻辑
![Alt text](pictures/CDC_pct/image-1.png)

# 2.数据保持（快时钟域到慢时钟域）
![Alt text](pictures/CDC_pct/image-2.png)
* 产生原因：
    * 数据从快时钟域到慢时钟域
    * 数据**保持的时间较短**，无法被慢时钟域采样
* 解决方案：
    * 数据展宽 (单bit)
        * ![Alt text](pictures/CDC_pct/image-3.png)

    * 脉冲同步器 (单bit)
        * ![Alt text](pictures/CDC_pct/image-4.png)
        * ![Alt text](pictures/CDC_pct/image-5.png)
        * 1. 将clock1时钟域的输入脉冲转换为clock1时钟域的电平信号；
        * 2. 对电平信号进行打拍（打两拍）同步到clock2时钟域；
        * 3. 对clock2时钟域的电平信号进行检测，产生clock2时钟域的脉冲信号；


# 3.数据冗余（慢时钟域到快时钟域）
![Alt text](pictures/CDC_pct/image-6.png)
* 产生原因：
    * 数据从慢时钟域到快时钟域
    * 数据在目标时钟域被**多次采样**
* 解决方案：
    * 边沿检测 (单bit)
        * ![Alt text](pictures/CDC_pct/image-7.png)
        * ![Alt text](pictures/CDC_pct/image-8.png)
        * 两个同步的触发器，将其他时钟域信号同步到clock A的时钟域中
        * 一个触发器加一个逻辑门， 逻辑门的两个输入引脚差了一个时钟周期


# 4.数据相关性丢失
* 产生原因：
    * 延迟与时钟不确定性令目标时钟域采样无效数据
    ![Alt text](pictures/CDC_pct/image-9.png)
* 解决方案：
    * 多位**控制信号**：格雷码编码
        * 相邻数据间转换时，只有**一位**产生变化，地址范围或状态数量为2的n次方个
        * ![Alt text](pictures/CDC_pct/image-10.png)
    * **数据信号**：
        * 异步FIFO
            * ![Alt text](pictures/CDC_pct/image-11.png)
            * 读写指针在**格雷码**编码后同步
            * “空信号”在读时钟域产生
            * “满信号”在写时钟域产生
        * 握手协议
            * ![Alt text](pictures/CDC_pct/image-12.png) 
            * “req”信号同步至clk_b；
            * “ack”信号同步至clk_a；
            * 数据保持直到握手完成
            * ![Alt text](pictures/CDC_pct/image-13.png)
        * 使能信号控制
            * ![Alt text](pictures/CDC_pct/image-14.png)
            * ![Alt text](pictures/CDC_pct/image-15.png)

# 5.复位信号的同步（异步复位同步释放）
* 复位时，复位信号不受时钟影响， 可在任意时刻低电平进行复位，不需要与时钟同步；
* 释放时，复位信号释放时，需与时钟信号同步，即刚好跟时钟同沿；
* ![Alt text](pictures/CDC_pct/image-16.png)

# 6.无毛刺时钟切换（Glitch Free Clock MUX）
* 利用**纯组合逻辑**来设计时钟切换电路，则不可避免产生毛刺；
* 两个时钟的频率相关 
    * 解决方案：在每个时钟源头选择路径上插入一个下降沿有效的D触发器；
    * ![Alt text](pictures/CDC_pct/image-17.png)

    利用时钟的下降沿寄存选择信号，保证了选择信号不会在两个时钟高电平的时候进行跳变，防止输出时钟被截断。

    * ![Alt text](pictures/CDC_pct/image-18.png)
* 两个时钟的频率无关
    * 解决方案：异步行为可能来自于select，因此在每个时钟源选择路径上插入一个上升沿有效的D触发器来避免亚稳态
    * ![Alt text](pictures/CDC_pct/image-19.png)

    利用上升沿触发的寄存器（可以使用两级），寄存选择信号，降低由于异步select导致的亚稳态，同时配合下降沿触发的寄存器，使切换都发生在时钟低电平处。降低毛刺产生的概率。

    * ![Alt text](pictures/CDC_pct/image-20.png)

# 同步常见错误用例
* 存在信号未同步
    * ![Alt text](pictures/CDC_pct/image-21.png)DC_pct/image-21.png)
* 两个或多个时钟源的数据在同步之前相遇
    * a. ![Alt text](pictures/CDC_pct/image-22.png)
    * b. ![Alt text](pictures/CDC_pct/image-23.png)