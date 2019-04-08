
# KVO
- KVO的本质是什么？（通过什么方式实现对一个对象的监听）
> KVO本质是在对象进行监听之后，利用runtime动态生成一个子类，然后让被监听的实例对象的isa指针指向这个子类；在这个子类中重写被监听属性的setter方法，当修改属性值时，会调用`willChangeValueForKey:`，父类原来的setter方法，最后调用`didChangeValueForKey:`，在`didChangeValueForKey:`的内部会触发监听器的监听方法（`observeValueForKeyPath:ofObject:change:context:`），从而实现对一个对象属性的监听。

- 如何手动触发KVO？
> 在修改属性值的时候，手动调用`willChangeValueForKey:`和`didChangeValueForKey:`方法

- 直接修改成员变量会触发KVO吗？
> 不会，因为成员变量没有生成setter方法。

# KVC
- KVC 的实现原理是怎么样的？
