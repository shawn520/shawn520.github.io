---
title: Thread 源码阅读
categories:
- 好好学习
tags:
  - JDK源码
date: 2019-02-20 20:19:46
---

# Thread 源码阅读

![image](https://wx1.sinaimg.cn/large/b727c653ly1fyrcehtqfnj20a50hm0t1.jpg)

<!-- more -->

```java
public
class Thread implements Runnable {
}
```

所有Thread类实现了`Runnable`接口并实现了`run()`方法，为什么叫Runnable，而不是Running呢? 那是因为线程是可运行状态，并不表示此时此刻正在运行，线程的运行还需要处理器的调度。



## 构造方法

```java
public Thread(ThreadGroup group, Runnable target, String name,
long stackSize) {
init(group, target, name, stackSize);
}
```

## 方法

### currentThread()

返回当前执行线程对象的一个引用。

```java
/**
* Returns a reference to the currently executing thread object.
*
* @return  the currently executing thread.
*/
public static native Thread currentThread();
```

### sleep(long millis) 

使当前执行的线程睡眠指定的 毫秒。当传入的时间millis为负数时，抛出`IllegalArgumentException`异常。

```java
public static native void sleep(long millis) throws InterruptedException;
```

### start() 

使当前线程开始执行。注意：一个已经执行的线程是不能执行`start() `方法的。

```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```

### void interrupt()

`interrupt()`中断当前线程的执行。

```java
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}
```

## boolean isInterrupted()

检查当前线程是不是被中断。

```java
public boolean isInterrupted() {
        return isInterrupted(false);
    }
```

## 线程的优先级

```java

public final static int MIN_PRIORITY = 1;

public final static int NORM_PRIORITY = 5;

public final static int MAX_PRIORITY = 10;
```

线程的优先级分为10级，最小优先级为1，最大优先级为10,普通优先级为5.



## setDaemon(boolean on)

如果on = true，则设置当前线程为守护线程。

注意：这个方法必须在线程启动前调用。

```java
public final void setDaemon(boolean on) {
    checkAccess();
    if (isAlive()) {
        throw new IllegalThreadStateException();
    }
    daemon = on;
}
```

