# Intel FPGA：线性序列机

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

### 点亮LED灯

#### 硬件资源

![Intel FPGA (1)：线性序列机-p1](./图库/Intel FPGA (1)：线性序列机/Intel FPGA (1)：线性序列机-p1.png)

由原理图可知，FPGA的IO口输出低电平，则LED点亮。



#### 程序编写

```verilog
module led_test(
	led
);
	output led;

	assign led = 1'b0;

endmodule 
```



### 点亮LED灯进阶

 将LED点亮200ms，熄灭800ms。

#### 程序编写

```verilog
module led_test(
	clk,
	rst_n,
	led
);

	input clk;
	input rst_n;
	output reg led;
	reg [27:0] r_led_cnt;
	localparam LED_CNT_MAX = 28'd50_000_000;
	localparam LED_CNT_TURN = 28'd40_000_000;

    always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_led_cnt <= 'd0;
		else if(r_led_cnt == LED_CNT_MAX-1)
			r_led_cnt <= 'd0;
		else
			r_led_cnt <= r_led_cnt + 1'd1;
	end
	
    always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			led <= 1'd1;
		else if(r_led_cnt == LED_CNT_TURN-1)
			led <= 1'd0;
		else if(r_led_cnt == LED_CNT_MAX-1)
			led <= 1'd1;
		else
			led <= led;
	end

	
endmodule 
```

#### 波形图

![Intel FPGA (1)：线性序列机-p2](./图库/Intel FPGA (1)：线性序列机/Intel FPGA (1)：线性序列机-p2.png)

由上述实验可以发现，通过计数器可以产生一个占空比不是50%的周期信号。那么是不是由此可以引申，通过计数器对时钟计数，产生一串带有数字信息的信号。

### 线性序列机（LSM）

产生一段信号，包含的内容为11011010，每个码元所占用的时间为50us。

```verilog
module tx_test(
	clk,
	rst_n,
	tx
);
	input clk;
	input rst_n;
	output reg tx;

	reg [15:0] r_tim_cnt;
	
	localparam TIM_CNT_MAX = 16'd20_000;
	localparam DATA0 = 16'd2_500;
	localparam DATA1 = 16'd5_000;
	localparam DATA2 = 16'd7_500;
	localparam DATA3 = 16'd10_000;
	localparam DATA4 = 16'd12_500;
	localparam DATA5 = 16'd15_000;
	localparam DATA6 = 16'd17_500;
	localparam DATA7 = 16'd20_000;
    always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_tim_cnt <= 'd0;
		else if(r_tim_cnt == TIM_CNT_MAX-1)
			r_tim_cnt <= 'd0;
		else
			r_tim_cnt <= r_tim_cnt + 1'd1;
	end
	
    always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			tx <= 1'd1;
		else if(r_tim_cnt == DATA0 - 1'd1)
			tx <= 1'd0;
		else if(r_tim_cnt == DATA1 - 1'd1)
			tx <= 1'd1;
		else if(r_tim_cnt == DATA2 - 1'd1)
			tx <= 1'd0;
		else if(r_tim_cnt == DATA3 - 1'd1)
			tx <= 1'd1;
		else if(r_tim_cnt == DATA4 - 1'd1)
			tx <= 1'd1;
		else if(r_tim_cnt == DATA5 - 1'd1)
			tx <= 1'd0;
		else if(r_tim_cnt == DATA6 - 1'd1)
			tx <= 1'd1;
		else if(r_tim_cnt == DATA7 - 1'd1)
			tx <= 1'd1;
		else 
			tx <= tx;
	end
	
//	always@(posedge clk or negedge rst_n) begin
//		if(!rst_n)
//			tx <= 1'd1;
//		else 
//			case(r_tim_cnt)
//			DATA0-1'd1: tx <= 1'd0;
//			DATA1-1'd1: tx <= 1'd1;			
//			DATA2-1'd1: tx <= 1'd0;
//			DATA3-1'd1: tx <= 1'd1;		
//			DATA4-1'd1: tx <= 1'd1;
//			DATA5-1'd1: tx <= 1'd0;			
//			DATA6-1'd1: tx <= 1'd1;
//			DATA7-1'd1: tx <= 1'd1;			
//			default:tx <= tx;
//			endcase
//	end
//	

endmodule 
```

#### 波形图

![Intel FPGA (1)：线性序列机-p3](./图库/Intel FPGA (1)：线性序列机/Intel FPGA (1)：线性序列机-p3.png)

由上述实验可以发现，通过线性序列机产生了8bits（11011010）的信号。那么是不是对于串行信号都可以通过线性序列机进行输出。



### 数码管驱动

上述是线性序列机（LSM）的简单应用。

这一部分展示线性序列机应用在数码管驱动电路中（完整请见[(Intel FPGA (3)：数码管显示)]()），需要通过线性序列机产生三个信号**seg_sclk（sh_cp）,seg_rclk（st_cp）,seg_dio(ds)**。

波形图

![Intel FPGA (1)：线性序列机-p4](./图库/Intel FPGA (1)：线性序列机/Intel FPGA (1)：线性序列机-p4.png)

#### 程序编写

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
	
	localparam DIV_CNT_MAX = 4;			//fsh_cp = 6.25MHz
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









## 总结

线性序列机可以画出任意波形的数字信号！常用作为最低层RTL设计。



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**



