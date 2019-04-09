
# KVO


- KVO的本质是什么？（通过什么方式实现对一个对象的监听）
> KVO本质是在对象进行监听之后，利用runtime动态生成一个子类，然后让被监听的实例对象的isa指针指向这个子类；在这个子类中重写被监听属性的setter方法，当修改属性值时，会调用`willChangeValueForKey:`，父类原来的setter方法，最后调用`didChangeValueForKey:`，在`didChangeValueForKey:`的内部会触发监听器的监听方法       （`observeValueForKeyPath:ofObject:change:context:`），从而实现对一个对象属性的监听。

![](https://ws2.sinaimg.cn/large/006tNc79gy1g1v7yyzq34j31k90u0ase.jpg)


- 如何手动触发KVO？
> 在修改属性值的时候，手动调用`willChangeValueForKey:`和`didChangeValueForKey:`方法

- 直接修改成员变量会触发KVO吗？
> 不会，因为成员变量没有生成setter方法。

# KVC
- KVC的常用API

```
- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;
- (void)setValue:(id)value forKey:(NSString *)key;
- (id)valueForKeyPath:(NSString *)keyPath;
- (id)valueForKey:(NSString *)key; 

```


- KVC 的赋值过程
> 在调用`setValue:forKey:`之后，会按`setKey:`、`_setKey:`的顺序查找方法，找到了则直接设值，没找到则会查看`accessInstanceVariablesDirectly`的返回值，如果返回YES，则会按照`_key`、`_isKey`、`key`、`isKey`、的顺序查找**成员变量**，找到了则直接设值，找不到或者`accessInstanceVariablesDirectly`的返回值为NO，则**调用setValue:forUndefinedKey:并抛出异常NSUnknownKeyException**

![setValue:forKey:设值过程](https://ws1.sinaimg.cn/large/006tNc79gy1g1v1zet91vj31v20tygxc.jpg)


- KVC 的取值过程
> 调用`valueForKey:`的时候，会按`getKey`、`key`、`isKey`、`_key`顺序查找方法，找到了则直接取值，否则查看`accessInstanceVariablesDirectly`的返回值，如果返回YES，则会按照`_key`、`_isKey`、`key`、`isKey`查找成员变量，找到了则直接取值，找不到或者`accessInstanceVariablesDirectly`的返回值为NO，则**调用setValue:forUndefinedKey:并抛出异常NSUnknownKeyException**

![valueForKey:取值过程](https://ws2.sinaimg.cn/large/006tNc79gy1g1v6l1zcf8j31v40ton8c.jpg)


- 通过KVC修改属性或成员变量值会触发KVO吗？

> 会触发
