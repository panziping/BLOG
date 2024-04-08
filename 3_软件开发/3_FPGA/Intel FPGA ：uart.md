# Intel FPGA：uart

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

### 物理层

![Intel FPGA (4)：uart-p1](./图库/Intel FPGA (4)：uart/Intel FPGA (4)：uart-p1.png)

### 电平标准

串口通讯分为TTL标准和RS-232标准。

| 通讯标准  |               电平标准               |
| :-------: | :----------------------------------: |
| 5V    TTL |  逻辑1：2.4V ~ 5V；逻辑0：0 ~ 0.5V   |
|  RS-232   | 逻辑1：-15V ~ -3V；逻辑0：+3V ~ +15V |

### 接口标准（DB9）

| 序号 |        名称         | 符号 | 数据方向 |                             说明                             |
| :--: | :-----------------: | :--: | :------: | :----------------------------------------------------------: |
|  1   |      载波检测       | DCD  | DTE→DCE  | Data Carrier Detect，数据载波检测，用于DTE告知对方，本机是否收到对方的载波信号 |
|  2   |      接收数据       | RXD  | DTE←DCE  |                  Receive Data，数据接收信号                  |
|  3   |      发送数据       | TXD  | DTE→DCE  |                 Transmit Data，数据发送信号                  |
|  4   | 数据终端（DTE）就绪 | DTR  | DTE→DCE  | Data Terminal Ready，数据终端就绪，用于DTE向对方告知本机是否已准备好 |
|  5   |       信号地        | GND  |    -     |                            共地线                            |
|  6   | 数据设备（DCE）就绪 | DSR  | DTE←DCE  | Data Set Ready，数据发送就绪，用于DCE告知对方本机是否处于待命状态 |
|  7   |      请求发送       | RTS  | DTE→DCE  |  Request To Send，请求发送，DTE请求DCE本设备向DCE端发送数据  |
|  8   |      允许发送       | CTS  | DTE←DCE  | Clear To Send，允许发送，DCE回应对方的RTS发送请求，告知对方是否可以发送数据 |
|  9   |      响铃指示       |  RI  | DTE←DCE  |       Ring Indicator，响铃指示，表示DCE端与线路已接通        |

> **实际使用中，受限于设备IO较少，成本等因素，只保留了RXD,TXD,GND三条信号线。**

### 串口协议

![Intel FPGA (4)：uart-p2](./图库/Intel FPGA (4)：uart/Intel FPGA (4)：uart-p2.png)

> 需要注意的上图采用的是正逻辑进行数据通信，同样有采用负逻辑进行数据通信。
>
> 停止位可以是1bit、1.5bits、2bits。
>
> 数据位可以是5bits、6bits、7bits、8bits。
>
> 校验位分为无校验、奇校验和偶校验。

verilog代码中数据的校验使用的是异或运算。

```verilog
module check_parity(
    data,
	 parity_odd,
	 parity_even
);

	input [7:0] data;
	output parity_odd;
	output parity_even;
		
	assign parity_odd = ~^data;
	assign parity_even = ^data;

endmodule 

```

> 注意上述数据使用的是正逻辑。

### 串口接收

#### 波形图

![Intel FPGA (4)：uart-p3](./图库/Intel FPGA (4)：uart/Intel FPGA (4)：uart-p3.png)

检测接收总线下降沿（起始位），使能串口接收功能，在数据位中间时刻采样，数据更加稳定。

#### 串口接收程序：

```verilog
module uart_rxd(
	clk,
	rst_n,
	rxd,
	parity,
	rxd_data,
	rxd_data_valid_go
); 
	input clk;
	input rst_n;
	input rxd;
	input [1:0] parity;
	output [7:0] rxd_data;
	output rxd_data_valid_go;
	
	
	localparam BAUD = 115200;
	localparam SYS_FREQ = 50_000_000;
	localparam BAUD_DR = SYS_FREQ / BAUD;
	
	
	localparam P_EVEN = 2'b00;
	localparam P_ODD  = 2'b01;
	localparam P_NONE = 2'b10;
	
	reg [3:0] r_bit_width;
	always@(*) begin
		case(parity)
		P_EVEN : r_bit_width = 4'd10;
		P_ODD  : r_bit_width = 4'd10;
		P_NONE : r_bit_width = 4'd9;
		default :r_bit_width = 4'd9;
		endcase
	end
	
	reg [2:0] r_rxd_sync;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_rxd_sync <= 'd0;
		else
			r_rxd_sync <= {r_rxd_sync[1:0],rxd};
	end
	
	wire w_rxd_nedge;
	assign w_rxd_nedge = (r_rxd_sync[2:1] == 2'b10) ? 1'b1:1'b0;
	

	reg r_rxd_en;
	reg [3:0] r_bit_cnt;
	wire w_rxd_end;
	reg r_rxd_data_error;
	reg [$clog2(BAUD_DR)-1:0] r_baud_cnt;
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_rxd_en <= 1'b0;
		else if(w_rxd_nedge == 1'b1)
			r_rxd_en <= 1'b1;
		else if(r_rxd_data_error == 1'b1 || w_rxd_end == 1'b1)
			r_rxd_en <= 1'b0;
		else
			r_rxd_en <= r_rxd_en;
	end
		
	assign w_rxd_end = (r_baud_cnt == BAUD_DR >> 1) && (r_bit_cnt == r_bit_width);
	
	

	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_baud_cnt <= 'd0;
		else if(r_rxd_en == 1'b1)
			if(r_baud_cnt == BAUD_DR - 1'd1)
				r_baud_cnt <= 'd0;
			else 
				r_baud_cnt <= r_baud_cnt + 1'd1;
		else
			r_baud_cnt <= 'd0;
	end
	wire w_bps_clk;
	assign w_bps_clk = (r_baud_cnt == BAUD_DR >> 1 ) ? 1'b1:1'b0;
	

 	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_bit_cnt <= 'd0;
		else if(r_rxd_en == 1'b1)
			if(r_baud_cnt == BAUD_DR - 1'd1)
				r_bit_cnt <= r_bit_cnt + 1'b1;
			else
				r_bit_cnt <= r_bit_cnt;
		else
			r_bit_cnt <= 'd0;
	end
	
	

	reg [7:0] r_rxd_data;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_rxd_data_error <= 1'd0;
			r_rxd_data <= 8'd0;
		end
		else if(w_bps_clk == 1'b1) begin
			case(r_bit_cnt)
			4'd0: begin r_rxd_data_error <= (r_rxd_sync[2] == 1'b1) ? 1'b1 : 1'b0; end
			4'd1: begin r_rxd_data[0] <= r_rxd_sync[2];end
			4'd2: begin r_rxd_data[1] <= r_rxd_sync[2];end
			4'd3: begin r_rxd_data[2] <= r_rxd_sync[2];end
			4'd4: begin r_rxd_data[3] <= r_rxd_sync[2];end
			4'd5: begin r_rxd_data[4] <= r_rxd_sync[2];end
			4'd6: begin r_rxd_data[5] <= r_rxd_sync[2];end
			4'd7: begin r_rxd_data[6] <= r_rxd_sync[2];end	
			4'd8: begin r_rxd_data[7] <= r_rxd_sync[2];end
			4'd9: begin
				case(parity)
				P_EVEN : r_rxd_data_error <= (^r_rxd_data^r_rxd_sync[2] == 1'b0) ? 1'b0 : 1'b1;
				P_ODD  : r_rxd_data_error <= (^r_rxd_data^r_rxd_sync[2] == 1'b1) ? 1'b0 : 1'b1;
				P_NONE : r_rxd_data_error <=  (r_rxd_sync[2] == 1'b0) ? 1'b1 : 1'b0;
				default:r_rxd_data_error <= 1'd1;
				endcase
			end	
			4'd10: r_rxd_data_error <= (r_rxd_sync[2] == 1'b0) ? 1'b1 : 1'b0;
			default:;
			endcase
		end //else if
		else begin 
			r_rxd_data_error <= 1'd0;
			r_rxd_data <= r_rxd_data;
		end	
	end
	
	

	
	reg r_rxd_data_valid_go;
	always@(posedge clk or negedge rst_n) begin
			if(!rst_n)
				r_rxd_data_valid_go <= 1'b0;
			else if((r_baud_cnt == BAUD_DR >> 1) && (r_bit_cnt == r_bit_width))
				r_rxd_data_valid_go <= 1'b1;
			else
				r_rxd_data_valid_go <= 1'b0;
	end
	assign rxd_data = r_rxd_data;
	assign rxd_data_valid_go = r_rxd_data_valid_go;
	
	
endmodule


```

### 串口发送

#### 波形图

![Intel FPGA (4)：uart-p4](./图库/Intel FPGA (4)：uart/Intel FPGA (4)：uart-p4.png)

 当发送使能脉冲信号到来时，启动发送模块。**需要注意此图中r_bit_cnt,应该到12位。**

#### 串口发送程序：

```verilog
module uart_txd(
	clk,
	rst_n,
	txd_data,
	txd_en_go,
	parity,
	txd,
	txd_busy
);
	input 		clk;
	input 		rst_n;
	input [7:0] txd_data;
	input 		txd_en_go;
	input [1:0] parity;
	output 		txd;
	output 		txd_busy;
	
	localparam BAUD = 115200;
	localparam SYS_FREQ = 50_000_000;
	localparam BAUD_DR = SYS_FREQ / BAUD;	
	
	localparam P_EVEN = 2'b00;
	localparam P_ODD  = 2'b01;
	localparam P_NONE = 2'b10;
	
	
	reg [3:0] r_bit_width;
	always@(*) begin
		case(parity)
		P_EVEN : r_bit_width = 4'd12;
		P_ODD  : r_bit_width = 4'd12;
		P_NONE : r_bit_width = 4'd11;
		default :r_bit_width = 4'd11;
		endcase
	end
	
	
	reg [7:0] r_txd_data;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_txd_data <= 8'd0;
		else if(txd_en_go == 1'b1)
			r_txd_data <= txd_data;
		else 
			r_txd_data <= r_txd_data;
	end
	
	
	reg r_txd_en;
	reg [3:0] r_bit_cnt;
	wire w_txd_end;
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_txd_en <= 1'b0;
		else if(txd_en_go == 1'b1)
			r_txd_en <= 1'b1;
		else if(w_txd_end == 1'b1)
			r_txd_en <= 1'b0;
		else
			r_txd_en <= r_txd_en;
	end
	
	assign txd_busy = r_txd_en;
	assign w_txd_end = (r_bit_cnt == r_bit_width) ? 1'b1 : 1'b0;
	

	reg [$clog2(BAUD_DR)-1:0] r_baud_cnt;
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_baud_cnt <= 'd0;
		else if(r_txd_en == 1'b1) begin
			if(r_baud_cnt == BAUD_DR - 1)
				r_baud_cnt <= 'd0;
			else
				r_baud_cnt <= r_baud_cnt + 1'b1;
		end
		else
			r_baud_cnt <= 'd0;
	end
	wire w_bps_clk;
	assign w_bps_clk = (r_baud_cnt == 1'b1) ? 1'b1:1'b0;
	

	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_bit_cnt <= 4'd0;
		else if(r_txd_en == 1'b1) begin
			if(w_bps_clk == 1'b1)
				r_bit_cnt <= r_bit_cnt + 1'b1;
			else
				r_bit_cnt <= r_bit_cnt;
		end
		else
			r_bit_cnt <= 4'd0;
	end
	
	
	reg r_txd;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_txd <= 1'b1;
		end
		else begin
			case(r_bit_cnt)
				0: begin r_txd <= 1'b1; end
				1: begin r_txd <= 1'b0; end
				2: begin r_txd <= r_txd_data[0]; end
				3: begin r_txd <= r_txd_data[1]; end
				4: begin r_txd <= r_txd_data[2]; end
				5: begin r_txd <= r_txd_data[3]; end
				6: begin r_txd <= r_txd_data[4]; end
				7: begin r_txd <= r_txd_data[5]; end
				8: begin r_txd <= r_txd_data[6]; end
				9: begin r_txd <= r_txd_data[7]; end
				10: begin
					case(parity)
						P_EVEN : r_txd <= ^r_txd_data;
						P_ODD  : r_txd <= ~^r_txd_data;
						P_NONE : r_txd <= 1'b1;
						default: r_txd <= 1'd1;
					endcase
				end
				11: begin r_txd <= 1'd1; end
				12: begin r_txd <= 1'd1; end
				default:begin r_txd <= 1'd1; end
			endcase
		end
	end
	
	assign txd = r_txd;
	
	
endmodule

```

### 顶层测试文件

```verilog
module uart_loop(
	i_clk,
	i_rst_n,
	i_rx,
	o_tx
);

	input i_clk;
	input i_rst_n;
	input i_rx;
	output o_tx;
	

	
	localparam P_EVEN = 2'b00;
	localparam P_ODD  = 2'b01;
	localparam P_NONE = 2'b10;
	
	
	wire [7:0] w_uart_data;
	wire w_uart_data_valid_go;

uart_rxd uart_rxd(
	.clk(i_clk),
	.rst_n(i_rst_n),
	.rxd(i_rx),
	.parity(P_NONE),
	.rxd_data(w_uart_data),
	.rxd_data_valid_go(w_uart_data_valid_go)
); 
	
uart_txd uart_txd(
	.clk(i_clk),
	.rst_n(i_rst_n),
	.txd_data(w_uart_data),
	.txd_en_go(w_uart_data_valid_go),
	.parity(P_NONE),
	.txd(o_tx),
	.txd_busy()
);
	
endmodule 



```

本工程名为uart，如有需要请至github仓库查看！！！





## 总结

工程中很少能够应用到串口回环测试程序。

一般数据流的处理过程应该是：

1. 串口接收
2. 数据存储（RAM或者FIFO）
3. 协议解析
4. 数据处理
5. 串口发送



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**



