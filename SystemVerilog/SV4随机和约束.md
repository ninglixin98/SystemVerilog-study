# SystemVerilog —— 第四章 随机和约束
## 4.1 随机约束和分布
- 随机测试与定向测试
	- 与定向测试相比，更加容易暴露出未知的问题
	- 随机测试环境比定向测试复杂，他需要激励、参考模型和在线比较
	- 随机测试相比定向测试可以减少代码量，而产生的激励也更加多样

- 随机和约束
	- 随机不是绝对的自由，而是应该有约束，否则会产生很多无效和非法的激励
	- 随机变量只能封装在类中

- 随机变量的类型
	- 整型
	- 定长数组
	- 动态数组
	- 关联数组
	- 队列
	- 句柄

### 4.1.1 随机变量的声明和使用
- 随机变量的声明
	- rand
		- `rand bit [31:0] src;`
	- 周期随机性 randc，随机过的数不会重复
		- `randc bit [7:0] kind;`

> rand和randc只能修饰类的成员变量

- 约束
	- 关键词 constraint
		```verilog
		constraint c {src > 10;
                        sec < 10;}
		```
	- 约束求解器进行求解
		- 种子和仿真器版本相同，最后随机的结果相同
	- SV只能随机化2值逻辑（可以使用4值逻辑，但不会随机出x,z）
	- 约束块内可以使用条件语句
	- 约束块内语句是并行的，需要同时满足约束
		- 约束是双向的，会同时计算所有随机变量的约束
	- 约束会被子类继承

- 随机化：关键词 randomize()
	- 句柄.randomize();
		- 有返回值，返回1代表随机化成功，返回0代表随机化失败
	- 系统函数
		- `std::randomize(src,date);`
		- 无需类和句柄直接对变量初始化

- 权重的分布：关键词 dist
	- 带有一个值的列表以及相应的权重，中间用:=或:/分开
		- :=
			- 值范围内的每一个值的权重是相同的
			- `src dist {0:=40, [1:3]:=60};`
		- :/
			- 权重要平均分到值范围内的每一个值
			- `src dist {0:/40, [1:3]:/60};`

- inside运算符
	- c inside {[1:10]};
	- 表示变量应属于某一个值的集合
	- 变量在集合内的取值概率是相等的
	- 集合里也可以使用变量
	- 集合里可以使用$代表最大值或最小值

## 4.2 约束块控制
- 一个类可以包含多个约束块
- 打开或关闭约束块
	- `constraint_mode();`
		- 关闭：constraint_mode(0);
		- 打开：constraint_mode(1);
	- 句柄.constraint_mode();打开或关闭全部约束块
	- 句柄.约束块名.constraint_mode();打开或关闭指定约束块

- 使用或禁止随机变量
	- rand_mode();
		- 关闭所有随机变量：packet_a.rand_mode(0);
		- 打开其中个别随机变量：packet_a.src.rand_mode(1);
	- 句柄.随机变量(可选).rand_mode();

- 内嵌约束
	- SV允许使用 randomize() with来增加额外的约束
	- randomize() with {}

- 软约束
	- 关键词 soft
	- soft addr inside {[1:1000]};
	- 表示约束有冲突时，优先级更低
	- 全部都是软约束时，就近的约束生效

- 子类会继承父类的约束
- 子类的约束与父类重名时，子类的约束会覆盖父类的约束

## 4.3 随机函数
- 在执行randomize之前或之后自动调用的函数,可以看做回调函数
	- pre_randomize();
	- post_randomize();

- 平均分布，返回32位有符号随机数：$random();
- 平均分布， 返回32位无符号随机数：$urandom();
- 在指定范围内的平均分布：$urandom()_range();
- 在调用randomize()函数时，()内可以传递变量的一个子集，这样只会随机化这几个变量
	- 所有约束仍然生效
	- 传递的变量不论是否用rand修饰都可以

## 4.3 数组约束

- 数组的属性约束
	- 数组的大小应该限制范围
		- `{d.size() inside {[1:10]};};`

- 约束数组的元素
	- 用 foreach 对数组的每一个元素进行约束
		- `foreach(len[i])  len[i] inside {[1:255]};`
	- 注意数组范围
		- `foreach(da[i])  da[i] <= da[i+1];`
		- i+1超出范围，报错

- 产生唯一元素值的数组
	- 使用嵌套 foreach
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706102900.png)

	- 使用 randc
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706102916.png)


- 随机化句柄数组
	- 使用rand后，随机函数会随机化数组中的每一个句柄所指向的对象中rand修饰的属性
	- 如果要产生多个随机对象，需要建立随机句柄数组

## 随机控制

- 产生随机序列
	- 关键词：randsequence (stream)
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706103229.png)

		- 1,2,5表示权重，会随机进入语句

	- 关键词：randcase
		- ![输入图片描述](http://www.ninglixin.com/wp-content/uploads/2022/07/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220706103238.png)

		- 1，8，1表示权重








