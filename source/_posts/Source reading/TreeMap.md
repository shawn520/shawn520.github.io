---
title: TreeMap源码阅读
categories:
- 好好学习
tags:
  - JDK源码
date: 2019-07-18 10:08:46
---

想学习`ConcurrentHashMap`源码，先阅读理解`TreeMap`源码，顺便复习一下红黑树, 提升一下功力。

为后续阅读`ConcurrentHashMap`源码打基础。

![](https://raw.githubusercontent.com/shawn520/storage/master/pictures/note/java/map-treeMap.png)

<!-- more -->

## 前言

基于红黑树的TreeMap增删该查操作最坏时间复杂度为O(lgn)。实现了NavigableMap接口，保证key有序。



### 红黑树特性

1. 根节点必须红色
2. 节点必须是红色或者黑色
3. **不能有两个连续的红色节点**
4. Nil节点为黑色
5. 根节点到Nil节点的所有路径，黑色节点个数相同。



## 源码

### 红黑树数据结构定义

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;	// 指向父节点
    boolean color = BLACK;	// 默认黑色

    /**
     * 初始化节点
     */
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }
}
```

### 类名和属性

`TreeMap `继承`AbstractMap`, 实现`NavigableMap`接口，`NavigableMap`接口继承`SortedMap`, key有序。类图如下所示。红色表示接口，蓝色表示抽象类，绿色表示线程安全。

![](https://raw.githubusercontent.com/shawn520/storage/master/pictures/note/java/map-treeMap.png)

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable{
    /**
     * comparator用来维持treeMap有序
     */
    private final Comparator<? super K> comparator;

    private transient Entry<K,V> root;

    /**
     * 树中所含节点的个数
     */
    private transient int size = 0;

    /**
     * 记录树结构修改的次数
     */
    private transient int modCount = 0;
}
    
```

### 插入操作

插入操作分为两步：

1. 根据规则寻找插入位置。`put()`
2. 插入之后，如果不满足红黑树的约束条件`fixAfterInsertion(e)`，则重新着色后者通过左旋右旋等操作是满足红黑树的约束条件。



#### 根据规则寻找插入位置

```java
/**
 * 向map中插入指定的键值对，如果key存在，则替换和返回旧的value。
 * 
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException 如果key为null抛出空指针异常
 */
public V put(K key, V value) {
    Entry<K,V> t = root; // t 指向当前节点，开始指向根节点。
    // 如果根节点不存在，则插入节点为根节点。
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        // 因为是根节点，所以parent为null
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // 寻找插入位置
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            // parent指向当前节点
            parent = t;
            cmp = cpr.compare(key, t.key);
            // 小于当前节点则left
            if (cmp < 0)
                t = t.left;
            // 大于当前节点则right
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value); //等于则覆盖，并返回原value
            // 直到找到合适位置
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 初始化entry
    Entry<K,V> e = new Entry<>(key, value, parent);
    // 插入
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    // 第二步，调整使满足红黑树特性。
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

#### 调整使满足红黑树特性

插入节点的父节点为红色时，不满足红黑树特性第三条，需要进行调整。

```java
private void fixAfterInsertion(Entry<K,V> x) {
    // 新插入的节点默认为红色
    x.color = RED;

    // 插入节点的父节点为红色时，不满足红黑树特性第三条，需要调整
    while (x != null && x != root && x.parent.color == RED) {
        // 父节点是祖父节点的左孩子
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            // y指向大（右）叔节点
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            // 右叔节点为红色的话，进行局部颜色调整，递归向上，最坏时间复杂度O(lgn)
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            // 如果右叔为黑色的话
            } else {
                // 且如果x是父节点的右孩子，左旋调整为左孩子
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x); // 左旋
                }
                // 父节点设置为黑色，祖父节点设置为红色，并对祖父节点右旋操作
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        // 父节点是祖父节点的右孩子
        } else {
            // y指向小（左）叔
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            // 如果左叔为红色，设置父节点和左叔为黑色，祖父节点为红色，x指向祖父节点，一路向上。
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
                // 如果左叔为黑色
            } else {
                // 如果x为左孩子，则右旋转为右孩子情况
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                // 父节点设置成黑色，祖父设置成红色，且祖父节点左旋
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 根节点永远使黑色
    root.color = BLACK;
}
```

#### 左旋

左旋、右旋的目的是通过不断旋转是树达到平衡，提升二叉树的使用效率。

左旋：以某个节点为中心，将它下沉到当前节点的左孩子节点位置，让当前右孩子节点作为局部根节点。

右旋和左旋类似。此处以左旋为例。

```java
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> r = p.right;
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        r.left = p;
        p.parent = r;
    }
}
```

## 参考资料

码出高效 java开发手册

