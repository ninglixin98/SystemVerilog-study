# SystemVerilog —— 第二章 接口、采样和数据驱动
## 2.1 接口
### 2.1.1 接口的优势
- 相比于input、output，接口（interface）连接更加方便。
- 将多个信号整合起来表示单一的抽象端口。
- 多个模块可以使用同一个接口，避免分散的多个端口信号连接。

### 2.1.2 接口的内容
- interface和module的异同：
	- 相同点：接口使用语法和module类似，都需要包含端口的信息。
	- 不同点：接口不允许包含设计层次，即接口无法例化module，但接口可以例化接口。

- 接口可以封装通信协议。
- 接口中可以嵌入与协议有关的断言检查、功能覆盖率收集等模块。
- 接口中可以进一步声明modport来约束信号方向。
	- modport master (input data, output ready);
	- 调用方法：bus.master

- 接口也可以有端口，如外部的时钟信号、复位信号等。

### 2.1.3 接口的例化和使用
- 和模块（module）的例化方法一致。
- 例化时必须连接到一个端口实例，或者另外一个接口端口
	```verilog
	rkv_mcdf_if mcdf_if();
	```
- 接口在TB中进行例化。
- 索引接口中信号的方法：
	- 端口名.信号名

- 利用接口，将测试平台TB和DUT连接在一起。

## 2.2 采样和数据驱动
### 2.2.1 竞争问题的避免方法

- 使用非阻塞赋值
	> 阻塞赋值： out1 = din; out2 = out1;  会综合成一个寄存器
	> 非阻塞赋值：out1 <= din; out2 <= out1; 会综合成两个寄存器
- 给出尽量明确的驱动时序和采样时序
	- 在采样和驱动时，人为添加延迟（clocking block）
	- 对于依然存在delta-cycle的情况时，在采样事件前的某时刻中进行采样
	> delta-cycle: 为了模拟实际电路的延迟，默认情况下，时钟对于组合电路的驱动会添加一个无限最小时间的延迟（delta-cycle），当clk和被采样数据之间存在delta-cycle时，采样可能会出现问题。

### 2.2.2 clocking block（时钟块）
- 时钟块：基于时钟周期对信号进行采样和驱动，从而消除信号竞争。
#### 2.2.2.1 clocking block的使用方法
![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/06/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220627140625.png)

- 第一行定义了一个clocking块bus，由clock1的上升沿来驱动和采样；
- 第二行指出，会在clock1上升沿前10ns进行输入采样，在clock1上升沿的后2ns进行输出采样；
- 第三行声明要采样的三个输入信号；
- 第四行声明了要驱动的信号，该信号是时钟下降沿，即覆盖了原有是默认输出事件；
- 第五行的addr也采用了自身定义的采样事件，即在clock1上升沿的前1step，step为时间片。

#### 2.2.2.2 clocking block的特点

- clocking block不但可以定义在interface中，还能定义在module和program中。
- clocking block中的信号不是自己定义的，而是由interface或其他声明clocking的模块定义的
- clocking block应定义默认的采样事件，如果不定义，则会默认在采样事件前1step进行输入采样，在采样事件后的#0进行输出驱动
- 可以在定义信号方向时，定义新的采样事件对默认事件进行覆盖




