# Intel FPGA：dac tlv5618a

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

**数字模拟转换器**（英语：**Digital to analog converter**，英文缩写：**DAC**）是一种将数字信号转换为模拟信号（以电流、电压或电荷的设备。

电流型DAC和电压型DAC是两种常见的数字模拟转换器类型，它们在设计和应用方面有所不同。以下是比较电流型DAC和电压型DAC的一些因素：

- **输出形式**：电流型DAC以电流形式输出模拟信号，而电压型DAC以电压形式输出模拟信号。这意味着电流型DAC的输出是通过传递电流来实现的，而电压型DAC的输出是通过产生电压来实现的。
- **负载匹配**：电流型DAC通常具有较低的输出阻抗，这使得它们对负载变化更具有稳定性。相比之下，电压型DAC的输出阻抗较高，需要进行额外的负载匹配以确保输出电压的稳定性。
- **功耗**：电流型DAC通常具有较低的功耗，因为它们不需要经过额外的缓冲放大器来驱动负载。电压型DAC则可能需要额外的缓冲放大器来提供足够的电流驱动能力，从而增加功耗。
- **动态范围**：在一些应用中，电流型DAC具有更广泛的动态范围，可以提供更高的分辨率和更精确的模拟输出。电压型DAC的动态范围可能受限于电源供应和输出缓冲电路的限制。

综上所述，选择电流型DAC还是电压型DAC取决于具体的应用需求。电流型DAC通常适用于对输出负载变化敏感、功耗要求较低且需要较高动态范围的应用。而电压型DAC则适用于对输出电压稳定性要求较高、对负载匹配较为灵活的应用。



本篇采用的DAC芯片是TLV5618A。这是一款双通道 ，12bit的电压输出型DAC。

### 硬件电路

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p1.png)

### TLV5618A

这里截取了74HC595的部分数据手册，读者自行阅读。

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p2.png)

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p3.png)

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p4.png)

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p5.png)

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p6.png)

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p7.png)

### 波形图

![](./图库/Intel FPGA (5)：dac tlv5618a/Intel FPGA (5)：dac tlv5618a-p8.png)

### 代码展示

```verilog
module tlv5618_driver(
	clk,
	rst_n,
	dac_data,
	dac_load_en_go,
	
	cs_n,
	sclk,
	din,
	dac_convert_busy
);

	input 		 clk;
	input 		 rst_n;
	input [15:0] dac_data;
	input 		 dac_load_en_go;
	output 		 cs_n;
	output 		 sclk;
	output 		 din;
	output 		 dac_convert_busy;
	
	
	reg [15:0] r_dac_data;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_dac_data <= 16'd0;
		else if(dac_load_en_go == 1'b1)
			r_dac_data <= dac_data;
		else
			r_dac_data <= r_dac_data;
	end
	
	
	localparam SPI_CLK = 12_500_000;
	localparam SYS_FREQ = 50_000_000;
	localparam SPI_CLK_DR =  SYS_FREQ / SPI_CLK;	//freq = 12.5Mhz,Fmax = 20Mhz
	
	reg r_dac_convert_en;
	wire w_dac_convert_end;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_dac_convert_en <= 1'b0;
		else if(dac_load_en_go == 1'b1)
			r_dac_convert_en <= 1'b1;
		else if(w_dac_convert_end == 1'b1)
			r_dac_convert_en <= 1'b0;
		else
			r_dac_convert_en <= r_dac_convert_en;
	end
	assign dac_convert_busy = ~r_dac_convert_en;

	reg [$clog2(SPI_CLK_DR)-1:0]r_sclk_cnt;
	wire w_sclk_pluse;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_sclk_cnt <= 'd0;
		else if(r_dac_convert_en == 1'b1) begin
			if(r_sclk_cnt == SPI_CLK_DR - 1'd1)
				r_sclk_cnt <= 'd0;
			else	
				r_sclk_cnt <= r_sclk_cnt + 1'd1;
		end
		else
			r_sclk_cnt <= 'd0;
	end
	assign w_sclk_pluse = (r_sclk_cnt == 'd1) ? 1'b1 : 1'b0;
	
	reg [5:0] r_bit_cnt;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_bit_cnt <= 'd0;
		else if(r_dac_convert_en == 1'b1) begin
			if(w_sclk_pluse == 1'b1)
				r_bit_cnt <= r_bit_cnt + 1'b1;
			else 
				r_bit_cnt <= r_bit_cnt;
		end
		else
			r_bit_cnt <= 'd0;
	end
	assign w_dac_convert_end = (r_bit_cnt == 6'd35) ? 1'b1 : 1'b0;
	
	
	reg r_sclk;
	reg r_cs_n;
	reg r_din;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_cs_n <= 1'b1;
			r_din <= 1'b0;
			r_sclk <= 1'b0;
		end
		else begin
			case(r_bit_cnt)
			6'd0 : begin r_cs_n <= 1'b1; r_din <= 1'b0; r_sclk <= 1'b0; end
			6'd1 : begin r_cs_n <= 1'b0; r_din <= 1'b0; r_sclk <= 1'b0; end
			6'd2 : begin r_din <= r_dac_data[15]; r_sclk <= 1'b1; end
			6'd3 : begin r_sclk <= 1'b0; end
			6'd4 : begin r_din <= r_dac_data[14]; r_sclk <= 1'b1; end
			6'd5 : begin r_sclk <= 1'b0; end
			6'd6 : begin r_din <= r_dac_data[13]; r_sclk <= 1'b1; end
			6'd7 : begin r_sclk <= 1'b0; end	
			6'd8 : begin r_din <= r_dac_data[12]; r_sclk <= 1'b1; end
			6'd9 : begin r_sclk <= 1'b0; end		
			6'd10 : begin r_din <= r_dac_data[11]; r_sclk <= 1'b1; end
			6'd11 : begin r_sclk <= 1'b0; end			
			6'd12 : begin r_din <= r_dac_data[10]; r_sclk <= 1'b1; end
			6'd13 : begin r_sclk <= 1'b0; end				
			6'd14 : begin r_din <= r_dac_data[9]; r_sclk <= 1'b1; end
			6'd15 : begin r_sclk <= 1'b0; end		
			6'd16 : begin r_din <= r_dac_data[8]; r_sclk <= 1'b1; end
			6'd17 : begin r_sclk <= 1'b0; end		
			6'd18 : begin r_din <= r_dac_data[7]; r_sclk <= 1'b1; end
			6'd19 : begin r_sclk <= 1'b0; end		
			6'd20 : begin r_din <= r_dac_data[6]; r_sclk <= 1'b1; end
			6'd21 : begin r_sclk <= 1'b0; end		
			6'd22 : begin r_din <= r_dac_data[5]; r_sclk <= 1'b1; end
			6'd23 : begin r_sclk <= 1'b0; end		
			6'd24 : begin r_din <= r_dac_data[4]; r_sclk <= 1'b1; end
			6'd25 : begin r_sclk <= 1'b0; end		
			6'd26 : begin r_din <= r_dac_data[3]; r_sclk <= 1'b1; end
			6'd27 : begin r_sclk <= 1'b0; end
			6'd28 : begin r_din <= r_dac_data[2]; r_sclk <= 1'b1; end
			6'd29 : begin r_sclk <= 1'b0; end	
			6'd30 : begin r_din <= r_dac_data[1]; r_sclk <= 1'b1; end
			6'd31 : begin r_sclk <= 1'b0; end		
			6'd32 : begin r_din <= r_dac_data[0]; r_sclk <= 1'b1; end
			6'd33 : begin r_sclk <= 1'b0; end			
			6'd34 : begin r_cs_n <= 1'b0; r_din <= 1'b0; r_sclk <= 1'b1; end  //notes:the next positive clock edge following the 16th falling clock edge.
			6'd35 : begin r_cs_n <= 1'b1; r_din <= 1'b0; r_sclk <= 1'b0; end
			default:begin r_cs_n <= 1'b1; r_din <= 1'b0; r_sclk <= 1'b0; end
			endcase
		end
	
	end
	assign sclk = r_sclk;
	assign cs_n = r_cs_n;
	assign din = r_din;
	

	
endmodule

```

TLV5618驱动代码有几点需要注意：

1. 由硬件电路可知TLV5618的参考电压为**2.048V**，根据DAC输出公式可知$2REF\frac{CODE}{2^{n}}$V,需要注意TLV5618在输出端接了一个放大倍数两倍的放大器;CODE的范围是0 ~ $(2^{n}-1)$,n=12，所以CODE的范围为0~4095。
   $$
   V_{out} = 2 * 2.048 *\frac{CODE}{2^{12}}
   $$

2. 本设计中SCLK的频率是12.5MHz,那么可以得到本设计中$t_{su(CS-CK)}$=80ns,$t_{su(C16-CS)}$=80ns。如果需要设计数据连续发送时，本设计一次发送周期需要1400ns，此时需要注意与$t_{s(FS)}$的值进行比较，要不然会导致精度下降，**所以需要延迟一段时间用来满足设计需求**。

3. **SCLK需要注意第16个下降沿之后还需要在产生一次上升沿！！！**，这样数据才能送到保持寄存器或者控制寄存器。







## 总结

本工程名为adda，如有需要请至github仓库查看！！！







---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**



