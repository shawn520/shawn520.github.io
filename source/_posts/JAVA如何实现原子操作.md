---
title: JAVA如何实现原子操作？
categories:
- 好好学习
tags:
  - Java并发
date: 2019-07-10 09:19:46
---


>java可以通过**锁**和**循环CAS**的方式实现原子操作。下面介绍循环CAS的方式。

<!-- more -->

## 循环CAS实现原子操作
循环CAS的**基本思路**就是通过循环执行CAS操作，直到执行成功跳出循环。
以下代码实现基于CAS线程安全的计数器方法safeCount()和非线程安全计数器count。
初始化100个线程，每个线程执行1w次计数,线程安全的结果应该是100w,而非线程安全的结果可能会小于这个数。

```java
public class Counter {
    private AtomicInteger atomicI = new AtomicInteger(0);
    private int i = 0;

    public static void main(String[] args) {
        final Counter cas = new Counter();
        List<Thread> threads = new ArrayList<>(600);
        long start = System.currentTimeMillis();

        //初始化100个线程，每个线程执行1w次计数
        for(int j=0; j<100; j++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0; i<10000; i++){
                        cas.safeCount();
                        cas.count();

                    }

                }
            });
            threads.add(thread);
        }

        for (Thread thread : threads) {
            thread.start();
        }

        //等待所有线程执行完成
        for(Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //非线程安全的结果:<100w
        System.out.println(cas.i);
        //线程安全的结果: 100w
        System.out.println(cas.atomicI.get());
        System.out.println(System.currentTimeMillis() - start);

    }

    /**
     * 使用CAS实现线程安全计数
     */
    private void safeCount() {
        //自旋CAS,直到成功跳出循环
        for(;;) {
            int i = atomicI.get();
            boolean suc = atomicI.compareAndSet(i, ++i);
            if (suc) {
                break;
            }
        }
    }

    /**
     * 非线程安全计数
     */
    private void count() {
        i++;
    }

}
```

## CAS实现原子操作的问题
循环CAS存在三大问题：ABA问题，循环时间长开销大，只能保证一个共享变量的原子操作。
## ABA问题
**什么是ABA问题？**
如果一个值原来是A，变成了B，又变成了A。那么CAS认为值没有发生变化，实际上是发生了变化的。

**解决思路**：在变量前面追加版本号
A -> B -> A就会变成1A -> 2B -> 3A

这里是完整的 [代码](
https://github.com/shawn520/algorithms/blob/master/src/others/concurent/Counter.java )。

参考资料：
方腾飞《Java并发编程的艺术》


