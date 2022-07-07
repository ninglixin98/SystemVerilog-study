# SystemVerilog —— 第六章 验证的量化和覆盖率
## 6.1 覆盖率类型
### 6.1.1 概述
- 覆盖率是衡量设计验证完备性的一个通用词语
- 测试结束会生成一个覆盖率文件

### 6.1.2 代码覆盖率
- 不需要添加任何额外的HDL代码
- 运行完测试后，代码覆盖率工具就会创建相应的数据库
- 仿真器都有代码覆盖率工具，并且可以将代码覆盖率转换为可读格式
- 代码覆盖率仅表示设计已执行
- 代码覆盖率类型
	- 行覆盖率
		- 表示多少行代码已经被执行过
		- 行覆盖率也可以成为块覆盖率，其中块是执行单个语句时的执行序列，如if块。块中有wait语句时，会被分为两个块。
	- 路径覆盖率（分支覆盖率）
		- 在穿过代码和表达式的路径中有哪些已经被执行过
		- 用来对条件语句分支的执行进行追踪

	- 条件覆盖率
		- 用来衡量布尔表达式中各个条件真伪判断的执行轨迹

	- 翻转覆盖率（跳转覆盖率）
		- 哪些单位比特变量的值为0或1
		- 衡量寄存器跳转的次数
		- 通过端口跳转覆盖率，来确保模块之间的互动
	- 状态机覆盖率
		- 状态机哪些状态和状态转换已经被访问过

### 6.1.3 断言覆盖率
- 断言是用于一次性地或在一段时间对一个或多个设计信号在逻辑或时序上的声明性代码
- 断言常用于查找错误，例如两个信号是否应该互斥，或者请求与许可信号之间的时序等
- 一旦检测到问题，仿真立即停止
- 可以使用 cover property来测量关心的信号值或状态是否发生
- 仿真结束会自动生成断言覆盖率数据

### 6.1.4 功能覆盖率
- 验证的目的是确保设计在实际环境中的行为正确
- 功能覆盖率是和功能设计意图紧密相连的，有时也被称为描述覆盖率，而代码覆盖率则是衡量设计的实现情况
- 每一次仿真都会产生一个带有覆盖率信息的数据库，记录随机游走的轨迹。把这些信息合并在一起，就可以得到功能覆盖率，从而衡量整体的进展程度
- 通过分析覆盖率数据可以决定如何修改回归测试集

### 漏洞率曲线（缺陷曲线）
- 在一个项目实施期间，应该保持追踪每周有多少个漏洞被发现

## 6.2 功能覆盖策略
- 收集信息而非数据
- 只测量需要的内容（收集覆盖率开销很大）
- 验证的完备性
	- 如果代码覆盖率低但功能覆盖率高，说明验证计划不完备，测试没有执行所有的设计代码
	- 如果代码覆盖率高但功能覆盖率低，说明测试没有定位到所有感兴趣的状态上
	- 目标是同时驱动代码覆盖率和功能覆盖率

## 6.3 覆盖组
- 覆盖组与类相同，一次定义后即可多次例化
- covergroup可以包含一个或者多个coverpoint，且全都在同一时间采集
- covergroup可以定义在类中，也可以定义在interface和module中
- covergroup可以采样任何可见变量
- 一个类中可以包含多个covergroup
- 当有多个covergroup时，每个covergroup可以根据需要使能或禁止
	- cg.stop();
	- cg.start();
- 每个covergroup可以定义单独的触发采样事件，允许从多个源头收集数据
- covergroup必须被例化才能收集数据
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706165357.png)

	- 定义
		- covergroup coverport;
        coverpoint tr.port;
endgroup
	- 例化
		- coverport cg1 = new();
			- 起实例名后可以多次例化
		- coverport = new();
			- 默认为covergroup名
	- 采样
		- 使用sample()函数采样
			- coverport.sample();
		- 使用wait或@在信号上阻塞采样
			- event e1;
covergroup coverport @(e1);
        coverpoint tr.port;
endgroup

## 6.4 覆盖率采样

### 6.4.1 coverpoint和bin
- 当你在coverpoint指定采样一个变量或表达式时，SV会创建很多的仓（bin）来记录每个数值被捕捉到的次数，这些bin是衡量功能覆盖率的基本单位
- coverpoint 中可以定义多个cover bin，或者SV会帮助自动定义多个cover bin
- 覆盖率就是采样值的数目除以bin的数目，例如一个信号的域为7:0，正常会自动分配8个Bin，如果仿真过程中有7个值被采样到，则覆盖率为7/8
- coverpoint命名
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706170038.png)

### 6.4.2 bin的创建和应用

- 自动创建bin
	- SV会默认为coverpoint创建bin
	- 可以通过auto_bin_max来指定自动创建bin的最大数目
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706170302.png)

- 手动创建bin
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706170400.png)

		- 右边代表关心想要采样的具体值

### 6.4.3 条件覆盖率
- 可以使用关键词 iff 给coverpoint添加条件
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706171206.png)

	![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706171217.png)

### 6.4.4 翻转覆盖率
- coverpoint可以记录变量的跳转情况和跳转次数
- 使用=>表示跳转
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706171522.png)

### 6.4.5 wildcard覆盖率

- 可以使用关键词 wildcard 来识别 x, z, ?，他们会被当成0或1的通配符
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706171640.png)

### 6.4.6 忽略bin
- 可以使用 ignore_bins 来排除不需要的Bin，最终不会计算在覆盖率内
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706171817.png)

### 6.4.7 非法的bin

- 有些采样值是非法的，出现应该报错
- 可以使用 illegal_bins 对bin进行标示
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706172006.png)

### 6.4.7 交叉覆盖率
- coverpoint只能记录单个变量的值，如果需要记录在某一时刻，多个变量之间值的组合情况，需要使用交叉覆盖率（cross）
- cross语句只允许带 coverpoint或者简单的变量名
- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706185732.png)

	- 使用cross表示看kind和port的组合情况

- 排除部分cross bin
	- 通过使用ignore_bins、binsof、intersect分别指定coverpoint和值域，这样可以清除很多不关心的cross bin
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706185852.png)


		- 交叉之后一共有8×11种组合
		- binsof(port) intersect {7};    排除port中的7，两个交叉，即排除了1×11个组合

- 精细的交叉覆盖率指定
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706190611.png)

		- 通过binsof指定关心的情况，不需要加ingore排除
		- 还可以写成 b1 = binsof(b) intersect{1};


- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706190856.png)

	- 错误，必须保证4个bin和6个bin分别包含了两个coverpoint的所有值，才为24。否则剩余的在cross时，系统会自动分配bin

## 6.5 覆盖选项
- 单个实例的覆盖率
	- 如果对covergroup例化了多次，默认情况下SV会将所有实例的覆盖率合并到一起
	- 要得到单个实例覆盖率需要添加选项 option.per_instance = 1;
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706191030.png)

- 注释
	- 如果有多个covergroup实例，可以通过参数对每一个实例传入单独的注释，最终显示在覆盖率总结报告中
	- 选项 option.comment
	- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706191039.png)

- 覆盖次数限定
	- 通过 option.at_least 可以在covergroup中声明来影响所有的coverpoint，也可以在coverpoint中声明来只影响该coverpoint下的bin
	- 默认为1
- 覆盖率目标
	- 默认为100%
	- 通过 opton.goal = 90;来设定目标为90%

- covergroup方法
	- sample(); 采样
	- get_coverage()/get_inst_coverage(); 获取覆盖率，返回0-100的real数值
	- set_inst_name(string);设置covergroup的名称
	- start()/stop(); 打开或关闭covergroup

## 6.6 数据分析

- 使用 $get_coverage() 可以得到总体覆盖率
- 可以使用 covergroup_inst.get_inst_coverage() 来获取单个covergroup的覆盖率






