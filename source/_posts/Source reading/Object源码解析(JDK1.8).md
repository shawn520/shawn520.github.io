---
title: Object源码解析(JDK1.8)
date: 2018-10-21 10:45:26
tags: JDK源码
categories: 好好学习
copyright: true

---

# Object源码解析(JDK1.8)

Object类是Java中所有类的基类，在编译时会自动导入，位于java.lang包中，而Object中具有的属性和行为，是Java语言设计背后的思维体现。**这里写的代码是JDK8中的，其他版本的JDK可能略有不同。**   包含的方法如下图：

所有方法按照字母`a-z`排序  

![object.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pLmxvbGkubmV0LzIwMTgvMTAvMjEvNWJjYmYyYTc5ZDM4NC5wbmc)

<!-- more-->

Object类方法说明：

Object类中的方法大部分都是native方法（如上图中方法前有一个绿色的实心小圆圈，圆圈的右上角有大写字母N，表示该方法是一个native方法），用此关键字修饰的方法是java中的本地方法，一般都是用C/C++语言实现的。



### 构造方法

Object类中没有显示的提供构造方法，是编译器默认提供的。

### `registerNatives()`方法

```java
private static native void registerNatives();
static {
    registerNatives();
}
```

`registerNatives()`是一个本地方法，具体是用C/C++在DLL中实现的，在类加载的时候执行。`private`修饰使方法私有化，在类加载时执行静态代码块调用该方法。

### `getClass()`方法

```java
    /**
     * Returns the runtime class of this {@code Object}. The returned
     * {@code Class} object is the object that is locked by {@code
     * static synchronized} methods of the represented class.
     *
     * <p><b>The actual result type is {@code Class<? extends |X|>}
     * where {@code |X|} is the erasure of the static type of the
     * expression on which {@code getClass} is called.</b> For
     * example, no cast is required in this code fragment:</p>
     *
     * <p>
     * {@code Number n = 0;                             }<br>
     * {@code Class<? extends Number> c = n.getClass(); }
     * </p>
     *
     * @return The {@code Class} object that represents the runtime
     *         class of this object.
     * @jls 15.8.2 Class Literals
     */
    public final native Class<?> getClass();
```

`getClass()` 被native修饰，同样也是调用本地方法实现的。返回当前对象运行时的类，`final`关键字修饰说明这个方法不能被子类重写。

### `hashcode()`方法

```java
    /**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     * <p>
     * As much as is reasonably practical, the hashCode method defined by
     * class {@code Object} does return distinct integers for distinct
     * objects. (This is typically implemented by converting the internal
     * address of the object into an integer, but this implementation
     * technique is not required by the
     * Java&trade; programming language.)
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
    public native int hashCode();
```

`Object类`中的`hashcode()`方法用来鉴定两个对象是否相等，返回对象在内存中地址转换成的一个 `int`类型的值。如果没有重写`hashcode()`方法，任何对象的`hashcode()`方法都是不相等的。

### `equals()`方法

入参：Object

返回值：Boolean类型

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

`Object类`中的`equals()`方法用来判断两个对象是否相等。如果当前对象与传入的对象相同，则返回true，否则返回false。

注意：这个`equals()`方法没有被final关键字修饰，也就是说可以被子类重写，比如说String类型里面重写了`equals()`方法，如下：

```java
public boolean equals(Object anObject) {
    //如果对象相等的话，直接返回true
    if (this == anObject) {
        return true;
    }
    /**
    	 * 如果对象不相同，则比较String中的value值是否相同。
     	 * 首先判断传入的对象是不是String类型的实例：
    	 * 如果是，将字符串转换成字符数组接着逐一比较数组里的字符是否相同，如果不同，直接返回false。
    	 * 遍历结束，则表示相同，返回true。
    	 * */
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

如果传入的对象和当前对象是同一个对象，返回true；如果传入的对象是String类型的实例，则判断传入对象的String和当前String是否相等，如果相等，则返回true，否则返回false。

`instanceof `关键字的作用：判断左边的对象是否是右边类的实例，例如 `objectA.instanceof(ClassB)`判断对象A是否是类B的实例，如果是则返回`true`，否则返回`false`。

### `clone()`方法

```java
protected native Object clone() throws CloneNotSupportedException;
```

返回当前实例的克隆对象。

### `toString()`方法

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

返回表示当前对象的字符串，字符串的形式是类名@`hashCode`的十六进制表示，比如`objectA`是`classB`的实例，`ObjectA`的`hashCode`为c,而c的十六进制为d,则调用`ObjectA`的`toString()`方法返回ClassB@d。推荐所有的子类都重写这个方法。

### `notify()`方法

```java
public final native void notify();
```

唤醒在该对象监视器(Monitor)里等待的一个线程。

### `notifyAll()`方法

```java
public final native void notifyAll();
```

唤醒当前对象监视器下的所有线程。

### `wait(long)`方法

```java
public final native void wait(long timeout) throws InterruptedException;
```

使当前线程等待直到另外一个线程调用等待线程的`notify()`方法或者`notifyAll()`方法，或者等待指定的时间。

### `wait(long,int)`方法

```java
public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}
```

和只有一个入参的`wait(long)`方法类似，但它允许更精细的时间控制，以纳秒进行实时的统计。

### `wait()`方法

```java
public final void wait() throws InterruptedException {
    wait(0);
}
```



### `finalize()`方法

```java
protected void finalize() throws Throwable { }
```

JAVA内存回收机制是通过根节点到当前对象之间有没有一条引用链指向来判断对象是否可以回收的。

该对象没有引用指向的时候，垃圾收集器在合适的时机调用该对象的`finalize()`方法对该对象进行回收。

参考资料

https://blog.csdn.net/benjaminlee1/article/details/72843713