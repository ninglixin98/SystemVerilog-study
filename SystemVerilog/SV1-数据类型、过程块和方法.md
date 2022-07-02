# SV 第一章 —— 数据类型、方法和过程块

## 1. 数据类型

### 1.1 内建数据类型

- SV中引入了新的数据类型：logic和bit。logic为无符号四值逻辑，bit为无符号二值逻辑。

- 变量类型分类
   * 按逻辑类型分
     + 四值逻辑类型：integer、logic、reg、net-type(wire、tri)
     + 二值逻辑：byte、shortint、int、longint、bit

	* 按有无符号分

		+ 有符号类型：byte、shortint、int、longint、integer
		+ 无符号类型：bit、logic、reg、net-type

- 浮点类型

	- SV：添加 shortreal 用来表示32位单精度浮点类型

	- verilog：real表示双精度浮点类型

- 类型的转换

	- 方法：目标类型’(原数据)

### 1.2 定宽数组

#### 1.2.1 数组的声明

- 一维数组

	+ `int lo_hi [0:15];`

- 多维数组

	- 完整声明

		- `int array2 [0:7][0:3];`

	- 紧凑声明

		- `int array2 [8][4];`

	- 设置最后一个原素

		- `int array2 [7][3] = 1;`
	- 定宽数组中，数组下标和位下标可以同时使用

		- `src[0][2:1];`

#### 1.2.2 多维数组类型

- 组合型（packed）

	- `bit [3][7:0] b_pack;`

	- 左边为高维数组，右边为低维数组

	- 数组高维是连续排列的，连续存储在一个word里，占用内存少

	- SV相比于verilog，扩展了组合型数组的类型，包括event、bit、bit、logic、int、longint、shortreal、real类型

	- 4值数据的数组占据的空间要乘2

- 非组合型unpacked

	- `bit [7:0] b_unpack [3];`

	- `bit b_unpack [3][7:0];`

	- 数组名右边为高维，左边为低维

	- 数组高维是不连续的，分别存储在3个不同的word里，占用内存多

	- 4值数据的数组占据的空间要乘2

- 混合型数组

	- `int [2][3] arr [4][5];`

	- 数组维度从高到低先看右边

	- 从高维到低维：4\*5\*2\*3

#### 1.2.3 数组的初始化和赋值

- 一维数组

	- 格式为：数据类型 数组名 \[位宽\] = '{数值1,数值2,.....};

	- `int ascend [4] = '{0,1,2,3};`

- 多维数组

	- 组合型

		- 因为是连续的，和向量一样，使用{}，不需要加'

		- `logic [3:0][7:0] a = 32'h0;`

		- `logic [3:0][7:0] a = {16'hz,16'h0};`

		- 使用连接符合时必须标明位宽，否则默认为32位

		- 左右两边大小和维度不同时，也可以进行操作

	- 非组合型

		- 需要对每一个数进行赋值，使用'{}

		- `int d[0:1][0:3] = '{'{1,2,3,4},'{5,6,7,8}};`

		- 注意：二维数组一定要有2层{}

		- 也可以用'{default:}全部赋值默认值

		- `int d[0:1][0:3] = '{default: 8'h55};`

		- 左右大小和维度必须严格一致

#### 1.2.4 数组循环

- for 循环

	- `for (int i = 0; $size(src); i++)`

	- $size(数组名) 可以获取数组的长度


- foreach 循环

	- 一维数组 
		- `foreach(dust[j])`

		- foreach比较方便，无需声明变量j即可使用，遍历从0到j的所有元素

	- 多维数组

		- 遍历全部数组
		
			- `foreach(md[i,j])`
			
		- 遍历高维数组
			- `foreach(md[i])`

	
		- 遍历低维数组
			- `foreach(md[,j] )`


#### 1.2.5 数组的系统函数

- $dimensions(arr_name) 返回数组维度

- $left(arr_name, dimension) 返回指定维度的最左索引值

- $size(arr_name, dimension) 返回指定维度的数组大小

- $increment(arr_name, dimension) 返回指定维度最左的索引值是否大于等于最右的索引值，是返回1，否返回-1

- $bits(expression) 返回数组存储的比特数目

#### 1.2.6 数组的复制和比较

- 复制

	- 使用 =

- 比较

	- 使用数组循环比较每一个原素

	- 直接使用== 和 != 对两个数组直接操作

#### 1.2.7 数组的方法

- 数组缩减方法

	- 数组求和

		- `on.sum;`

		- 需要注意位宽

	- 数组求积

		- `on.product;`

	- 数组与

		- `on.and;`

	- 数组或

		- `on.or;`

	- 数组异或

		- `on.xor;`

- 数组定位方法

	- 找出最大值

		- `q.min();`

	- 找出最小值

		- `q.max();`

	- 找出具有唯一值的队列

		- `q.unique();`

	- 定位方法find

		- find
			- find_index;

			- find_first;

			- find_last;

			- find_first_index;

			- find_last_index;
		- 表达式with(item)

			- tq = d.find with (item \> 3);

			- 返回大于3的值


### 1.3 动态数组

- 可以在运行过程中调整大小
#### 1.3.1 声明
- `int dyn [];`

- 重新分配大小

	- `dyn = new [5];`

	- 动态数组可以通过new随时改变大小
	- dyn = new [20] (dyn);
	- 重新分配大小并拷贝原数组dyn

#### 1.3.2 动态数组内建的子程序

- 删除所有元素

	- `dyn.delete();`

- 获取数组大小

	- `dyn.size();`

### 1.4 队列

- 可以在任何地方添加、删除、访问元素

#### 1.4.1  声明

- `q[$];`

- 使用\$符号的下标

#### 1.4.2 赋值

- `q[$] = {2,3};`

- {}前不加'

- 索引值从左到右为0，1，2...

#### 1.4.3 队列方法

- 在X位置插入元素X

	- `q.insert(1,4);`

	- 在第二个元素前插入4

- 放入元素

	- `q.push_back(5);`

	- 从尾部放入

	- `q.push_front();`

	- 从头部放入

- 取出元素

	- `q.pop_front();`

	- 从头部取出

	- `q.pop_back();`

	- 从尾部取出

- 删除元素

	- `q.delete();`

- 注意：使用pop方法时要注意队列不能为空，否则会报错。

- 在方法中对队列进行操作时，需要在参数列表处加上ref。

### 1.5 关联数组

- 用来保存稀疏矩阵的元素

#### 1.5.1 声明

- 关联数组采用在方括号中放置数据类型的形式来进行声明，数组名左边为存放的数据的类型，右边为地址值的类型。

- `bit [63:0] assoc [int];`
#### 1.5.2 方法
- 找到并删除第一个元素
 

- assoc.first(idx);
 assoc.delete(idx);

### 1.6 结构体

- 结构体表示变量的合集

- 可以使用组合型和非组合型来限定数组类型

- `struct packed { bit [7:0] r ,g, b; } pixel;`

#### 1.6.1 声明和创建
- 使用struct创建结构

	- `struct { bit [7:0] r ,g, b; } pixel;`

- 使用typedef创建新类型，利用新类型声明更多的变量

	- `typedef struct { bit [7:0] r, g, b; } pixel_s;`
- 声明： `pixel_s my_pixel;`

#### 1.6.2 赋值
- 单个赋值
- `my_pixel = '{'h10, 'h10, 'h10};`

- 结构体名.变量名

- `my_pixel.r = 'h10;`


### 1.7 枚举类型

- 规范的操作码更有利于代码的编写和维护

- 枚举类型 enum 经常和 typedef 搭配使用，便于共享使用，不加 typedef 默认为缺省类型

	- `typedef enum {INIT, DECODE, IDLE} fsmstate_e;`

- 可以通过 XX.name() 来获取枚举类型的值的值

- 将枚举类型转换为整型时INIT，DECODE，IDLE默认分别为0，1，2

- 枚举类型能直接复制给整型，整型不能直接复制给枚举类型，需要进行格式转换

- 枚举类型默认为int型，SV中可以指定类型

	- t`ypedef enum bit {INIT, DECODE, IDLE} fsmstate_e;`

- 枚举类型可以被赋值，但是值必须依次增加

	- {INT = 1, DECODE = 2, IDLE = 3}

### 1.8 字符串

- 存储单元为byte类型

- 没有空字符，内存是动态分配的

#### 1.8.1 声明和赋值

- `string s;`



- `s[0] = "h";`

- 注意：字符串索引从左到右为0，1，2...

#### 1.8.2 字符串方法

- 获取第X个字符

	- `s.getc(0);`

- 改变大小写

	- `s.tolower();`

- 获取字符串长度

	- `s.len();`

- 改变最后一个字符

	- `s.putc(s.len()-1, "-");`

- 获取第2到5个字符

	- `s.substr(2, 5);`

- 字符串转化为整型

	- 十进制

		- `str.atoi();`

	- 十六进制

		- `str.atohex();`

	- 八进制

		- `str.atooct();`

	- 二进制

		- `str.atobin();`

- 整型转换为字符串

	- 十进制

		- `i1.itoa();`

- 生成新的字符串
	- `$sformatf();`
- {} 拼接
	- `$psprintf();`


### 1.9 自定义类型(SV新加)

- typedef

	- typedef enum

	- typedef struct

## 2. 方法和过程块

### 2.1 过程块
- always

	- always是硬件层面的，是可综合的，用来设计

- initial

	- initial是软件层面的，是不可综合的，用来验证和测试

### 2.2 软件方法------函数function

      function int dou(input a);
      endfunction

- 可在参数列表中定义输入参数(input)、输出参数(output)、输入输出参数(inout)、引用参数(ref)

	- 只有数据变量才能被声明为ref，线网类型不能被声明为ref

- output在函数和任务结束后才会更新外部数据

- ref类似指针，是即时更新数据

- const ref可以检测变量数据但不能修改

- 可以返回数值和不返回数值（void）

- 也可用函数名替代return返回

    myfunc1 = x \* y;
    return x \* y;

- 有返回值但不想使用返回值时，可以用 void'() 将函数转化为无返回值

- 默认的返回类型为logic

- 默认的参数类型为1位宽的logic

- 参数传递时可以跟着名字对应

	- `myfunc1(.y(20));`

### 2.3 软件方法------任务task

     task my_task (output logic [31:0] x, input logic y);
     endtask；

- 与function的区别

	- task无法通过return返回结果，只能通过input、output、ref返回

	- 能使用return跳出任务

	- task可以内置耗时语句，而function不能

	- function只能用于数字或逻辑运算，task可用于需要耗时的信号采样或驱动场景

	- 如果要调用function，则function和task都可调用。如果要调用task，则建议使用task调用，因为task可能含有耗时语句。

	- 函数和任务的参数列表都可以为空

### 2.4 变量声明周期

- 静态static

	- 伴随程序开始到结束一直存在，可被多个方法/进程共享

	- static方法， 内部变量默认为static类型，可对内部定义的变量进行单个声明

	- 在声明static变量时，应对其进行初始化

	- task、function、module、interface、package、program默认为static类型

- 动态automatic

	- 在方法开始时创建，方法结束后被销毁

	- automatic方法，内部变量默认为automatic类型，可对内部定义的变量进行单个声明

## 3. 设计例化和连接

- 模块例化

	- 在上层例化底层模块，或TB例化DUT时，均需完成模块例化。

	- 例化时需要注意模块名、参数例化传递、例化名和端口例化对应

- 模块连接

	- 在testbench中的连接指的是有硬件模块参与作为信号驱动方或负载方

	- 两个硬件可由logic类型完成连接

	- 硬件与激励连接，需要考虑激励一端如何正确产生数据并发送至DUT一端，同时对DUT的反馈信号做出响应




