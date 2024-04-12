# Intel FPGA ：iic主机通信

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

![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P1.png)

- 它是一个支持设备的总线。“总线”指多个设备共用的信号线。在一个I2C 通讯总线中，可连接多个I2C 通讯设备，支持多个通讯主机及多个通讯从机。
- 一个I2C 总线只使用两条总线线路，一条双向串行数据线(SDA) ，一条串行时钟线(SCL)。数据线即用来表示数据，时钟线用于数据收发同步。
- 每个连接到总线的设备都有一个独立的地址，主机可以利用这个地址进行不同设备之间的访问。
- 总线通过上拉电阻接到电源。当I2C 设备空闲时，会输出高阻态，而当所有设备都空闲，都输出高阻态时，由上拉电阻把总线拉成高电平。
- 多个主机同时使用总线时，为了防止数据冲突，会利用仲裁方式决定由哪个设备占用总线。
- 具有三种传输模式：标准模式传输速率为100kbit/s ，快速模式为400kbit/s ，高速模式下可达3.4Mbit/s，但目前大多I2C 设备尚不支持高速模式。
- 连接到相同总线的IC 数量受到总线的最大电容400pF 限制。

### 协议层(一主多从)

理论上IIC是**多主多从**总线，这里为了简化只设计了**一个主机**。

![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P2.png)

- 起始信号和终止信号

  起始信号：当SCL 线是高电平时SDA 线从高电平向低电平切换，这个情况表示通讯的起始。

  终止信号：当SCL 是高电平时SDA 线由低电平向高电平切换，表示通讯的停止。

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P3.png)

- 数据有效性

  传输时，SCL 为高电平的时候SDA 表示的数据有效，即此时的SDA 为高电平时表示数据“1”，为低电平时表示数据“0”；当SCL 为低电平时，SDA的数据无效，一般在这个时候SDA 进行电平切换，为下一次表示数据做好准备。

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P4.png)

- 应答信号

  当数据发出方（不一定是主机还是从机）将8 位数据或命令传出后，会将数据总线（SDA）释放，即设置为输入，然后等待数据接收方将SDA 信号拉低以作为成功接收的应答信号；无论是什么状态，I2C 总线的SCL 信号始终由I2C 主机驱动。

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P5.png)


#### IIC读写操作

写操作：

iic one byte write :

start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + data + (slave ack) + stop;	

iic multiple bytes write :

start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + data + (slave ack) + ... + data + (slave ack) + stop;

读操作：

iic one byte read :

start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + 

start + (device address & 8'b0000_0001) + (slave ack) + data + (master nack) + stop; 

iic multiple bytes read : 

start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + 

start + (device address & 8'b0000_0001) + (slave ack) + data + (master ack) + ... + data + (master nack) + stop;

由于IIC协议中关于传输device address 、reg address和数据的过程类似，

所以将IIC读写操作分别细化为以下几个过程

- Start Sign + Device address + Slave Ack
- Reg address + Slave Ack
- Data + Slave Ack
- Data + Slave Ack +Stop Sign
- Data + Master Ack
- Data + Master Nack +Stop Sign

### 波形图

- 起始信号

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P6.png)

- 写数据

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P7.png)

- 读数据

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P8.png)

  

- 终止信号

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P9.png)

- 主机产生ACK/NACK

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P10.png)

- 从机产生ACK

  ![](./图库/Intel FPGA ：iic主机通信/Intel FPGA ：iic主机通信-P11.png)



### 代码展示

iic控制

```verilog
module iic_ctrl(
	clk,
	rst_n,
	
	iic_device_addr,
	iic_reg_addr,
	
	iic_rd_en_go,
	iic_rd_data,
	
	iic_wr_en_go,
	iic_wr_data,
	iic_rw_data_valid_go,
	iic_busy,
	iic_sclk,
	iic_sdat
);

	input 			clk;
	input 			rst_n;
	
	input  [6:0] 	iic_device_addr;
	input  [7:0] 	iic_reg_addr;
	
	input 			iic_rd_en_go;
	output [7:0] 	iic_rd_data;
	
	input 			iic_wr_en_go;
	input  [7:0] 	iic_wr_data;
	output 			iic_rw_data_valid_go;
	
	output 			iic_busy;
	output 			iic_sclk;
	inout 			iic_sdat;

	// ******************************************************************************************************************************************************************* //
	// write : *write = 0*																																																  //
	// iic one byte write : 																																												           //
	//	start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + data + (slave ack) + stop;																		  //
	// iic multiple bytes write : 																																												  	  //
	//	start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + data + (slave ack) + ... + data + (slave ack) + stop;									  //																																								              //
	// ******************************************************************************************************************************************************************* //
	// read  : *read = 1*																																											  	  				  //
	// iic one byte read : 																																												  	  			  //
	//	start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + 																										  //
	// start + (device address & 8'b0000_0001) + (slave ack) + data + (master nack) + stop; 																										  //
	// iic multiple bytes read : 																																												  	     //
	// start + (device address & 8'b0000_0000) + (slave ack) + reg address + (slave ack) + 																										  //
	// start + (device address & 8'b0000_0001) + (slave ack) + data + (master ack) + ... + data + (master nack) + stop; 																	  //
	// ******************************************************************************************************************************************************************* //

	localparam STA  = 6'b000_001;
	localparam WR   = 6'b000_010; 
	localparam RD   = 6'b000_100;
	localparam ACK  = 6'b001_000;	// master gen ack
	localparam NACK = 6'b010_000;	// master gen nack
	localparam STOP = 6'b100_000;
	
	
	localparam S_IDLE  	    = 7'b000_0001;
	localparam S_WR_REG		 = 7'b000_0010;
	localparam S_WAIT_WR_REG = 7'b000_0100;
	localparam S_WR_REG_DONE = 7'b000_1000;
 	localparam S_RD_REG 		 = 7'b001_0000;
	localparam S_WAIT_RD_REG = 7'b010_0000;
	localparam S_RD_REG_DONE = 7'b100_0000;
	
	reg r_iic_en_go;
	reg [5:0] r_iic_cmd;
	reg [7:0] r_wr_data;
	wire [7:0] w_rd_data;
	wire w_trans_done;
	wire w_slave_ack;
iic_driver iic_driver(
	.clk(clk),
	.rst_n(rst_n),
	.iic_cmd(r_iic_cmd),
	.iic_en_go(r_iic_en_go),
	.rd_data(w_rd_data),
	.wr_data(r_wr_data),
	.trans_done(w_trans_done),
	.slave_ack(w_slave_ack),
	.iic_sclk(iic_sclk),
	.iic_sdat(iic_sdat)
);

	task read_byte;
	input [5:0] ctrl_cmd;
	begin
		r_iic_en_go <= 1'b1;
		r_iic_cmd <= ctrl_cmd;
	end
	endtask
	
	task write_byte;
	input [5:0] ctrl_cmd;
	input [7:0] wr_byte_data;
	begin
		r_iic_en_go <= 1'b1;
		r_iic_cmd <= ctrl_cmd;
		r_wr_data <= wr_byte_data;
	end
	endtask
	
	reg [6:0] r_current_state;
	reg [6:0] r_next_state;
	reg [7:0] r_cnt; 
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_current_state <= S_IDLE;
		else
			r_current_state <= r_next_state;
	end
	
	always@(*) begin
		case(r_current_state)
			S_IDLE: begin
				if(iic_wr_en_go == 1'b1)
					r_next_state = S_WR_REG;
				else if(iic_rd_en_go == 1'b1)
					r_next_state = S_RD_REG;
				else
					r_next_state = S_IDLE;
			end
			S_WR_REG: begin
				r_next_state = S_WAIT_WR_REG;
			end
			S_WAIT_WR_REG: begin
				if(w_trans_done == 1'b1) begin
					case(r_cnt)
						8'd0: r_next_state = (w_slave_ack == 1'd0) ? S_WR_REG : S_IDLE; // device addr
						8'd1: r_next_state = (w_slave_ack == 1'd0) ? S_WR_REG : S_IDLE; // reg addr
						8'd2: r_next_state = (w_slave_ack == 1'd0) ? S_WR_REG_DONE : S_IDLE; //wr data
						default: r_next_state = S_IDLE;
					endcase
				end
				else
					r_next_state = S_WAIT_WR_REG;
			end
			S_WR_REG_DONE: begin
				r_next_state = S_IDLE;
			end
			
			S_RD_REG: begin
				r_next_state = S_WAIT_RD_REG;
			end
			S_WAIT_RD_REG: begin
				if(w_trans_done == 1'b1) begin
					case(r_cnt)
						8'd0: r_next_state = (w_slave_ack == 1'd0) ? S_RD_REG : S_IDLE; // device addr
						8'd1: r_next_state = (w_slave_ack == 1'd0) ? S_RD_REG : S_IDLE; // reg addr
						8'd2: r_next_state = (w_slave_ack == 1'd0) ? S_RD_REG : S_IDLE; // device addr
						8'd3: r_next_state = S_RD_REG_DONE ; // rd data
						default: r_next_state = S_IDLE;
					endcase
				end
				else
					r_next_state = S_WAIT_RD_REG;
			end
			S_RD_REG_DONE: begin
				r_next_state = S_IDLE;
			end
			default: r_next_state = S_IDLE;
		endcase
	end
	
	

	reg r_rw_data_valid_go;
	reg [7:0] r_iic_rd_data;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_cnt <= 8'd0;
			r_iic_cmd <= 6'd0;
			r_iic_en_go <= 1'b0;
			r_rw_data_valid_go <= 1'b0;
			r_wr_data <= 8'd0;
			r_iic_rd_data <= 8'd0;
		end
		else begin
			case(r_current_state)
				S_IDLE: begin
					r_cnt <= 8'd0;
					r_iic_cmd <= 6'd0;
					r_iic_en_go <= 1'b0;
					r_rw_data_valid_go <= 1'b0;
					r_wr_data <= 8'd0;
				end // S_IDLE
				
				S_WR_REG: begin
					case(r_cnt)
						8'd0: write_byte(STA|WR,{iic_device_addr,1'b0} | 8'b0000_0000);
						8'd1: write_byte(WR,iic_reg_addr);
						8'd2: write_byte(WR|STOP,iic_wr_data);
						default:;
					endcase
				end //S_WR_REG
				
				S_WAIT_WR_REG: begin
					r_iic_en_go <= 1'b0;
					if(w_trans_done == 1'b1)
						r_cnt <= r_cnt + 1'b1;
					else	
						r_cnt <= r_cnt;
				end // S_WAIT_WR_REG
				
				S_WR_REG_DONE: begin
					r_rw_data_valid_go <= 1'b1;
				end // S_WR_REG_DONE
				
				S_RD_REG: begin
					case(r_cnt)
						8'd0: write_byte(STA|WR,{iic_device_addr,1'b0} | 8'b0000_0000);
						8'd1: write_byte(WR,iic_reg_addr);
						8'd2: write_byte(STA|WR,{iic_device_addr,1'b0}| 8'b0000_0001);
						8'd3: read_byte(RD|NACK|STOP);
						default:;
					endcase
				end // S_RD_REG
				
				S_WAIT_RD_REG: begin
					r_iic_en_go <= 1'b0;
					if(w_trans_done == 1'b1)
						r_cnt <= r_cnt + 1'b1;
					else	
						r_cnt <= r_cnt;
				end //S_WAIT_RD_REG
				
				S_RD_REG_DONE: begin
					r_iic_rd_data <= w_rd_data;
					r_rw_data_valid_go <= 1'b1;
				end
				default: begin
					r_cnt <= 8'd0;
					r_iic_cmd <= 6'd0;
					r_iic_en_go <= 1'b0;
					r_rw_data_valid_go <= 1'b0;
					r_wr_data <= 8'd0;
					r_iic_rd_data <= 8'd0;
				end
			endcase
		end 
	end
	
	assign iic_rd_data = r_iic_rd_data;
	assign iic_rw_data_valid_go = r_rw_data_valid_go;
	
	assign iic_busy = (r_current_state != S_IDLE);



endmodule

```

iic底层驱动

```verilog

module iic_driver(
	clk,
	rst_n,
	iic_cmd,
	iic_en_go,
	rd_data,
	wr_data,
	trans_done,
	slave_ack,
	iic_sclk,
	iic_sdat
);

	input 			clk;
	input 			rst_n;
	
	input  [5:0] 	iic_cmd;
	input 			iic_en_go;
	
	output [7:0] 	rd_data;
	input  [7:0] 	wr_data;
	
	output 			trans_done;
	output 			slave_ack;
	
	output 			iic_sclk;
	inout 			iic_sdat;
	

	
	parameter SYS_CLOCK         = 50_000_000;
	parameter IIC_SCLK          = 400_000;
	localparam IIC_SCLK_CNT_MAX = SYS_CLOCK / IIC_SCLK / 4;
	
	reg r_sclk_div_en;
	reg [$clog2(IIC_SCLK_CNT_MAX)-1:0] r_sclk_div_cnt;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_sclk_div_cnt <= 'd0;
		else if(r_sclk_div_en == 1'b1)
			if(r_sclk_div_cnt == IIC_SCLK_CNT_MAX - 1'b1)
				r_sclk_div_cnt <= 'd0;
			else
				r_sclk_div_cnt <= r_sclk_div_cnt + 1'b1;
		else
			r_sclk_div_cnt <= 'd0;
	end
	
	wire w_sclk_plus;
	assign w_sclk_plus = (r_sclk_div_cnt == IIC_SCLK_CNT_MAX - 1'b1) ? 1'b1 : 1'b0; 
	
	reg [7:0] r_wr_data;
	reg [5:0] r_iic_cmd;
	reg r_iic_en_go;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)begin
			r_wr_data <= 8'd0;
			r_iic_cmd <= 6'd0;
		end
		else if(iic_en_go == 1'b1) begin
			r_wr_data <= wr_data;
			r_iic_cmd <= iic_cmd;
		end
		else begin
			r_wr_data <= r_wr_data;
			r_iic_cmd <= r_iic_cmd;
		end
	end
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_iic_en_go <= 1'b0;
		else
			r_iic_en_go <= iic_en_go;
	end
	
	
	
	
	localparam STA  = 6'b000_001;
	localparam WR   = 6'b000_010; 
	localparam RD   = 6'b000_100;
	localparam ACK  = 6'b001_000;
	localparam NACK = 6'b010_000;
	localparam STOP = 6'b100_000;
	
	localparam S_IDLE		  = 7'b0_000_001;
	localparam S_GEN_STA	  = 7'b0_000_010;
	localparam S_WR_DATA   = 7'b0_000_100;
	localparam S_RD_DATA	  = 7'b0_001_000;
	localparam S_CHECK_ACK = 7'b0_010_000;
	localparam S_GEN_ACK   = 7'b0_100_000;
	localparam S_GEN_STOP  = 7'b1_000_000;
	
	
	
	reg [6:0] r_current_state;
	reg [6:0] r_next_state;
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n)
			r_current_state <= S_IDLE;
		else
			r_current_state <= r_next_state;
	end
	
	reg [7:0] 	r_bit_cnt;

	always@(*) begin
		case(r_current_state)
			S_IDLE : begin
				if(r_iic_en_go == 1'b1) begin
					if(r_iic_cmd & STA)
						r_next_state = S_GEN_STA;
					else if(r_iic_cmd & WR)
						r_next_state = S_WR_DATA;
					else if(r_iic_cmd & RD)
						r_next_state = S_RD_DATA;
					else
						r_next_state = S_IDLE;
				end
				else
					r_next_state = S_IDLE; 
			end //S_IDLE
		
			S_GEN_STA: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3) begin
						if(r_iic_cmd & WR)
							r_next_state = S_WR_DATA;
						else if(r_iic_cmd & RD)
							r_next_state = S_RD_DATA;
						else
							r_next_state = S_IDLE;
					end
					else	
						r_next_state = S_GEN_STA;
				end 
				else
					r_next_state = S_GEN_STA;
			end //S_GEN_STA
		
			S_WR_DATA: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd31)
						r_next_state = S_CHECK_ACK;
					else
						r_next_state = S_WR_DATA;
				end 
				else
					r_next_state = S_WR_DATA;
			end // S_WR_DATA

			S_RD_DATA: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd31)
						r_next_state = S_GEN_ACK;
					else
						r_next_state = S_RD_DATA;
				end
				else
					r_next_state = S_RD_DATA;
			end // S_RD_DATA
			
			S_CHECK_ACK: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3) begin
						if(r_iic_cmd & STOP)
							r_next_state = S_GEN_STOP;
						else 
							r_next_state = S_IDLE;
					end
					else
						r_next_state = S_CHECK_ACK;
				end
				else
					r_next_state = S_CHECK_ACK;
			end // S_CHECK_ACK
		
			S_GEN_ACK: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3) begin
						if(r_iic_cmd & STOP)
							r_next_state = S_GEN_STOP;
						else
							r_next_state = S_IDLE;
					end
					else
						r_next_state = S_GEN_ACK;
				end
				else
					r_next_state = S_GEN_ACK;
			end //S_GEN_ACK
			S_GEN_STOP: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3)
						r_next_state = S_IDLE;
					else
						r_next_state = S_GEN_STOP;
				end
				else
					r_next_state = S_GEN_STOP;
			end // S_GEN_STOP	
			default:r_next_state = S_IDLE;
		endcase
	end
	
	

	reg 		  	r_iic_sclk;
	reg 			r_iic_sdat_o;
	reg  			r_iic_sdat_en;
	reg 			r_slave_ack;
	reg [7:0] 	r_rd_data;
	
	always@(posedge clk or negedge rst_n) begin
		if(!rst_n) begin
			r_bit_cnt <= 'd0;
			r_iic_sclk <= 1'b1;
			r_iic_sdat_o <= 1'b1;
			r_iic_sdat_en <= 1'b0;
			r_slave_ack <= 1'b1;
			r_rd_data <= 8'd0;
			r_sclk_div_en <= 1'b0;
		end
		else begin
			case(r_current_state)
			S_IDLE : begin
				r_sclk_div_en <= 1'b0;
				r_iic_sdat_en <= 1'b0;
				//r_iic_sclk <= 1'b1; // dont pull up. Unless the sclk bus is a hiz state
				r_bit_cnt <= 'd0;
				if(r_iic_en_go == 1'b1)
					r_sclk_div_en <= 1'b1;
				else
					r_sclk_div_en <= 1'b0;
			end //S_IDLE
			
			S_GEN_STA: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3)
						r_bit_cnt <= 'd0;
					else
						r_bit_cnt <= r_bit_cnt + 1'd1;
						
					case(r_bit_cnt)
						'd0: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b1; end
						'd1: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b1; end
						'd2: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b0; r_iic_sdat_en <= 1'b1; end
						'd3: begin r_iic_sclk <= 1'b0; r_iic_sdat_o <= 1'b0; r_iic_sdat_en <= 1'b1; end
						default: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b0; end
					endcase
				end 
				else begin
					r_bit_cnt <= r_bit_cnt;
					r_iic_sclk <= r_iic_sclk;
					r_iic_sdat_o <= r_iic_sdat_o;
					r_iic_sdat_en <= r_iic_sdat_en;
				end
			end //S_GEN_STA
			
			S_WR_DATA: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd31)
						r_bit_cnt <= 'd0;
					else
						r_bit_cnt <= r_bit_cnt + 1'b1;
						
					case(r_bit_cnt)
						'd0,'d4,'d8, 'd12,'d16,'d20,'d24,'d28: begin r_iic_sclk <= 1'b0; r_iic_sdat_o <= r_wr_data[7 - r_bit_cnt[4:2]]; r_iic_sdat_en <= 1'b1; end 
						'd1,'d5,'d9, 'd13,'d17,'d21,'d25,'d29: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= r_iic_sdat_o; r_iic_sdat_en <= 1'b1; end
						'd2,'d6,'d10,'d14,'d18,'d22,'d26,'d30: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= r_iic_sdat_o; r_iic_sdat_en <= 1'b1; end
						'd3,'d7,'d11,'d15,'d19,'d23,'d27,'d31: begin r_iic_sclk <= 1'b0; r_iic_sdat_o <= r_iic_sdat_o; r_iic_sdat_en <= 1'b1; end
						default: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b0; end
					endcase
				end 
				else begin
					r_bit_cnt <= r_bit_cnt;
					r_iic_sclk <= r_iic_sclk;
					r_iic_sdat_o <= r_iic_sdat_o;
					r_iic_sdat_en <= r_iic_sdat_en;
				end
			end // S_WR_DATA
			
			S_RD_DATA: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd31)
						r_bit_cnt <= 'd0;
					else
						r_bit_cnt <= r_bit_cnt + 1'b1;
						
					case(r_bit_cnt)
						'd0,'d4,'d8, 'd12,'d16,'d20,'d24,'d28: begin r_iic_sclk <= 1'b0; r_iic_sdat_en <= 1'b0; end 
						'd1,'d5,'d9, 'd13,'d17,'d21,'d25,'d29: begin r_iic_sclk <= 1'b1; r_iic_sdat_en <= 1'b0; end
						'd2,'d6,'d10,'d14,'d18,'d22,'d26,'d30: begin r_iic_sclk <= 1'b1; r_rd_data <= {r_rd_data[6:0],iic_sdat}; r_iic_sdat_en <= 1'b0; end
						'd3,'d7,'d11,'d15,'d19,'d23,'d27,'d31: begin r_iic_sclk <= 1'b0; r_rd_data <= r_rd_data; r_iic_sdat_en <= 1'b0; end
						default: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b0; end
					endcase
				end
				else begin
					r_bit_cnt <= r_bit_cnt;
					r_iic_sclk <= r_iic_sclk;
					r_iic_sdat_o <= r_iic_sdat_o;
					r_iic_sdat_en <= r_iic_sdat_en;
					r_rd_data <= r_rd_data;
				end
			end // S_RD_DATA
			
			S_CHECK_ACK: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3)
						r_bit_cnt <= 'd0;
					else
						r_bit_cnt <= r_bit_cnt + 1'b1;
						
					case(r_bit_cnt)
						'd0: begin r_iic_sclk <= 1'b0; r_iic_sdat_en <= 1'b0; end
						'd1: begin r_iic_sclk <= 1'b1; r_iic_sdat_en <= 1'b0; end
						'd2: begin r_iic_sclk <= 1'b1; r_slave_ack <= iic_sdat; r_iic_sdat_en <= 1'b0; end
						'd3: begin r_iic_sclk <= 1'b0; r_slave_ack <=r_slave_ack; r_iic_sdat_en <= 1'b0; end
						default: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b0; end
					endcase
				end
				else begin
					r_bit_cnt <= r_bit_cnt;
					r_iic_sclk <= r_iic_sclk;
					r_iic_sdat_o <= r_iic_sdat_o;
					r_iic_sdat_en <= r_iic_sdat_en;
					r_slave_ack <= r_slave_ack;
				end
			end // S_CHECK_ACK
			
			S_GEN_ACK: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3)
						r_bit_cnt <= 'd0;
					else
						r_bit_cnt <= r_bit_cnt + 1'b1;
						
					case(r_bit_cnt) 
						'd0: begin 
							r_iic_sclk <= 1'b0; 
							if(r_iic_cmd & ACK)
								r_iic_sdat_o <= 1'b0;
							else if(r_iic_cmd & NACK)
								r_iic_sdat_o <= 1'b1;
							else
								r_iic_sdat_o <= r_iic_sdat_o;
							r_iic_sdat_en <= 1'b1;
						end
						'd1: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= r_iic_sdat_o; r_iic_sdat_en <= 1'b1; end
						'd2: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= r_iic_sdat_o; r_iic_sdat_en <= 1'b1; end
						'd3: begin r_iic_sclk <= 1'b0; r_iic_sdat_o <= r_iic_sdat_o; r_iic_sdat_en <= 1'b1; end
						default: begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b0; end
					endcase	
				end
				else begin
					r_bit_cnt <= r_bit_cnt;
					r_iic_sclk <= r_iic_sclk;
					r_iic_sdat_o <= r_iic_sdat_o;
					r_iic_sdat_en <= r_iic_sdat_en;
				end
				
			end //S_GEN_ACK
			
			S_GEN_STOP: begin
				if(w_sclk_plus == 1'b1) begin
					if(r_bit_cnt == 'd3)
						r_bit_cnt <= 'd0;
					else
						r_bit_cnt <= r_bit_cnt + 1'b1;
						
					case(r_bit_cnt)
						'd0: begin r_iic_sclk <= 1'b0;r_iic_sdat_o <= 1'b0; r_iic_sdat_en <= 1'b1; end
						'd1: begin r_iic_sclk <= 1'b1;r_iic_sdat_o <= 1'b0; r_iic_sdat_en <= 1'b1; end
						'd2: begin r_iic_sclk <= 1'b1;r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b1; end
						'd3: begin r_iic_sclk <= 1'b1;r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b1; end
						default:begin r_iic_sclk <= 1'b1; r_iic_sdat_o <= 1'b1; r_iic_sdat_en <= 1'b0; end
					endcase
				end
				else begin
					r_bit_cnt <= r_bit_cnt;
					r_iic_sclk <= r_iic_sclk;
					r_iic_sdat_o <= r_iic_sdat_o;
					r_iic_sdat_en <= r_iic_sdat_en;
				end
			end // S_GEN_STOP	
			default : begin
				r_bit_cnt <= 'd0;
				r_iic_sclk <= 1'b1;
				r_iic_sdat_o <= 1'b1;
				r_iic_sdat_en <= 1'b0;
				r_slave_ack <= 1'b1;
				r_rd_data <= 8'd0;
				r_sclk_div_en <= 1'b0;
			end
			endcase
		end
	end
	
	assign iic_sclk = r_iic_sclk;
	assign slave_ack = r_slave_ack;
	assign rd_data = r_rd_data;

	assign iic_sdat = (r_iic_sdat_en && !r_iic_sdat_o) ? 1'b0 : 1'bz;
	
	
	reg r_trans_done;
	always@(posedge clk or negedge rst_n)
	begin
		if(!rst_n)
			r_trans_done <= 1'b0;
		else if(r_current_state == S_CHECK_ACK && r_next_state == S_IDLE)
			r_trans_done <= 1'b1;
		else if(r_current_state == S_GEN_ACK && r_next_state == S_IDLE)
			r_trans_done <= 1'b1;	
		else if(r_current_state == S_GEN_STOP && r_next_state == S_IDLE)
			r_trans_done <= 1'b1;
		else
			r_trans_done <= 1'b0;
	end
	
	
	assign trans_done = r_trans_done;


endmodule 
```



## 总结

本工程名为eeprom，如有需要请至github仓库查看！！！本设计只是一主多从的IIC主机，而标准IIC协议是多主多从设计，也就是说SCLK的inout类型。





---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/FPGA](https://blog.csdn.net/zipingpan/category_12609215.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**





