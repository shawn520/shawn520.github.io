---
title: ArrayList 源码解析(JDK1.7)
categories:
- 好好学习
tags:
  - JDK源码
date: 2018-10-22 20:21:46
---

# ArrayList 源码解析(JDK1.7)



包含的方法如下图：

所有方法按照字母`a-z`排序  

[![ArrayList1.7.png](https://i.loli.net/2018/10/22/5bcd225c4ca4f.png)](https://i.loli.net/2018/10/22/5bcd225c4ca4f.png)

<!-- more -->

## 属性

### DEFAULT_CAPACITY

```java
    /**
     * 默认的初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;
```

list的默认初始容量。

### EMPTY_ELEMENTDATA

```java
    /**
     * Shared empty array instance used for empty instances.空的对象数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
```

空的数组对象。

### elementData

```java
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == EMPTY_ELEMENTDATA will be expanded to
     * DEFAULT_CAPACITY when the first element is added.
     */
    private transient Object[] elementData;
```

当插入第一个元素的时候，空的list的容量将会从0扩展到10.

### size

```
/**
     * The size of the ArrayList (the number of elements it contains).
     * 数组的大小
     * @serial
     */
    private int size;
```

数组中含有的元素的个数。

## 构造函数

### ArrayList(int initialCapacity)

```java
    /**
     * Constructs an empty list with the specified initial capacity.
     * 
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }
```

构造一个带有指定大小容量的list, 如果入参小于0的话则抛出`IllegalArgumentException`不合法的参数异常。

### ArrayList()

```java
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }
```

构造一个带有初始容量为10的空list。

### ArrayList(Collection<? extends E> c)

```java
    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```

构造一个list包含指定容器的元素，其中的元素顺序按照容器的遍历顺序。

### trimToSize()

```java
    /**
     * Trims the capacity of this <tt>ArrayList</tt> instance to be the
     * list's current size.  An application can use this operation to minimize
     * the storage of an <tt>ArrayList</tt> instance.
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = Arrays.copyOf(elementData, size);
        }
    }
```

将当前`ArrayList`的实例的容量调整为当前含有的元素的个数。

这个方法可以节省存储空间。

### ensureCapacity(int minCapacity)

```java
    /**
     * Increases the capacity of this <tt>ArrayList</tt> instance, if
     * necessary, to ensure that it can hold at least the number of elements
     * specified by the minimum capacity argument.
     *
     * @param   minCapacity   the desired minimum capacity 所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
    	//如果数组为空，minExpand容量取0，否则取默认值10
        int minExpand = (elementData != EMPTY_ELEMENTDATA)
            // any size if real element table
            ? 0
            // larger than default for empty table. It's already supposed to be
            // at default size.
            : DEFAULT_CAPACITY;
		//如果minCapacity大于minExpand,则设为最小容量
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```

### size()

```java
    /**
     * Returns the number of elements in this list.
     *
     * @return the number of elements in this list
     */
    public int size() {
        return size;
    }
```

返回当前list中所含元素的个数。

### isEmpty()

```java
    /**
     * Returns <tt>true</tt> if this list contains no elements.
     *
     * @return <tt>true</tt> if this list contains no elements
     */
    public boolean isEmpty() {
        return size == 0;
    }
```

作用：判断当前list是否为空。

如果list中所含元素个数为0的话，表示为空，返回true，否则返回false。

### contains(Object o)

```java
    /**
     * Returns <tt>true</tt> if this list contains the specified element.
     * More formally, returns <tt>true</tt> if and only if this list contains
     * at least one element <tt>e</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>.
     *
     * @param o element whose presence in this list is to be tested
     * @return <tt>true</tt> if this list contains the specified element
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```

作用：判断当前list是否包含指定元素。

如果包含则返回true，不包含则返回false。

### indexOf(Object o)

```java
    /**
     * Returns the index of the first occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the lowest index <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     */
    public int indexOf(Object o) {
        //如果对象为空的话
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

返回元素在list中的第一个索引。

### lastIndexOf(Object o)

```java
    /**
     * Returns the index of the last occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the highest index <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

返回数组list里面匹配的最后一个元素。

### clone()

```java
    /**
     * Returns a shallow copy of this <tt>ArrayList</tt> instance.  (The
     * elements themselves are not copied.)
     *
     * @return a clone of this <tt>ArrayList</tt> instance
     */
    public Object clone() {
        try {
            @SuppressWarnings("unchecked")
                ArrayList<E> v = (ArrayList<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0; //将修改次数计数器置位0
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError();
        }
    }
```

浅拷贝：只复制当前对象，对该对象内部的引用不能复制。

深拷贝：对对象内部的引用均复制，是创建一个新的实例，并复制实例。

返回当前数组list实例的浅拷贝(元素本身不复制)。

### toArray()

```java
    /**
     * Returns an array containing all of the elements in this list
     * in proper sequence (from first to last element).
     *
     * <p>The returned array will be "safe" in that no references to it are
     * maintained by this list.  (In other words, this method must allocate
     * a new array).  The caller is thus free to modify the returned array.
     *
     * <p>This method acts as bridge between array-based and collection-based
     * APIs.
     *
     * @return an array containing all of the elements in this list in
     *         proper sequence
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```

以合适的顺序（从第一个到最后一个元素）返回当前list中的所有元素到一个数组

返回一个数组，这个数组包含当前list的所有元素，并且以从第一个到最后一个元素的顺序。

### toArray(T[] a)

### get(int index)

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * @param  index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        rangeCheck(index);	//对index进行检查，是否超出了边界

        return elementData(index);
    }
```

返回指定位置上的元素。



