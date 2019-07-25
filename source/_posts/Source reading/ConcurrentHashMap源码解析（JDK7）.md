---
title: ConcurrentHashMap源码解析(JDK7)
categories:
- 好好学习
tags:
  - JDK源码
date: 2018-10-23 21:03:46
---


# ConcurrentHashMap源码解析(JDK8)

![ConcurrentHashMap.png](https://i.loli.net/2018/10/23/5bced8c568df1.png)


<!-- more -->
## Constants (常量)

```java
    /**
     * The default initial capacity for this table,
     * used when not otherwise specified in a constructor.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 16;
```

默认的最初的容量为16

```java
    /**
     * The default load factor for this table, used when not
     * otherwise specified in a constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

默认的负载因子为0.75

```java
    /**
     * The default concurrency level for this table, used when not
     * otherwise specified in a constructor.
     */
    static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

默认的并发级别为16

```java
    /**
     * The maximum capacity, used if a higher value is implicitly
     * specified by either of the constructors with arguments.  MUST
     * be a power of two <= 1<<30 to ensure that entries are indexable
     * using ints.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;
```

最大容量为2的32次幂。

```java
    /**
     * The minimum capacity for per-segment tables.  Must be a power
     * of two, at least two to avoid immediate resizing on next use
     * after lazy construction.
     */
    static final int MIN_SEGMENT_TABLE_CAPACITY = 2;
```

最小的段数。

```java
    /**
     * The maximum number of segments to allow; used to bound
     * constructor arguments. Must be power of two less than 1 << 24.
     */
    static final int MAX_SEGMENTS = 1 << 16; // slightly conservative
```

略微保守的最大段数为2的16次幂。

```java
    /**
     * Number of unsynchronized retries in size and containsValue
     * methods before resorting to locking. This is used to avoid
     * unbounded retries if tables undergo continuous modification
     * which would make it impossible to obtain an accurate result.
     */
    static final int RETRIES_BEFORE_LOCK = 2;
```

## 字段(Fields)

```java
    /**
     * A randomizing value associated with this instance that is applied to
     * hash code of keys to make hash collisions harder to find.
     */
    private transient final int hashSeed = randomHashSeed(this);

    private static int randomHashSeed(ConcurrentHashMap instance) {
        if (sun.misc.VM.isBooted() && Holder.ALTERNATIVE_HASHING) {
            return sun.misc.Hashing.randomHashSeed(instance);
        }

        return 0;
    }
```

transient: 用transient关键字标记的成员变量不参与序列化过程。







