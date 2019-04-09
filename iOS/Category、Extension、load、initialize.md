# Category

- Category 使用场合

    > 通常在把一个类拆解为很多模块就可以使用分类来实现

- Category 的底层结构
![](https://ws3.sinaimg.cn/large/006tNc79gy1g1wbw5825nj30ye0h4q69.jpg)

- Category 的加载处理过程
    1. 通过Runtime加载某个类的所有category数据
    2. 把所有的category的方法、协议、属性数据合并到一个数组中
        - 注意：后参与编译的category数据会在数组的前面
    3. 将合并后的分类数据（方法、协议、属性），**插入到类原来数据的前面**

- Category实现原理

    > Category编译之后的底层结构（category_t）存储着分类的方法、协议、属性等信息，在程序运行的时候，会通过Runtime将这些信息合并到类信息里边去（类对象、元类对象中）


- Category 和 Class Extension的区别是什么

    1. Extension在编译的时候，他的数据就已经包含在类信息里边了；而Category是在运行的时候才将数据合并到类信息中。
    2. Extension可以直接添加成员变量，而Category无法直接添加成员变量（可通过关联对象来实现）。

- Category中有load方法吗？load方法是什么时候调用的？load方法能继承吗?

    > Category中有load方法，load方法是runtime加载类或分类的时候运行的；load方法可以继承，但是一般很少会主动调用load方法。

- load方法和initialize方法的区别是什么？
    
    1. load方法是在类或分类**加载的时候调用的，而且只会调用一次**，而initialize方法是在类**第一次接收到消息的时候调用的，每一个类只会initialize一次，父类的initialize方法可能会被调用多次**。
    2. load方法是根据**函数地址直接调用的**，而initialize方法是通过**objc_msgSend调用的**


- load 方法的调用顺序?

    > 先编译的类先调用load，调用类的load会先调用父类的load，最后再调用分类的load，先编译的分类先调用load。

- initinalize 方法的调用顺序

    > 先初始化父类，再初始化子类，如果父类存在分类，会先调用父类的initialize。