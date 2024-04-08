# Intel FPGA：adc adc128s102

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



**模数转换器**（英语：**analog to Digital converter**，英文缩写：**ADC**）是一种将模拟信号（例如麦克风拾取的声音或进入数码相机的光线）转换为数字信号的系统。

SPS(sample per second，每秒采样次数)，是衡量模数转换（ADC）时采样速率的单位。采样率定义为对输入信号输入信的采样频率，采样率不仅表示模数转换器的转换速度，同时也决定了系统可处理信号的带宽范围。

比如这款ADC芯片adc128s102,使用的是SPI进行传输,完整传输一次数据所需要花费的时间是16个sclk周期。

这里假设sclk的时钟频率为8MHz
$$
一次采样周期=16*\frac{1}{f_{sclk}}=16*125ns = 2000ns\\
SPS = \frac{1 s}{2000 ns} = 500 000 = 500KSPS
$$
本篇采用的ADC芯片是adc128s102。这是一款8通道，500KSPS到1MSPS，12bit的ADC。

### 硬件电路

![Intel FPGA (6)：adc adc128s102-p1](./图库/Intel FPGA (6)：adc adc128s102/Intel FPGA (6)：adc adc128s102-p1.png)

### ADC128S102

这里截取了AC128S102的部分数据手册，读者自行阅读。

![Intel FPGA (6)：adc adc128s102-p2](./图库/Intel FPGA (6)：adc adc128s102/Intel FPGA (6)：adc adc128s102-p2.png)

![Intel FPGA (6)：adc adc128s102-p3](./图库/Intel FPGA (6)：adc adc128s102/Intel FPGA (6)：adc adc128s102-p3.png)

![Intel FPGA (6)：adc adc128s102-p4](./图库/Intel FPGA (6)：adc adc128s102/Intel FPGA (6)：adc adc128s102-p4.png)

![Intel FPGA (6)：adc adc128s102-p5](./图库/Intel FPGA (6)：adc adc128s102/Intel FPGA (6)：adc adc128s102-p5.png)

![Intel FPGA (6)：adc adc128s102-p6](./图库/Intel FPGA (6)：adc adc128s102/Intel FPGA (6)：adc adc128s102-p6.png)

**ADC为12位分辨率，因此1bit代表的电压值为$V_{A}$/4096。当模拟输入电压低于$V_{A}$/8192时，输出数据为0000_0000_0000b。同理由于芯片本身内部构造当输出数据0000_0000_0000b变为0000_0000_0001b时，实际输入电压变化为$V_{A}$/8192而不是$V_{A}$/4096。当输入电压大于或等于$V_{A}$-1.5*$V_{A}$/4096时，输出数据即为1111_1111_1111b。**

### 代码展示

adc128s102底层驱动代码：

```verilog
module adc128s102_driver(
	clk,
	rst_n,
	adc_addr,
	adc_convert_en_go,
	adc_cs_n,
	adc_sclk,
	adc_dout,
	adc_din,
	adc_convert_busy,
	adc_data,
	adc_data_convert_valid_go
);

	input clk;
	input rst_n;
	
	input [2:0] adc_addr;
	input adc_convert_en_go;
	
	output adc_cs_n;
	output adc_sclk;
	input  adc_dout;
	output adc_din;
	output adc_convert_busy;
	
	output [11:0] adc_data;
	output adc_data_convert_valid_go;
	

	reg [2:0] r_adc_addr;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_adc_addr <= 3'd0;
		else if(adc_convert_en_go == 1'b1)
			r_adc_addr <= adc_addr;
		else
			r_adc_addr <= r_adc_addr;
	end
	
	reg r_adc_convert_en;
	wire w_adc_convert_end;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_adc_convert_en <= 1'b0;
		else if(adc_convert_en_go == 1'b1)
			r_adc_convert_en <= 1'b1;
		else if(w_adc_convert_end == 1'b1)
			r_adc_convert_en <= 1'b0;
		else
			r_adc_convert_en <= r_adc_convert_en;
	end
	
	assign adc_convert_busy = r_adc_convert_en;

	

	
	localparam SYSCLK = 50_000_000;
	localparam SPI_CLK = 6_250_000;
	localparam SPI_CLK_DR = SYSCLK / SPI_CLK;
	
	reg [$clog2(SPI_CLK_DR)-1:0] r_div_cnt;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_div_cnt <= 'd0;
		else if(r_adc_convert_en == 1'b1) begin
			if(r_div_cnt == SPI_CLK_DR - 1'b1)
				r_div_cnt <= 'd0;
			else
				r_div_cnt <= r_div_cnt + 1'b1;
		end
		else
			r_div_cnt <= 'd0;
	end
	wire w_sclk_pluse;
	assign w_sclk_pluse = (r_div_cnt == 'd1) ? 1'b1 : 1'b0; 
	
	
	
	
	reg [5:0] r_bit_cnt;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_bit_cnt <= 6'd0;
		else if(r_adc_convert_en == 1'b1) begin
			if(w_sclk_pluse == 1'b1)
				r_bit_cnt <= r_bit_cnt + 1'b1;
			else
				r_bit_cnt <= r_bit_cnt;
		end
		else
			r_bit_cnt <= 6'd0;
	end
	assign w_adc_convert_end = (r_bit_cnt == 6'd35) ? 1'b1 : 1'b0;
	
	
	
	//fpga negedge output addr data,adc posedge collect addr data
	//fpga posedge collect adc data,adc negedge output adc data
	reg r_adc_cs_n;
	reg [11:0] r_adc_data;
	reg r_adc_sclk;
	reg r_adc_din;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_adc_data <= 12'd0;
			r_adc_cs_n <= 1'd1;
			r_adc_sclk <= 1'b1;
			r_adc_din <= 1'b1;
		end
		else begin
			case(r_bit_cnt)
			6'd0: begin r_adc_cs_n <= 1'b1; r_adc_sclk <= 1'b1; r_adc_din <= 1'b1; end
			6'd1: begin r_adc_cs_n <= 1'b0; r_adc_sclk <= 1'b1; r_adc_din <= 1'b1; end
			6'd2: begin r_adc_sclk <= 1'b0; r_adc_din <= 1'b1; end
			6'd3: begin r_adc_sclk <= 1'b1; r_adc_din <= 1'b1; end
			6'd4: begin r_adc_sclk <= 1'b0; r_adc_din <= 1'b1; end
			6'd5: begin r_adc_sclk <= 1'b1; r_adc_din <= 1'b1; end
			6'd6: begin r_adc_sclk <= 1'b0; r_adc_din <= r_adc_addr[2]; end
			6'd7: begin r_adc_sclk <= 1'b1; end
			6'd8: begin r_adc_sclk <= 1'b0; r_adc_din <= r_adc_addr[1]; end
			6'd9: begin r_adc_sclk <= 1'b1; end	
			6'd10: begin r_adc_sclk <= 1'b0; r_adc_din <= r_adc_addr[0]; end
			6'd11: begin r_adc_sclk <= 1'b1; r_adc_data[11] <= adc_dout; end
			6'd12: begin r_adc_sclk <= 1'b0; end
			6'd13: begin r_adc_sclk <= 1'b1; r_adc_data[10] <= adc_dout; end		
			6'd14: begin r_adc_sclk <= 1'b0; end
			6'd15: begin r_adc_sclk <= 1'b1; r_adc_data[9] <= adc_dout; end	
			6'd16: begin r_adc_sclk <= 1'b0; end
			6'd17: begin r_adc_sclk <= 1'b1; r_adc_data[8] <= adc_dout; end	
			6'd18: begin r_adc_sclk <= 1'b0; end
			6'd19: begin r_adc_sclk <= 1'b1; r_adc_data[7] <= adc_dout; end				
			6'd20: begin r_adc_sclk <= 1'b0; end
			6'd21: begin r_adc_sclk <= 1'b1; r_adc_data[6] <= adc_dout; end			
			6'd22: begin r_adc_sclk <= 1'b0; end
			6'd23: begin r_adc_sclk <= 1'b1; r_adc_data[5] <= adc_dout; end			
			6'd24: begin r_adc_sclk <= 1'b0; end
			6'd25: begin r_adc_sclk <= 1'b1; r_adc_data[4] <= adc_dout; end				
			6'd26: begin r_adc_sclk <= 1'b0; end
			6'd27: begin r_adc_sclk <= 1'b1; r_adc_data[3] <= adc_dout; end			
			6'd28: begin r_adc_sclk <= 1'b0; end
			6'd29: begin r_adc_sclk <= 1'b1; r_adc_data[2] <= adc_dout; end	
			6'd30: begin r_adc_sclk <= 1'b0; end
			6'd31: begin r_adc_sclk <= 1'b1; r_adc_data[1] <= adc_dout; end		
			6'd32: begin r_adc_sclk <= 1'b0; end
			6'd33: begin r_adc_sclk <= 1'b1; r_adc_data[0] <= adc_dout; end	
			6'd34: begin r_adc_cs_n <= 1'b1; r_adc_sclk <= 1'b1; r_adc_din <= 1'b1;end	
			6'd35: begin r_adc_cs_n <= 1'b1; r_adc_sclk <= 1'b1; r_adc_din <= 1'b1;end		
			default:begin r_adc_cs_n <= 1'd1;r_adc_sclk <= 1'b1; r_adc_din <= 1'b1;end
			endcase
		end
	end
	
	assign adc_cs_n = r_adc_cs_n;
	assign adc_sclk = r_adc_sclk;
	assign adc_din = r_adc_din;
	assign adc_data = r_adc_data;
	
	reg r_adc_data_valid_go;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_adc_data_valid_go <= 1'b0;
		else if(r_bit_cnt == 6'd34 && r_div_cnt == SPI_CLK_DR - 1'b1) 
			r_adc_data_valid_go <= 1'b1;
		else
			r_adc_data_valid_go <= 1'b0;
	end
	assign adc_data_convert_valid_go = r_adc_data_valid_go;
	



endmodule


```

ADC控制代码：

```verilog
module adc_ctrl(
	clk,
	rst_n,
	
	adc_addr,
	key_press_valid_go,
	
	adc_cs_n,
	adc_sclk,
	adc_dout,
	adc_din,
	
	adc_data,
	adc_data_valid_go
);


	input 			clk;
	input 			rst_n;
	input 			key_press_valid_go;
	input [2:0]		adc_addr;
	output 			adc_cs_n;
	output 			adc_sclk;
	input  			adc_dout;
	output 			adc_din;
	
	output [11:0] adc_data;
	output 		  adc_data_valid_go;
	
	

	

	
	wire w_adc_convert_en_go;
	wire w_adc_convert_busy;
	wire [11:0] w_adc_data;
	wire w_adc_data_convert_valid_go;
adc128s102_driver adc_driver(
	.clk(clk),
	.rst_n(rst_n),
	.adc_addr(adc_addr),
	.adc_convert_en_go(w_adc_convert_en_go),
	.adc_cs_n(adc_cs_n),
	.adc_sclk(adc_sclk),
	.adc_dout(adc_dout),
	.adc_din(adc_din),
	.adc_convert_busy(w_adc_convert_busy),
	.adc_data(w_adc_data),
	.adc_data_convert_valid_go(w_adc_data_convert_valid_go)
);
	
	

	
	
	reg [1:0]r_adc_convert_busy_sync;
	//wire w_adc_convert_busy_pedge;
	wire w_adc_convert_busy_nedge;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_adc_convert_busy_sync <= 2'd0;
		else
			r_adc_convert_busy_sync <= {r_adc_convert_busy_sync[0],w_adc_convert_busy};
	end

	//assign w_adc_convert_busy_pedge = (r_adc_convert_busy_sync == 2'b01) ? 1'b1 : 1'b0;
	assign w_adc_convert_busy_nedge = (r_adc_convert_busy_sync == 2'b10) ? 1'b1 : 1'b0; 
	
	localparam ADC_COLLECT_TIMES = 100;
	
	
	localparam S_IDLE  	= 4'b0001;
	localparam S_START 	= 4'b0010;
	localparam S_SAMPLE = 4'b0100;
	localparam S_DONE 	= 4'b1000;
	
	
	reg [3:0] r_current_state;
	reg [3:0] r_next_state;
	
	reg [$clog2(ADC_COLLECT_TIMES)-1:0] r_sample_cnt;
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_current_state <= S_IDLE;
		else
			r_current_state <= r_next_state;
	end

	
	always@(*) begin
		case(r_current_state)
			S_IDLE: begin
				if(key_press_valid_go == 1'b1)
					r_next_state = S_START;
				else
					r_next_state = S_IDLE;
			end
			S_START: begin
				if(w_adc_data_convert_valid_go == 1'b1)
					r_next_state = S_SAMPLE;
				else
					r_next_state = S_START;
			end
			S_SAMPLE: begin
				if(r_sample_cnt == ADC_COLLECT_TIMES)
					r_next_state = S_DONE;
				else
					r_next_state = S_SAMPLE;
			end
			S_DONE: begin
				if(w_adc_convert_busy_nedge == 1'b1)
					r_next_state = S_IDLE;
				else
					r_next_state = S_DONE;
			end
		default: r_next_state = S_IDLE;
		endcase
	end
	
	

	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_sample_cnt <= 'd0;
		else if(r_current_state == S_SAMPLE) begin
			if(w_adc_data_convert_valid_go == 1)
				r_sample_cnt <= r_sample_cnt + 1'b1;
			else
				r_sample_cnt <= r_sample_cnt;
		end
		else
			r_sample_cnt <= 'd0;
	end
	
	reg r_adc_convert_en_go;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_adc_convert_en_go <= 1'b0;
		else if(r_current_state == S_IDLE && r_next_state == S_START)
			r_adc_convert_en_go <= 1'b1;
		else if(r_current_state == S_SAMPLE && w_adc_convert_busy_nedge == 1'b1)
			r_adc_convert_en_go <= 1'b1;
		else
			r_adc_convert_en_go <= 1'b0;
	end
	assign w_adc_convert_en_go = r_adc_convert_en_go;
	
	reg r_adc_data_valid_go;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_adc_data_valid_go <= 'd0;
		end
		else if(r_current_state == S_SAMPLE && w_adc_data_convert_valid_go == 1 'b1)
			r_adc_data_valid_go <= 1'b1;
		else
			r_adc_data_valid_go <= 1'b0;
	end
	assign adc_data_valid_go = r_adc_data_valid_go;
	assign adc_data = w_adc_data;

	
endmodule




```

ADC控制代码有几点需要注意：

1. adc128s102在输入地址之后，adc输出的数据并不是当前地址通道的数据，而是在下一次输入地址之后得到想要的数据。所以状态机中START跳转到SAMPLE的条件是第一次转换完成脉冲信号（脉冲信号的宽度必须是一个时钟周期！！！）。那么在SAMPLE状态中就能得到想要的数据。








## 总结



本文后续将AD采集到数据通过uart输出。具体工程为adda，如有需要请至github仓库查看！！！





**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**



