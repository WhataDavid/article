# article

基于Vitis HLS的FIR滤波器设计及其性能分析
张海龙(1,2,3,4)，杜旭(1,2)，张萌(1,2)，张亚州(1,2)，王杰(1,4)，
冶鑫晨(1,4)，王万琼(1)，李嘉(1)，吴涵(1,2)，张婷(1,2)
(1.中国科学院新疆天文台，新疆乌鲁木齐 830011；2.中国科学院大学，北京 100049；3.中国科学院射电天文重点实验室，南京 210008；4.国家天文科学数据中心，北京 100101)
摘要：针对硬件开发过程中存在编程效率低、开发难度高及Vivado HLS资源使用率高相关问题，本文利用Vivado及Vitis HLS平台设计并实现了基于127阶Hamming窗的流水线式直接型并行结构有限冲激响应滤波器，在Vivado Simulator环境下对比了HDL FIR IP、HLS FIR IP与XILINX FIR IP滤波表现，详细分析了不同实现方式在资源使用率、时序、功耗、执行时间等方面差异。实验结果表明，在相同条件下HLS FIR IP相较HDL FIR IP及XILINX FIR IP的资源使用率降低了1 %，执行时间分别降低了24.5 %、808.2 %，且代码量节省了98.5 %。以本文提出的实验方法为基础与前人工作进行了对比，客观分析了在一定条件下不同开发平台及方式的效率差异，结果表明本文的设计方法可显著降低逻辑单元与存储资源的使用率并提升开发效率。
关键词：天文信息技术；高层次综合；FIR滤波器；Vitis HLS；Verilog
中图分类号：TP301.6         文献标志码：A                 文章编号：
DOI：10.13229/j.cnki.jdxbgxb
Filter Design and Performance Analysis Based on Vitis HLS
ZHANG Hai-long(1,2,3,4)，DU Xu(1,2)，ZHANG Meng(1,2)，ZHANG Ya-zhou(1,2)，WANG Jie(1,4) ，
YE Xin-chen(1,4)，WANG Wan-qiong(1)，Li Jia(1)，WU Han(1,2)，ZHANG Ting(1,2)
(1. Xinjiang Astronomical Observatory, Chinese Academy of Sciences, Urumqi 830011,China；2. University of
Chinese Academy of Sciences, Beijing 100049,China；3. Key Laboratory of Radio Astronomy, Chinese Academy of
Sciences, Nanjing 210008,China；4. National Astronomical Data Center, Beijing 100101, China)
Abstract: To address the problems of low programming efficiency, high development difficulty and high Vivado HLS resource utilization in the hardware development process, this paper designs and implements a pipelined direct parallel finite impulse response filter with a 127-order Hamming window using Vivado and Vitis HLS platforms. And compares the filtering performance of HDL FIR IP, HLS FIR IP, and XILINX FIR IP in the Vivado Simulator environment. A detailed analysis is conducted on the differences in resource utilization, timing, power consumption and execution time among different implementation methods. The experimental results show that under the same conditions, the resource utilization of HLS FIR IP is reduced by 1% compared with HDL FIR IP and XILINX FIR IP, the execution time is reduced by 24.5% and 808.2% respectively, and the code size is saved by 98.5%. Based on the experimental method proposed in this paper, we make a comparison with previous work and objectively analyze the efficiency difference of different development platforms and methods under certain conditions. The results show that our design method can significantly reduce the usage of logic units and storage resources and improve the development efficiency.
Key words：Astroinformatics; High-level synthesis; FIR Filter; Vitis HLS; Verilog

0 引言
现场可编程门阵列(FPGA，Field Programmable Gate Array)器件是XILINX公司1984年首家推出的一种介于专用集成电路与中央处理器的产物，开发者能够根据应用及功耗重新配置FPGA架构，其灵活性与可重配置性使得硬件结构更加适配计算特性([1])。随着FPGA与机器学习、通信行业的发展，越来越多的学者与产业工作者意识到硬件描述语言（HDL，Hardware Description Language）所带来的开发效率低下的问题([2])，不断探索新的提高开发效率的方法。可将高级语言代码自动转化为HDL代码的高层次综合(HLS，High Level Synthesis)工具([)(3)(])受到越来越多人的关注。本文系统分析了Vitis HLS高层次综合工具，其能将C、C++([4])函数自动转化为HDL代码，并将其综合为寄存器传输级(RTL，Register Transfer Level)知识产权(IP，Intellectual Property)([5])，从而在可编程逻辑中加速开发并降低成本。
HLS生成的IP核与HDL编写的IP核所表现性能存在差异，Lan Huang等人提出了HLS性能评估方程来描述性能优化与结果质量之间的关系，其主要与频率、延迟、吞吐量、资源使用率及功耗相关([6])。Roberto Millón等人利用HDL与HLS在Vivado平台进行图像边缘检测应用开发，并对其进行资源使用率、执行时间、开发时间的实验比较，获得了更高的开发效率，但资源使用率较高且执行时间较长([7])。S. Srilakshmi分别用VHDL与Vitis HLS对CNN卷积层进行比较，通过在HLS中使用循环展开及流水线优化指令，可达到与HDL相当的性能水平，但由于其卷积层较小，资源使用率约为1 %([8])。
随着射电观测设备获取数据的能力显著提高，如何实现海量天文信号高速数据流的实时传输成为棘手问题。目前主流方法是利用多相滤波器组(PFB，Polyphase Filter Banks)对数据流进行分通道([)(9)(])，并对子带进行VDIF格式、UDP/IP协议栈打包，通过以太网进行传输([1)(0)(])。整个过程需消耗大量逻辑资源([1)(1)(])，对于资源紧缺的FPGA，需要尽可能降低资源使用率及延迟、功耗等。多相滤波器组结构中存在多个有限冲激响应(FIR，Finite Impulse Response)滤波器([1)(2)(])，单FIR滤波器性能直接影响多相滤波器组性能，故需对FIR滤波器进行性能分析并优化。
针对上述问题，本文通过利用Verilog及C++分别在Vivado和Vitis HLS上实现了基于Hamming窗的127阶直接型有限冲激响应滤波器，并对使用相同算法且滤波结果一致的XILINX FIR IP核进行资源使用率、时序、能耗等方面比较。通过实验测试并对比实验结果，本文得出了Vivado及Vitis HLS实现的FIR滤波器与XILINX FIR IP三者在资源使用率、时序、能耗等方面的差异，并分析了各自的优势。
1 FIR滤波器设计
1.1 窗函数分析
常见的窗函数有Rectangle窗、Hanning窗、Hamming窗、Blackman窗及Kaiser窗。不同窗函数的频率响应特点如图1所示，综合考虑幅值失真度与频率分辨率及计算复杂度，本文选取Hamming窗实现算法，其窗函数计算公式如公式1所示，w(n)表示窗函数值，n表示窗函数下标，N表示窗函数长度。使用Hamming窗函数对理想低通滤波器脉冲响应进行截断和平滑，其脉冲响应如公式2所示。h(d)(n)表示理想低通滤波器脉冲响应，其计算表达式如公式3所示，ω(c)表示截止频率，M=N−1表示FIR滤波器阶数。Hamming窗FIR滤波器的频率响应如公式4所示，其中ω表示归一化角频率。
 $$w(n)=0.54-0.46\cos{(}\frac{2\pi n}{N-
1}\text) \ 0≤n≤N-1$$(1）
 $$h(n)=w(n)h_d(n) \ 0≤n≤N-1$$ (2）
 $$h_d(n)=\frac{sin(ω_c(n-M/2))}{\pi (n-M/2)} \ 0≤n≤N-1$$(3）
$$H(ω)=\sum_{n=0}^{N-1}h(n)e^{-jωn} \ ω \in[0,\pi]$$(4）

 
 
图1 不同窗函数的频率响应比较
Fig.1 Comparison of frequency response of different window function

1.2 FIR滤波器结构设计
FIR滤波器由数字乘法器、加法器及延时单元组成，其可通过离散卷积实现滤波操作。因其具有稳定、易实现、线性相位等特点，在信号处理领域获得广泛应用。FIR滤波器是多相滤波器组中的重要组成部分，其在结构简单的基础上可通过提高阶数来提升资源使用率。其设计如公式5所示，其中k表示样本索引号，y(k)表示滤波后的信号。n表示冲激响应的下标，n∈[0,127]，h(n)表示FIR滤波系数，x(k-n)表示输入信号。N表示窗函数长度也是滤波器系数个数，滤波器阶数为N-1。
  $$y(k)=\sum_{n=0}^{N-1}h_nx(k-n) \ k=1,2,3,…$$                   （5）
本文使用127阶直接型并行结构FIR滤波器，结构如图2所示，127阶FIR滤波器由127个加法器、128个乘法器及127个延时单元实现，图2中＋为加法器，×为乘法器，D为延时单元。设计中令FIR滤波器以并发方式执行数据读取与卷积运算操作，无需等待所有数据读取完成，以缩短启动时间间隔，提高数据吞吐量。并使用定点算法，提升运算效率。
图2 直接型FIR滤波器结构
Fig.2 Structure of direct FIR filter
1.3 Vitis HLS FIR IP设计 
利用C++在Vitis HLS中为器件设计并开发FIR RTL IP核的流程如图3所示。基于设计原则在高抽象层建立C++ FIR算法架构；利用激励文件编译后进行仿真，在行为级别验证滤波功能，若滤波的仿真结果正确则进行综合，在给定80 MHz时钟速度的输入约束下将C++ FIR代码转换成RTL FIR代码，进行C++/RTL协同仿真，以验证RTL实现的结果是否与原始C++实现的结果相同，若相同则将其转换为FIR RTL IP核。
图3 高层次综合生成FIR IP核流程
Fig.3 High level synthesis flowchart
2 FIR IP实现
2.1 基于Verilog HDL实现FIR IP
Verilog实现FIR滤波器IP核主要分为三个步骤，在初始化后首先需要根据生产者使用者原则及流水线原则将含标记信息的随机噪声信号依次读入到delay_pipeline数组中，将读入的待滤波数据分别与滤波器系数相乘再相加，以达到卷积滤波的目的。在计算最终结果时，以树状结构，分多个周期进行流水，优化时序。本文实现的HDL FIR IP代码及整体工程已在码云上开放获取。在Vivado的块设计图中利用Block Memory Generator读取含标记信息的随机噪声信号，为FIR滤波器提供16 bit位宽待滤波数据。添加HDL编写的HDL FIR IP到块设计图中并连线，通过HDL FIR IP进行滤波并在Vivado Simulator下进行仿真实验。系统参数如表1所示，HDL FIR块设计图如图4所示。
表1 系统参数表
Table 1 System parameter table
时钟频率(MHz)
数据输入位宽（bit）
数据输出位宽（bit）
80
16
35
图4 HDL FIR块设计图
Fig.4 HDL FIR block design diagram
2.2 基于Vitis HLS实现FIR IP
利用#pragma HLS PIPELINE编译指令将循环读入与循环处理的后续迭代发生重叠，以并发方式执行。添加rewind关键字将已流水打拍的循环回绕以保障流水性能。利用#pragma HLS interface编译指令将FIR输入、输出接口定义为数据接口，无需使用协议，避免复杂的握手信号。利用#pragma HLS ARRAY_PARTITION编译指令将原始系数阵列及结果阵列拆分为较小的阵列及独立的寄存器，提高待滤波数据吞吐量。已打包的HLS FIR IP核通过IP Repository加载到Vivado IP Catalog库中，同样在块设计图中利用Block Memory Generator加载含标记信息的随机噪声信号，通过HLS FIR IP进行滤波并在Vivado Simulator下进行仿真实验。HLS FIR块设计图如图5所示。




图5 HLS FIR块设计图
Fig.5 HLS FIR block design diagram
2.3 XILINX FIR IP
在Vivado块设计图中调用FIR Compiler IP核，并在Filter Options页面配置其滤波系数与前两个FIR滤波器系数相同。在Channel Specification页面设置FIR滤波器输入样本频率为80 MHz，时钟频率为80 MHz，此情况下FIR Compiler IP为流水线式直接型结构。在Implementation页面设置滤波系数位宽为16 bit，输出的舍入模式为全精度，Optimization Goal为Speed模式。添加Block Memory Generator IP，加载含标记信息的随机噪声信号并传输给XILINX FIR IP进行滤波，在Vivado Simulator下进行仿真实验。XILINX FIR块设计图如图6所示。




图6 XILINX FIR块设计图
Fig.6 XILINX FIR block design diagram
3 FIR IP仿真
3.1 实验数据
本文所使用的数据为随机噪声信号，采样频率为80 MHz。对其分成8个通道，并对每个通道内的时域噪声信号添加高强度标记信息，以标记不同信道，各通道内标记信息的个数分别与其通道序号相同。利用10 MHz截止频率，80 MHz采样频率FIR滤波器对其进行滤波，期望允许10 MHz以下信号通过，滤除10 MHz以上的噪声，保留前2个通道数据，即保留通道内标记信息个数分别为1和2的通道数据。
3.2 滤波结果
通过Vivado Simulator对滤波器的滤波结果进行仿真，可得到如图7所示的滤波结果，图中从上至下的波形分别为含标记信息的随机噪声信号滤波前时域数据、HDL FIR IP滤波后时域结果、HLS FIR IP滤波后时域结果、XILINX FIR IP滤波后时域结果。分别对其滤波后结果进行傅里叶变换，使其在频域展示滤波后结果，并与未滤波数据频域图进行比较，可得到图8，图中从上至下的波形分别为含标记信息的随机噪声信号滤波前频域数据、HDL FIR IP滤波后频域结果、HLS FIR IP滤波后频域结果、XILINX FIR IP滤波后频域结果。由图7、8可知三种滤波器对基带数据都有很好的滤波效果，在数据比对中，三种FIR滤波器的滤波结果一致，实现了对10 MHz以下频率数据允许通过，10 MHz以上频率数据进行滤除。
 

 
图7 滤波结果时域对比图
Fig.7 Comparison of filtering results in time domain
 
图8 滤波结果频域对比图
Fig.8 Comparison of filtering results in frequency domain

4 性能评估 
4.1 资源使用
利用Vivado对三种FIR IP进行综合并实现，使用report_utilization Tcl命令可获得各IP核所使用资源数量及各自占资源总数的百分比，如图9、图10所示。
本文在RFSoC ZCU111器件的仿真环境下执行，优化策略均采用默认策略，为均衡考量整体资源的使用率，本文提出资源使用率之和计算式用以量化不同实现方式的资源使用率差异。假定U为所使用的不同资源使用率之和，共使用z种资源，r(n)代表第n种资源的使用量，r(nsum)代表第n种资源的总数。通过计算不同资源消耗数量与该资源总量之比，可得到三种FIR滤波器的资源使用率占比之和分别为19.83 %、18.86 %、19.85 %，其计算式如公式6所示。
                        (6)
从实验结果图10中可以得知本文中HDL FIR IP与XILINX FIR IP在资源使用数量方面较HLS FIR IP多1 %，约为其本身所使用资源的5 %。LUT为查找表，是FPGA中最基本的存储资源，可存储输入到输出的映射关系。LUTRAM用作RAM存储单元，通常存储中间结果，本文设计中XILINX FIR IP使用了1867个LUTRAM，使用量分别为HDL FIR IP及HLS FIR IP的117倍、52倍。FF为触发器，通常用于存储状态及时序信息。BRAM为块RAM，具有高速、大容量的特点；IO为FPGA与外界交互的接口；BUFG为全局时钟缓冲，可实现控制时钟功能。三种FIR IP核在BRAM、IO与BUFG资源上的使用接近，仅有个位数差距。HLS FIR IP在DSP资源使用率上比HDL FIR IP少1.7 %，XILINX FIR IP只使用了1.5 %DSP，但使用了更多的IO。

 
图9 资源使用数量对比图
Fig.9 Comparison chart of resource usage quantity
 
图10 资源使用率对比图
Fig.10 Comparison chart of resource utilization rate

4.2 时序
经综合并实现后使用report_timing_summary Tcl命令可获得FIR IP时序报告，三种FIR IP的时序结果均为正值，都能达到时序要求，详细数据如表2所示。最差负时序裕量(WNS，Worst Negative Slack)表示设计中最紧迫的时序路径的负迟滞值，即该路径的最小要求与实际延迟之间的差值，其影响了时钟频率的提升。本文中由Vitis生成的HLS FIR IP与其它两种FIR IP在WNS方面有明显差距，HLS FIR IP的最差负时序裕量与其它两种FIR IP滤波器的均值相差约4.677 ns，为自身的202 %。最差保持时序裕量(WHS，Worst Hold Slack)表示时钟与数据的相对时序关系，即所需保持时间与实际保持时间之差，WHS越小，时钟与数据之间干扰越严重，XILINX FIR IP在WHS方面劣于其它两种FIR滤波器，与其平均WHS相差0.012 ns，为自身的209 %。最差脉冲宽度时序裕量(WPWS，Worst Pulse Width Slack)表示时钟脉冲宽度与规定脉冲宽度之间的偏差，WPWS越小，表示信号的脉冲宽度越不稳定，越容易出现时序问题，三种FIR IP在WPWS方面没有差异。
表2 时序比较表
Table 2 Time series comparison table
比较项
HDL FIR IP
HLS FIR IP
XILINX FIR IP
最差负时序裕量(WNS)
8.79 ns
4.582 ns
9.728 ns
最差保持时序裕量(WHS)
0.022 ns
0.024 ns
0.011 ns
最差脉冲宽度时序裕量(WPWS)
5.677 ns
5.677 ns
5.677 ns
时序裕量之和
14.489 ns
10.283 ns
15.416 ns
4.3 功耗
经综合并实现后使用report_power Tcl命令可获得FIR IP功耗报告，三种FIR滤波器在预计结温、热裕度及有效热阻方面表现接近一致。XILINX FIR IP在片上总功耗方面低于HDL FIR IP 0.106 W，约为自身的9.3 %，低于HLS FIR IP 0.091 W，约为自身的8 %，其数据如表3所示。
表3 功耗比较表
Table 3 Power consumption comparison table
比较项
HDL FIR IP
HLS FIR IP
XILINX FIR IP
片上总功耗
Total On-Chip Power
1.243 W
1.228 W
1.137 W
预计结温
Junction Temperature
26.0 ℃
26.0 ℃
26.0 ℃
热裕度
Thermal Margin
74.0 ℃
74.0 ℃
74.0 ℃
有效热阻
Effective
0.8 ℃/W
0.8 ℃/W
0.8 ℃/W
4.4 执行时间
利用Block Memory Generator在100 ns时开始为FIR滤波器传输待滤波数据，HDL FIR IP与HLS FIR IP分别在接收数据122 ns、98 ns后开始输出数据，XILINX FIR IP输出数据最晚，在接收数据后890 ns时输出数据。HLS FIR IP相较HDL FIR IP提前了24 ns，执行时间降低了24.5%；相较XILINX FIR IP提前了792 ns，执行时间降低了808.2 %，其执行时间波形图及执行时间表如图11、表4所示。
图11 执行时间波形图




Fig.11 Arrival time waveform
表4 执行时间表
Table 4 Execution time
比较项
HDL FIR IP
HLS FIR IP
XILINX FIR IP
执行时间
122ns
98ns
890ns
4.5 代码量
由Verilog实现127阶FIR滤波器所需代码量巨大，耗费大量时间，使用树形结构并遵循生产者使用者原则与流水线原则的情况下除去注释及空行共需2397行代码。利用Vitis HLS对C++ FIR代码进行综合则方便很多，只需36行即可得到相同的滤波结果，代码量节省了98.5 %，有效提升开发效率。
表5 127阶FIR IP的代码量比较表（不含注释及空行）
Table 5 Code Quantity Comparison Table for 127th Order FIR IP (excluding comments and blank lines)
比较项
HDL FIR IP
HLS FIR IP
代码行（LOC）
2397
36

4.6 性能结果比较
Michael D. Zwagerman利用Verilog与Vivado HLS 2013.4实现图像滤波，比较HDL与HLS实现方式所使用资源数量，结果表明HLS方式比HDL方式资源使用率之和多3.3 %。Roberto Millón利用VHDL与Vivado HLS 2019.1进行图像边缘检测开发，结果表明HLS方式比HDL方式资源使用率之和多5.5 %。S. Srilakshmi利用Verilog与Vitis HLS 实现CNN卷积层，在Vivado 2018.1平台比较HDL与HLS实现方式所使用资源数量，结果表明HLS方式比HDL方式资源使用率之和多1.7 %。本文中使用Vitis HLS 2022.1在资源使用率方面的结果已经优于HDL实现方式与XILINX IP，优势约为1 %。Michael D. Zwagerman与本文的研究中，HDL在时序的表现均优于HLS。Roberto Millón使用VHDL及Vivado HLS 2019.1比较的结果表明HLS实现方式比HDL实现方式执行时间长24.9 ms，本文中Vitis HLS 2022.1实现方式比Verilog实现方式快24 ns，且远优于XILINX FIR IP。所有相关研究成果表明HLS可明显提高开发效率，降低开发难度。Vitis HLS 2022.1引入新的特性，HLS尝试使用最少的资源来达标([)(14])，为模块数据流类型提供阵列分区支持，可将大型阵列处理分解成更小的流水线片段并改进PL分析，因此可获得更高的性能([15])。其比较结果如表6所示。

表6 127不同实现方式下HDL与HLS的性能比较
Table 6 Performance comparison table of HDL and HLS under different implementation methods
研究者
HDL语言
HLS工具
比较项
资源使用率之和
时序裕量之和
或时钟速度
总功耗
执行时间
编程时间
或代码行
Michael D. Zwagerman([13])
Verilog
Vivado HLS 2013.4
图像滤波
HDL:129.8%
HLS:133.1%
HDL:200MHz
HLS:181MHz
-
-
HDL:33h
HLS:15h
Roberto Millón
VHDL
Vivado HLS 2019.1
Sobel边缘检测
HDL:2.9%
HLS:8.4%
-
-
HDL:58ms
HLS:82.9ms
HDL:493LOC
HLS:90LOC
S. Srilakshmi
VHDL
Vitis HLS
N/A
CNN卷积层
HDL:0.7%
HLS:2.4%
-
-
-
-
本文
Verilog
Vitis HLS 2022.1
FIR滤波器
HDL:19.83%
HLS:18.86
XILINX:19.85%
HDL:14.5ns
HLS:10.3ns
XILINX:15.4ns
HDL:1.243W
HLS:1.228W
XILINX:1.137W
HDL:122ns
HLS:98ns
XILINX:890ns
HDL:2397LOC
HLS:36LOC

5 结论
本文基于127阶Hamming窗流水线式直接型结构设计了FIR滤波器IP核，利用含有标记信号的仿真数据进行了实验测试，在严格控制差异的条件下量化了HDL、HLS实现方式与XILINX IP核在资源使用率、时序、功耗、执行时间及代码量等方面的差异并与前人相关工作进行了比对，在滤波器结构、滤波结果相一致的前提下HLS FIR IP资源使用率之和更低，执行时间最短，开发速度与HDL FIR IP相比更快。HDL FIR IP在时序方面优势明显且更加灵活，但代码冗长，迭代慢。XILINX FIR IP在功耗方面表现最优，但其具体实现细节未知，安全性不可控。在宽带信号子带划分情况下可优先使用Vitis HLS编写IP核，应用于临界采样或过采样多相滤波器组及其它功能的实现，若需完全定制化并尽可能提升时序，则建议使用硬件描述语言实现相关功能。本文创新性提出了基于FIR滤波器分析不同硬件实现方式性能的评估方法，测试及结果分析方法在宽带信号处理及其它FPGA开发领域具备一定实际应用价值。
 
致谢
本论文实验过程中使用到的软硬件环境来自中国科学院国家天文台，感谢段然老师及其团队成员的支持和指导。感谢 Xilinx 公司提供的免费Vitis HLS工具。
 
参考文献
[1]周柚，杨森，李大琳，等. 基于现场可编程门电路的人脸检测识别加速平台[J]. 吉林大学学报：工学版，2019,，49(6):7.
Zhou You, Yang Sen, Li Da-lin, et al. Acceleration platform for face detection and recognition based on field programmable gate array [J]. Journal of Jilin University (Engineering and Technology Edition), 2019, 49 (6): 2051-2057.
[2]汤嘉武，郑龙，廖小飞，等.面向高性能图计算的高效高层次综合方法[J].计算机研究与发展，2021，58(3):467-478.
Tang Jia-wu, Zheng Long, Liao Xiao-fei, Jin Hai, et al. Effective High-Level Synthesis for High-Performance Graph Processing [J]. Journal of Computer Research and Development, 2021, 58 (3): 467-478.
[3]Schledt D, Kebschull U, Blume C. Developing a cluster-finding algorithm with Vitis HLS for the CBM-TRD[J]. Nuclear Instruments and Methods in Physics Research Section A: Accelerators, Spectrometers, Detectors and Associated Equipment, 2023, 1047: 167797.
[4]Xilinx Inc. Versal ACAP Hardware, IP, and Platform Development Methodology Guide (UG1387) [EB/OL].[2022-11-16]
https://docs.xilinx.com/r/en-US/ug1387-acap-hardware-ip-platform-dev-methodology/Design-Creation-with-Vitis-HLS
[5]Xilinx Inc. Vivado Design Suite User Guide: Designing with IP(UG896)[EB/OL].[2022-11-2]
https://www.xilinx.com/content/dam/xilinx/support/documents/sw_manuals/xilinx2022_2/ug896-vivado-ip.pdf
[6]Lan Huang, LI Da-lin, Wang Kang-ping, et al. A survey on performance optimization of high-level synthesis tools[J]. Journal of Computer Science and Technology, 2020, 35: 697-720.
[7]Millón R, Frati E, Rucci E. A comparative study between HLS and HDL on SoC for image processing applications[J]. arXiv preprint arXiv:2012.08320, 2020.
[8]Srilakshmi S, Madhumati G L. A Comparative Analysis of HDL and HLS for Developing CNN Accelerators[C]//2023 Third International Conference on Artificial Intelligence and Smart Energy (ICAIS), Coimbatore, India, 2023: 1060-1065.
[9]张海龙，张萌，张亚州，等.基于临界采样多相滤波器组的宽带信号信道化器设计与实现[J/OL].吉林大学学报(工学版):1-13[2023-04-06].DOI:10.13229/j.cnki.jdxbgxb20200274.
Zhang Hai-long, Zhang Meng, Zhang Ya-zhou, et al. Design and implementation of wideband signal channelizer based on critical sampling polyphase filter bank [J/OL]. Journal of Jilin University (Engineering and Technology Edition): 1-13 [2023-04-06]. DOI: 10.13229/j.cnki.jdxbgxb20200274.
[10]Nosov E, Marshalov D, Fedotov L, et al. Multifunctional digital backend for quasar VLBI network[J]. Journal of Instrumentation, 2021, 16(05): P05003.
[11]Smith J P, Bailey J I, Tuthill J, et al. A high-throughput oversampled polyphase filter bank using vivado hls and pynq on a rfsoc[J]. IEEE Open Journal of Circuits and Systems, 2021, 2: 241-252.
[12]Hossain K, Ahmed R, Haque A, et al. A Review of Digital FIR Filter Design in Communication Systems[J]. International Journal of Science and Research (IJSR), 2021, 10(2):09.
[13]Zwagerman M D. High level synthesis, a use case comparison with hardware description language[J]. 2015.
[14] Xilinx Inc.2022.1 Release Highlights [DB/OL].[2023-05-16] https://xilinx.com/products/design-tools/vitis/vitis-hls.html#2022_1
[15] Xilinx Inc. What's New for the Vitis Software Platform [DB/OL].[2023-05-16] 
https://china.xilinx.com/products/design-tools/vitis/vitis-whats-new.html#20221
