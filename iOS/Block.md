[TOC]

# block的本质是什么

- block的本质是什么
	> Block本质是一个**封装了函数调用以及函数调用环境的OC对象**，Block转换成C++代码之后是一个结构体，结构体中存在一个isa指针。内存如下图：
	
	![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1jp8ox7cfj30t60ten3j.jpg)


# block的变量捕获

- block的变量捕获

	|变量类型|捕获到block内部|访问方式|
	|---|---|---|
	|auto局部变量|会捕获|捕获变量值|
	|static局部变量|会捕获|捕获变量指针|
	|全局变量|不会捕获|直接访问无需捕获|
	- block访问局部变量：
		>- auto变量： 值传递（auto变量离开作用域后会自动销毁，所以需要捕获变量值，以防在访问block访问变量的时候变量已经被销毁。）
		>- static变量： 指针传递（static变量会一直存在内存中，所以只需捕获变量的指针）
	- block访问全局变量：
		>直接访问，**无需捕获**
		```
		auto int age = 10;
		static int height = 10;
		void (^block)(void) = ^{
			// 打印结果：age is 10, height is 20
			NSLog(@"age is %d, height is %d", age, height);
		};
		age = 20;
		height = 20;
		block();
		```


	- 下面这段代码self会被block捕获吗？
		>答案是**会**，因为OC的函数默认会传递两个参数：**self, SEL _cmd**），所以这里的self相当于局部变量，因此会被block所捕获。
		```
		- (void)test {
			void(^block)(void) = ^{
				NSLog(@"lch----%p", self);
			};
			block();
		}
		```

	- 下面这段代码name会被block捕获吗？
		> name不会被block捕获，但是self会被block捕获，从而访问到name。
		```
		@interface Test()
		@property(nonatomic, copy)NSString *name;
		@end

		@implementation Test

		- (void)test {
			void(^test)(void) = ^ {
				NSLog(@"lch----%@", _name);
			};
			test();
		}

		@end
		```

# block 的类型

- block 的类型

	
	>block有三种类型，可以通过调用class方法或者isa指针查看具体类型，这三种类型最终都继承自NSBlock，而NSBlock有继承自NSObject，这也说明block本质是一个OC对象。


	|block的类型|环境|存储区域|执行copy之后|
	|---|---|---|---|
	|__ NSGlobalBlock __|没有访问auto变量的block|程序的数据段区域|什么也不做|
	|__ NSStackBlock __|访问了auto变量的block|栈|从栈上复制到堆|
	|__ NSMallocBlock __| __ NSStackBlock __调用了copy操作之后|堆|引用技术增加|
	注：[关于OC存储区域堆栈内存的区别](https://www.jianshu.com/p/c8e1d91dda99)

# block 的copy操作

- 在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上，比如以下几种情况：
	- block作为函数返回值时
	- 将block赋值给__strong指针时
	- block作为Cocoa API中方法名含有usingBlock的方法参数时
	- block作为GCD方法参数时
- ARC环境下block建议写法
	```
	@property(strong, nonatomic) void(^block)(void);
	@property(copym, nonatomic) void(^block)(void);
	```

# block对象类型的auto变量

- 当block内部访问了**对象类型的auto变量时**
	
	- 如果block在栈上，则不会对auto变量产生强引用
	- 如果block被拷贝到堆上
		- 会调用block内部的copy函数
		- copy函数会调用_Block_object_assign函数
		- _Block_object_assign函数会根据auto变量的修饰符（__strong, __weak, __unsafe_unretained），决定形成强引用还是弱引用。
	- 如果block从堆上移除
		- 会调用block内部的dispose函数
		- dispose 函数内部会调用_Block_object_dispose函数
		- _Block_object_dispose会自动清除block对auto变量的强引用。

- 下面这段代码，当执行到26行时，person对象会被释放吗？

![block的](https://ws4.sinaimg.cn/large/006tKfTcgy1g1mwn5xqvqj30q80gatbb.jpg)

> 答案是**不会**，因为在ARC环境下会自动对block进行copy操作，从栈拷贝到堆上，而person对象的默认修饰符是__strong ,auto，所以block会对person对象形成强引用，执行到26行的时候，block未被释放，所以person对象也不会释放。

# __block修饰符

## __block的作用

> __block可以用于解决block内部无法修改auto变量值的问题
__block不能修饰全局变量、静态变量（staitc）
编译器会将__block变量包装成一个对象，具体结构如下

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1n6su4p42j31mc0sgwop.jpg)

## __block的内存管理

- 当block在栈上时，不会对__blcok变量产生强引用，所以无需进行内存管理。
- 当block被拷贝到堆上时，会对__block变量形成强引用；当block从堆上移除时会执行block内部的dispose函数，从而释放引用的变量。

# block的循环引用问题

## 造成循环引用的原因
> block持有外部对象，外部对象持有block，导致双方都无法释放，造成内存泄漏。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1n7feucdvj30va0iqwks.jpg)

## 解决循环引用

- 使用__weak、__unsafe_unretained解决
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1n7msc3u4j317c0cuwjw.jpg)

- 使用__block解决(只能调用一次block，因为第二次调用的时候person对象已经被置空)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1n7ojzno4j31jk09mqal.jpg)


# 相关面试题

- block的本质是什么，原理是怎样的？

	> block本质是一个封装了函数调用以及调用环境的OC对象，block内部提供了函数的调用地址，以及函数调用所需的变量。

- __block的作用是什么？有什么注意点？
	> __block可以让block在内部修改外部变量。使用时要注意引起循环引用的问题，因为block会持有__block修饰的变量。

- block的属性修饰词为什么是copy？

	> 因为block不进行copy操作的话，就不会在堆上，从而无法控制block的释放时机。

- block在给NSMutableAraay添加对象时需不需要用__block修饰？

	>不需要。因为添加对象不用修改NSMutableArray的地址。
