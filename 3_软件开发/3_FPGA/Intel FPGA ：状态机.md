# Intel FPGA：状态机

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - 芯片型号：Intel EP4CE10F17C8(Cyclone IV E)
   - 《数字电子技术基础》-阎石
   - 《FPGA自学笔记---设计与验证》袁玉卓，曾凯锋，梅雪松
   - 《Verilog 数字系统设计教程》夏宇闻
   - 《Verilog HDL 高级数字设计》Michael D.Ciletti
   - 《Intel FPGA/CPLD设计》（基础篇）王欣 王江宏等
   - 《Intel FPGA/CPLD设计》（高级篇）王江宏 蔡海宁等
   - 《综合与时序分析的设计约束 Synopsys设计约束（SDC）实用指南》Sridhar Gangadharan

---

## 正文

### 理论基础

状态机简写为FSM（Finite State Machine），也称为同步有限状态机，一般简称为状态机，之所以说“同步”是因为状态机中所有的状态跳转都是在时钟的作用下进行的，而“有限”则是说状态的个数是有限的。

状态机根据影响输出的原因分为两大类，即Moore型状态机和Mealy型状态机，其共同点 是：状态的跳转都只和输入有关。区别主要是在输出的时候：若最后的输出只和当前状态有关而与输入无关则称为Moore型状态机；若最后的输出不仅和当前状态有关还和输入有关则称为Mealy型状态机。而输入信号可能在一个时钟周期内任意时刻变化，这使得Mealy型状态机对输入的响应发生在当前时钟周期，比Moore有限状态机对输入信号的响应要早一个周期。因此，输入信号的噪声可能影响在输出的信号。

Moore型状态机和Mealy型状态机的区别 ：

1. Moore型状态机：输出信号只取决于当前状态。
2. Mealy型状态机：输出信号不仅取决于当前状态，还取决于输入信号的值。

它们的区别就在于输出信号是否与输入信号有关，造成的结果是：

实现相同功能时，Moore型状态机需要比Mealy型状态机多一个状态，且Moore型状态机的输出比Mealy型延后一个时钟周期。

状态机是时序逻辑电路中非常重要的一个应用，常在大型复杂的系统中使用较多。

![Intel FPGA (3)：状态机-p1](./图库/Intel FPGA (3)：状态机/Intel FPGA (3)：状态机-p1.png)

状态机的主要写法有一段式、二段式和三段式。

- 一段式指的是在一段状态机中使用时序逻辑既描述状态的转移，也描述数据的输出；

- 二段式指在第一段状态机中使用时序逻辑描述状态转移，在第二段状态机中使用组合逻辑描述数据的输出；

- 三段式指在第一段状态机中采用时序逻辑描述状态转移，在第二段在状态机中采用组合逻辑判断状态转移条件描述状态转移规律，在第三段状态机中描述状态输出，可以用组合电路输出，也可以时序电路输出。

优缺点：

- 一段式在描述大型状态机时会比较困难，会使整个系统显得十分臃肿，不够清晰；
- 二段式状态机的好处是其结构和理想的理论模型完全吻合，即不会有附加的结构存在，比较精简，但是由于二段状态机的第二段是组合逻辑描述数据的输出，所以有一些情况是无法描述的，比如输出时需要类似计 数的累加情况，这种情况在组合逻辑中会产生自迭代，自迭代在组合逻辑电路中是严格禁止的，而且第二段状态机主要是描述数据的输出，输出时使用组合逻辑往往会产生更多的毛刺，所以并不推荐。
- 所以衍生出三段式状态机，三段状态机的输出就可是时序逻辑了，但是其结构并不是最精简的了。

所以就衍生出一种新的二段式状态机。

三段式状态机的第一段状态机是用时序逻辑 描述当前状态，第二段状态机是用组合逻辑描述下一状态，如果把这两个部分进行合并而第三段状态机保持不变，就是现在最新的二段式状态机了。这种新的写法在现在不同综合器中都可以被识别出来，这样既消除了组合逻辑可能产生的毛刺，又减小了代码量，还更加容易上手，不必再去关心理论模型是怎样的，仅仅根据状态转移图就 非常容易实现。

### 按键控制LED

#### 硬件资源

![Intel FPGA (3)：状态机-p2](./图库/Intel FPGA (3)：状态机/Intel FPGA (3)：状态机-p2.png)

**按下按键S2控制4个LED左移，按下按键S3控制4个LED右移。**

当按键按下时，由于按键的硬件原因，需要消除按键带来的抖动问题，主要途径有两种：硬件消抖（RC滤波）和软件消抖（软件延时）。本文采用的是软件消抖，通过检测到按下按键之后，延迟20ms检测此时按键的状态判断按键是否按下；同时按键松开也同样进行消抖操作。

状态转移图为：

![Intel FPGA (3)：状态机-p3](./图库/Intel FPGA (3)：状态机/Intel FPGA (3)：状态机-p3.png)

状态转移条件为：

![Intel FPGA (3)：状态机-p4](./图库/Intel FPGA (3)：状态机/Intel FPGA (3)：状态机-p4.png)

按键消抖程序：

```verilog
module key_filter(
	clk,
	rst_n,
	key,
	key_state_valid_go	//1:push on;0:no push.
);

	input clk;
	input rst_n;
	input key;
	output reg key_state_valid_go;

	
	
	
	//--------key wave----------//
	//---|___________|----------//
	//**************************//

	reg [2:0] r_key_sync;
	always@(posedge clk) begin
		if(!rst_n) 
			r_key_sync <= 'd0;
		else 
			r_key_sync <= {r_key_sync[1:0],key};
	end
	
	assign w_key_pedge  = (r_key_sync[2:1]  == 2'b01);
	assign w_key_nedge  = (r_key_sync[2:1]  == 2'b10);
	
	
	localparam S_IDLE 		= 4'b0001;
	localparam S_FILTER0 	= 4'b0010;
	localparam S_DONE		 	= 4'b0100; 	
	localparam S_FILTER1 	= 4'b1000;

	reg [3:0] r_state;
	
	
	
	
	/*---------------------------------state----------------------------------------------*/
	// S_IDLE    -> S_FILTER0 : w_key_nedge == 1'b1										  //
	// S_FILTER0 -> S_IDLE    : w_key_pedge == 1'b1										  //
	// S_FILTER0 -> S_DONE	  : r_timer_cnt == TIMER_CNT_MAX - 1 && r_key_sync[2] == 1'b0 //
	// S_DONE	 -> S_FILTER1 : w_key_pedge == 1'b1										  //
	// S_FILTER1 -> S_DONE    : w_key_nedge == 1'b1										   //
	// s_FILTER1 -> S_IDLE	  : r_timer_cnt == TIMER_CNT_MAX - 1 && r_key_sync[2] == 1'b1 Z//
	
	/*---------------------------------state----------------------------------------------*/
	
	
	
	localparam TIMER_CNT_MAX = 20_000_000 / 20;//20ms
	reg [19:0] r_timer_cnt;
	reg r_cnt_en;
	
	always@(posedge clk)
	begin
		if(!rst_n)
			r_timer_cnt <= 'd0;
		else if(r_cnt_en == 1'b1)
			r_timer_cnt <= r_timer_cnt + 1'b1;
		else
			r_timer_cnt <= 'd0;
	end
	
	
	always@(posedge clk) begin
		if(!rst_n) begin
			r_state <= S_IDLE;
			r_cnt_en <= 1'b0;
			key_state_valid_go <= 1'd0;
		end
		else begin
			case(r_state)
			S_IDLE: begin
				if(w_key_nedge == 1'b1) begin
					r_state <= S_FILTER0;
					r_cnt_en <= 1'b1;
				end
				else begin
					r_state <= S_IDLE;
					r_cnt_en <= 1'b0;
					key_state_valid_go <= 1'd0;
				end
			end
			S_FILTER0: begin
				if(w_key_pedge == 1'b1) begin
					r_state <= S_IDLE;
					r_cnt_en <= 1'b0;
				end
				else if(r_timer_cnt == TIMER_CNT_MAX - 1 && r_key_sync[2] == 1'b0) begin
					r_state <= S_DONE;
					r_cnt_en <= 1'b0;
				end
				else begin
					r_state <= S_FILTER0;
					r_cnt_en <= r_cnt_en;
				end
			end
			S_DONE: begin
				if(w_key_pedge == 1'b1) begin
					r_state <= S_FILTER1;
					r_cnt_en <= 1'd1;
				end
				else begin
					r_state <= S_DONE;
					r_cnt_en <= 1'b0;
				end
			end
			S_FILTER1: begin
				if(w_key_nedge == 1'b1) begin
					r_state <= S_DONE;
					r_cnt_en <= 1'b0;
				end
				else if(r_timer_cnt == TIMER_CNT_MAX - 1 && r_key_sync[2] == 1'b1) begin
					r_state <= S_IDLE;
					r_cnt_en <= 1'b0;
					key_state_valid_go <= 1'd1;
				end
				else begin
					r_state <= S_FILTER1;
					r_cnt_en <= r_cnt_en;
				end 
			end 
			default: begin
				r_state <= S_IDLE;
				r_cnt_en <= 1'b0;
			end 
			endcase
		end 
	end
	

endmodule

```

顶层文件：

```verilog
`timescale 1ns/1ns
//function:key0 push on,led left shift.key1 push on ,led right shift.
module led(			
	clk,
	rst_n,
	key,
	led
);


	input clk;
	input rst_n;
	input [1:0] key;
	output reg [3:0] led;

	reg [31:0] r_led_cnt;
	
	localparam LED_CNT_MAX = 500_000_000/20;	//500ms
	

	always@(posedge clk or negedge rst_n)
	begin
		if(!rst_n)
			r_led_cnt <= 'd0;
		else if(r_led_cnt == LED_CNT_MAX - 1)
			r_led_cnt <= 'd0;
		else
			r_led_cnt <= r_led_cnt + 1'd1;
	end

	
	wire [1:0] w_key_state_valid_go;
	
key_filter key_filter1(	//left shift
	.clk(clk),
	.rst_n(rst_n),
	.key(key[0]),
	.key_state_valid_go(w_key_state_valid_go[0])	//1:push on;0:no push.
);	


key_filter key_filter2(	// right shift
	.clk(clk),
	.rst_n(rst_n),
	.key(key[1]),
	.key_state_valid_go(w_key_state_valid_go[1])	//1:push on;0:no push.
);
	
	reg r_shift_en;
	always@(posedge clk)
	begin
		if(!rst_n)
			r_shift_en <= 1'd0;
		else if(w_key_state_valid_go[0] == 1'b1)	
			r_shift_en <= 1'd1;
		else if(w_key_state_valid_go[1] == 1'b1)
			r_shift_en <= 1'd0;
		else
			r_shift_en <= r_shift_en; 
	end
	
	always@(posedge clk or negedge rst_n)
	begin
		if(!rst_n)
			led <= 4'b1110;
		else if(r_shift_en) begin
			if(r_led_cnt == LED_CNT_MAX - 1)
				led <= {led[0],led[3:1]};
			else
				led <= led;
		end
		else begin
			if(r_led_cnt == LED_CNT_MAX - 1)
				led <= {led[2:0],led[3]};
			else
				led <= led;
		end
	end
	
endmodule
	
```









## 总结

本工程名为led，如有需要请至github仓库查看！！！







---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**



