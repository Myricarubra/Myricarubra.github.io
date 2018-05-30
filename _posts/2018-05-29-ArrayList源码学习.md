---
layout: post
title: ArrayList源码学习
---

###1.概述：

&emsp;&emsp;ArrayList是基于数组实现的，是一个动态数组，其容量能自动增长，类似于C语言中的动态申请内存，动态增长内存。

&emsp;&emsp;ArrayList不是线程安全的，只能用在单线程环境下，多线程环境下可以考虑用Collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类。 

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{}
```

&emsp;&emsp;ArrayList继承自AbstractList，实现了List接口，Serializable接口，因此它支持序列化，能够通过序列化传输，实现了RandomAccess接口，支持快速随机访问，实际上就是通过下标序号进行快速访问，实现了Cloneable接口，可拷贝。

### 2.实现：

1. 属性：

```java
private static final long serialVersionUID = 8683452581122892189L; // 版本号
private static final int DEFAULT_CAPACITY = 10; // 默认容量
private static final Object[] EMPTY_ELEMENTDATA = {}; // 空object数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};  // 默认容量空object数组
transient Object[] elementData; // 元素数组 transient关键字修饰的属性不进行序列化
private int size; // 元素大小
```

2. 构造方法：

```java
// 容量大小参数的构造函数
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity]; // 初始化一个object数组，大小为传入参数
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA; // 参数=0时，初始化空的object数组
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);  // 参数<0,抛出参数不合法异常
    }
}

// 空参数
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; // 初始化一个默认容量空object数组
}

// Collection对象
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();  // Collection集合转化成数组
    if ((size = elementData.length) != 0) { // 参数是非空集合
        if (elementData.getClass() != Object[].class) // 是否转化成了object数组
            elementData = Arrays.copyOf(elementData, size, Object[].class); // 不为object数组，直接进行复制
    } else {
        this.elementData = EMPTY_ELEMENTDATA; // 非空集合，元素数组设置为空
    }
}
```

3. 核心方法：

```java
// add(E e)
// 在使用add()添加元素时，首先确保容量够用
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 元素size+1，执行ensureCapacityInternal方法
    elementData[size++] = e; // 执行完一系列扩容操作后 将元素添加到最后一个位置
    return true; // 返回true
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) { // 元素数组为空
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); // 比较默认容量和传入的size+1大小，选大的赋值minCapacity
    }
    ensureExplicitCapacity(minCapacity); // 使用minCapacity执行ensureExplicitCapacity方法
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++; // 结构性修改+1

    if (minCapacity - elementData.length > 0) // 传入参数大于元素数组参数
        grow(minCapacity); // 执行grow方法，进行扩容
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length; // 旧容量
    int newCapacity = oldCapacity + (oldCapacity >> 1);  // 新容量=1.5旧容量
    if (newCapacity - minCapacity < 0) // 1.5旧容量还不够用
        newCapacity = minCapacity; 		// 新容量=传入的参数
    if (newCapacity - MAX_ARRAY_SIZE > 0) // 新容量-最大数组容量>0
        newCapacity = hugeCapacity(minCapacity); // 执行hugeCapacity方法传入参数，并赋值给新容量
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity); // 将旧元素数组赋值并扩容
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; // 最大数组容量=最大整数-8

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE; // 返回一个最大的值
}

// indexOf(Object o)
// 查找元素
public int indexOf(Object o) {
    if (o == null) { // 待查找元素为空
        for (int i = 0; i < size; i++) // 遍历元素数组
            if (elementData[i]==null) // 找到null元素的角标
                return i; // 返回
    } else {
        for (int i = 0; i < size; i++) // 遍历元素数组
            if (o.equals(elementData[i])) // 找到元素相等的角标
                return i; // 返回
    }
    return -1; // 没有找到返回-1
}

// lastIndexOf(Object o)
// 从后查找元素
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

// get(int index)
	public E get(int index) {
    rangeCheck(index); // 进行角标检查

    return elementData(index); // 返回角标对应的元素
}

private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

// set(int index,E element)
public E set(int index, E element) {
    rangeCheck(index); // 进行角标检查

    E oldValue = elementData(index); // 数组下标对应的元素
    elementData[index] = element; // 这个下标赋值新元素
    return oldValue; // 返回旧元素
}

// remove(int index)
public E remove(int index) {
    rangeCheck(index); // 下标越界检查

    modCount++; // 结构修改+1
    E oldValue = elementData(index); // 移除的元素

    int numMoved = size - index - 1; // 需要移动的元素个数
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved); // 数组复制
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

### 3.总结

&emsp;&emsp;ArrayList数据结构是数组，数组元素是Object类，可以存放所有类型数据。ArrayList有其特殊的应用场景，与LinkedList相对应。其优点是随机读取，缺点是插入元素时需要移动大量元素，效率不太高。至此，ArrayList的源码分析就到这里，总体来说，ArrayList的底层还是很简单的。