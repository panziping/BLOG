# Intel FPGA ：数码管显示

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

## 正文

### 硬件资源

![Intel FPGA (2)：数码管显示-p1](./图库/Intel FPGA (2)：数码管显示/Intel FPGA (2)：数码管显示-p1.png)

图中HEX_A、HEX_B、HEX_C、HEX_D、HEX_E、HEX_F、HEX_G、HEX_DP控制的是数码管的段码；HEX_SEL0、HEX_SEL1、HEX_SEL2、HEX_SEL3、HEX_SEL4、HEX_SEL5、HEX_SEL6、HEX_SEL7控制的是数码管的位码。

数码管分为共阴极数码管和共阳极数码管，**本篇中使用的是共阳极数码管**。

下表为共阳极数码管译码表。

![Intel FPGA (2)：数码管显示-p2](./图库/Intel FPGA (2)：数码管显示/Intel FPGA (2)：数码管显示-p2.png)

数码管的显示方式有独立显示和扫描显示。**本篇使用的扫描显示**。

扫描显示的原理是由于人眼的余晖效应，上图8个数码管，每个时刻只有一个数码管显示，依次循环，只要扫描的频率足够高（1KHz），在人眼看到的数码管显示就是连续的。

由于直接驱动需要消耗IO的数量为18,太过于占用IO资源，所以数码管显示使用到的驱动芯片是74HC595，这样只需要消耗3个IO，将串行信号转化为并行信号。

#### 74HC595

这里截取了74HC595的部分数据手册，读者自行阅读。

![Intel FPGA (2)：数码管显示-p3](./图库/Intel FPGA (2)：数码管显示/Intel FPGA (2)：数码管显示-p3.png)

![Intel FPGA (2)：数码管显示-p4](./图库/Intel FPGA (2)：数码管显示/Intel FPGA (2)：数码管显示-p4.png)

![Intel FPGA (2)：数码管显示-p5](./图库/Intel FPGA (2)：数码管显示/Intel FPGA (2)：数码管显示-p5.png)





### 程序编写

根据74HC595数据手册编写驱动代码。

由上文74HC595数据手册可以知道SH_CP（SCLK）、ST_CP（RCLK）和DS（DIO）的时序关系。

![Intel FPGA (2)：数码管显示-p6](./图库/Intel FPGA (2)：数码管显示/Intel FPGA (2)：数码管显示-p6.png)

74HC595驱动程序：

```verilog
module hc595_driver(
	clk,
	rst_n,
	seg_data,
	seg_data_valid_go,
	seg_sclk,
	seg_rclk,
	seg_dio
);

	input clk;
	input rst_n;
	input [15:0] seg_data;
	input seg_data_valid_go;
	output reg seg_sclk;
	output reg seg_rclk;
	output reg seg_dio;
	

	reg [15:0] r_seg_data;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_seg_data <= 'd0;
		else if(seg_data_valid_go == 1'b1)
			r_seg_data <= seg_data;
		else
			r_seg_data <= r_seg_data;
	end
	
	localparam DIV_CNT_MAX = 4;			//fseg_sclk = 6.25MHz
	reg [2:0] r_div_cnt;
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_div_cnt <= 'd0;
		else if(r_div_cnt == DIV_CNT_MAX -1)
			r_div_cnt <= 'd0;
		else	
			r_div_cnt <= r_div_cnt + 1'b1;
	end
	wire w_sclk_pluse;	//SH_CP
	assign w_sclk_pluse = (r_div_cnt == DIV_CNT_MAX -1) ? 1'b1 :1'b0;
	
	
	reg [4:0] r_sclk_edge_cnt;	//SH_CP
	
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_sclk_edge_cnt <= 'd0;
		else if(w_sclk_pluse == 1'b1)
			if(r_sclk_edge_cnt == 5'd31)
				r_sclk_edge_cnt <= 'd0;
			else
				r_sclk_edge_cnt <= r_sclk_edge_cnt + 1'd1;
		else
			r_sclk_edge_cnt <= r_sclk_edge_cnt;
	end
	

	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			seg_sclk <= 1'd0;
			seg_rclk <= 1'd0;
			seg_dio <= 1'd0;
		end
		else begin
			case(r_sclk_edge_cnt)
				5'd0 : begin seg_sclk = 1'b0; seg_rclk = 1'b1; seg_dio = r_seg_data[15]; end //Q2H(HEX_DP)
				5'd1 : begin seg_sclk = 1'b1; seg_rclk = 1'b0;  								 end 
				5'd2 : begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[14]; end //Q2G(HEX_G)
				5'd3 : begin seg_sclk = 1'b1;								   						 end
				5'd4 : begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[13]; end //Q2F(HEX_F)
				5'd5 : begin seg_sclk = 1'b1;														    end
				5'd6 : begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[12]; end //Q2E(HEX_E)
				5'd7 : begin seg_sclk = 1'b1;														    end
				5'd8 : begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[11]; end //Q2D(HEX_D)	
				5'd9 : begin seg_sclk = 1'b1;														    end
				5'd10: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[10]; end //Q2C(HEX_C)	
				5'd11: begin seg_sclk = 1'b1;														    end
				5'd12: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[9];  end //Q2B(HEX_B)	
				5'd13: begin seg_sclk = 1'b1;					    									 end
				5'd14: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[8];  end //Q2A(HEX_A)
				5'd15: begin seg_sclk = 1'b1;					    									 end
				5'd16: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[7];  end //Q1H(HEX_SEL7)		
				5'd17: begin seg_sclk = 1'b1;					    									 end
				5'd18: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[6];  end //Q1G(HEX_SEL6)
				5'd19: begin seg_sclk = 1'b1;					    									 end
				5'd20: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[5];  end //Q1F(HEX_SEL5)
				5'd21: begin seg_sclk = 1'b1;					    									 end
				5'd22: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[4];  end //Q1E(HEX_SEL4)		
				5'd23: begin seg_sclk = 1'b1;					    									 end
				5'd24: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[3];  end //Q1D(HEX_SEL3)			
				5'd25: begin seg_sclk = 1'b1;					    									 end
				5'd26: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[2];  end //Q1C(HEX_SEL2)	
				5'd27: begin seg_sclk = 1'b1;					    									 end
				5'd28: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[1];  end //Q1B(HEX_SEL1)	
				5'd29: begin seg_sclk = 1'b1;					    									 end
				5'd30: begin seg_sclk = 1'b0;				        seg_dio = r_seg_data[0];  end //Q1A(HEX_SEL0)
				5'd31: begin seg_sclk = 1'b1;					    									 end
				default:;
			endcase
		end
	end

endmodule 

```

8位数码管显示程序：

显示内容（32位数据，每个数码管占4位）转化为数码管显示编码（16位数据，依次是1位dp位，7位段码，8位位码）。

```verilog
module seg_disp(
	clk,
	rst_n,
	disp_en,
	disp_data,
	disp_data_valid_go,
	seg_data,
	seg_data_valid_go
);
	input clk;
	input rst_n;
	input disp_en;
	input [31:0] disp_data;
	input disp_data_valid_go;
	output reg [14:0] seg_data;
	output reg seg_data_valid_go;
	
	
	reg [31:0] r_disp_data;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_disp_data <= 'd0;
		else if(disp_data_valid_go == 1'b1) 
			r_disp_data <= disp_data;
		else
			r_disp_data <= r_disp_data;
	end
    
    
    
	//segment refresh frequency: 1KHz
	localparam REFRESH_CNT_MAX = 50_000_000 / 1000;
	
	reg [$clog2(REFRESH_CNT_MAX)-1:0] r_refresh_cnt;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_refresh_cnt <= 'd0;
		else if(disp_en == 1'b1) 
			if(r_refresh_cnt == REFRESH_CNT_MAX - 1)
				r_refresh_cnt <= 'd0;
			else
				r_refresh_cnt <= r_refresh_cnt + 1'd1;
		else
			r_refresh_cnt <= 'd0;
	end
	wire w_refresh_pluse;
	assign w_refresh_pluse = (r_refresh_cnt == REFRESH_CNT_MAX - 1) ? 1'b1:1'b0;
	
	reg [7:0] r_seg_sel;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_seg_sel <= 8'b0000_0001;
		else if(w_refresh_pluse == 1'b1)
			r_seg_sel <= {r_seg_sel[6:0],r_seg_sel[7]};
		else
			r_seg_sel <= r_seg_sel;
	end
	
	
	reg [3:0] r_seg_disp;
	always@(*) begin
		case(r_seg_sel)
		8'b0000_0001 : r_seg_disp = r_disp_data[3:0];
		8'b0000_0010 : r_seg_disp = r_disp_data[7:4];
		8'b0000_0100 : r_seg_disp = r_disp_data[11:8];
		8'b0000_1000 : r_seg_disp = r_disp_data[15:12];
		8'b0001_0000 : r_seg_disp = r_disp_data[19:16];
		8'b0010_0000 : r_seg_disp = r_disp_data[23:20];
		8'b0100_0000 : r_seg_disp = r_disp_data[27:24];
		8'b1000_0000 : r_seg_disp = r_disp_data[31:28];		
		default:r_seg_disp = 4'b0000;
		endcase
	end
	
	reg [6:0] r_seg_code;
	always@(*) begin
		case(r_seg_disp)
			4'h0: r_seg_code = 7'b1000000;
			4'h1: r_seg_code = 7'b1111001;
			4'h2: r_seg_code = 7'b0100100;
			4'h3: r_seg_code = 7'b0110000;
			4'h4: r_seg_code = 7'b0011001;
			4'h5: r_seg_code = 7'b0010010;
			4'h6: r_seg_code = 7'b0000010;
			4'h7: r_seg_code = 7'b1111000;
			4'h8: r_seg_code = 7'b0000000;
			4'h9: r_seg_code = 7'b0010000;
			4'ha: r_seg_code = 7'b0001000;
			4'hb: r_seg_code = 7'b0000011;
			4'hc: r_seg_code = 7'b1000110;
			4'hd: r_seg_code = 7'b0100001;
			4'he: r_seg_code = 7'b0000110;
			4'hf: r_seg_code = 7'b0001110;
			default:;
		endcase
	end
	
	
	
	reg [1:0] r_refresh_pluse_sync;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_refresh_pluse_sync <= 'd0;
		else 
			r_refresh_pluse_sync <= {r_refresh_pluse_sync[0],w_refresh_pluse};
	end
	
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			seg_data <= 'd0;
			seg_data_valid_go <= 1'd0;
		end
		else if(r_refresh_pluse_sync[1] == 1'b1) begin
			seg_data <= disp_en ? {r_seg_code,r_seg_sel} : 15'd0;
			seg_data_valid_go <= disp_en ? 1'b1 : 1'b0;
		end	
		else begin
			seg_data <= seg_data;
			seg_data_valid_go <= seg_data_valid_go;
		end
	end

	
endmodule

```

用于测试的顶层文件：

```verilog
`timescale 1ns/1ns
module segment(
	i_clk,
	i_rst_n,
	o_seg_sclk,
	o_seg_rclk,
	o_seg_dio
);


	input i_clk;
	input i_rst_n;
	output o_seg_sclk;
	output o_seg_rclk;
	output o_seg_dio;
	


	
	wire  [31:0] disp_data;
	assign disp_data = 32'habcdef12;
	
	wire [15:0] seg_data;
	wire seg_data_valid_go;
	
seg_disp seg_disp(
	.clk(i_clk),
	.rst_n(i_rst_n),
	.disp_en(1'b1),
	.disp_data(disp_data),
	.disp_data_valid_go(1'b1),
	.seg_data(seg_data[14:0]),
	.seg_data_valid_go(seg_data_valid_go)
);
	
	



hc595_driver hc595_driver(
	.clk(i_clk),
	.rst_n(i_rst_n), 
	.seg_data({1'b1,seg_data[14:0]}),		//{1bit dp,7bit seg_code, 8bit seg_sel}
	.seg_data_valid_go(seg_data_valid_go),
	.seg_sclk(o_seg_sclk),
	.seg_rclk(o_seg_rclk),
	.seg_dio(o_seg_dio)
);




endmodule

```









## 总结

本工程名为segment，如有需要请至github仓库查看！！！



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**



