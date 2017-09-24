---
layout:     post
title:      "Java集合（二）-ArrayList"
subtitle:   ""
date:       2017-09-24 03:00:00
author:     "zhang.xx"
header-img: "img/post-everydayguess.jpg"
catalog: true
tags:
    - Java进阶
---

> If more of us valued food and cheer and song above hoarded gold, it would be a merrier world. ——**John·Ronald·Reuel·Tolkien**

---
## 前言

List接口常用实现类的特点：  
* ArrayList 查询快，修改、插入、删除慢； 底层实现：数组；
* LinkedList 查询慢，修改、插入、删除快； 底层实现：链表；
* Vector 线程安全的，效率低；

## 概览
**ArrayList**：  
> public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable

ArrayList 实现了 List 接口，是顺序容器，即元素存放的数据与放进去的顺序相同，允许放入null元素，底层实现：**数组**  
> transient Object[] elementData;

每个 ArrayList 都有一个容量（capacity），表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。

ArrayList中的操作**不是线程安全的**！所以，建议在单线程中才使用ArrayList，如果需要多个线程并发访问，用户可以手动同步，也可以选择Vector或者CopyOnWriteArrayList。


## 源码解析(基于jdk1.8.0_91)

#### 构造方法
```java
  // 参数为容量大小
  public ArrayList(int initialCapacity) {
      if (initialCapacity > 0) {
          this.elementData = new Object[initialCapacity];
      } else if (initialCapacity == 0) {
          this.elementData = EMPTY_ELEMENTDATA;
      } else {
          throw new IllegalArgumentException("Illegal Capacity: "+
                                             initialCapacity);
      }
  }
  // 默认构造空数组
  public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
  }
  // 构造包含指定元素的list
  public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
  }
```
#### size() isEmpty()
```java
    private int size;
    public int size() {
        return size;
    }
    public boolean isEmpty() {
        return size == 0;
    }
```
#### add()
```java
  public boolean add(E e) {
      // 扩容检查
      ensureCapacityInternal(size + 1);  // Increments modCount!!
      elementData[size++] = e;
      return true;
  }
  // 指定位置添加元素
  public void add(int index, E element) {
      // 索引越界检查
      rangeCheckForAdd(index);

      ensureCapacityInternal(size + 1);  // Increments modCount!!
      // 空出index的位置插入element，并将index后的元素位移一个位置
      System.arraycopy(elementData, index, elementData, index + 1,
                       size - index);
      // index位置插入新元素
      elementData[index] = element;
      size++;
  }
```

**数组扩容**
```java
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); //原来的1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity); //扩展空间并复制
    }
```
示意图：  
![](http://owqdwc45t.bkt.clouddn.com/148526680168618_ArrayList_add.png)

#### addAll()
```java
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        // 将数组a添加到elementData的尾部
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
    // 在指定位置，增加一个集合元素
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        // 计算需要移动的长度（index之后的元素个数）
        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved); // 将数组index后的元素向右移动numNum个位置
        // 将要插入的集合元素复制到数组空出的位置中
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
```
#### get()
对数组的指定位置赋值
```java
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```
#### set()
会返回被替代的值
```java
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```
#### remove()

```java
    // 删除指定位置的元素
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved); // 数组复制，将index之后的元素往前移动一个位置
        // 显式的为最后一个位置赋null值
        // size减一
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
    // 删除第一个满足o.equals(elementData[index])的元素
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                // null用==比较
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                // 非null用equals比较
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```
有了垃圾收集器并不意味着一定不会有内存泄漏。对象能否被 GC 的依据是是否还有引用指向它，上面代码中如果不手动赋null值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。

增加和删除需要特别关心的就两个地方：1.数组扩容，2.数组复制，这两个操作都是极费效率的，最惨的情况下(添加到list第一个位置，删除list最后一个元素或删除list第一个索引位置的元素)时间复杂度可达O(n)。

#### trimToSize()
```Java
// 将底层数组的容量调整为当前实际元素的大小
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

#### contains(Object o)
```Java
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
    public int indexOf(Object o) {
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


## 总结与参考
ArrayList和LinkedList的区别
1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2. 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3. 对于新增和删除操作add和remove，LinkedList比较占优势，因为ArrayList要移动数据。

ArrayList和Vector的区别
1. Vector和ArrayList几乎是完全相同的,唯一的区别在于Vector是同步类(synchronized)，属于强同步类。因此开销就比ArrayList要大，访问要慢。正常情况下,大多数的Java程序员使用ArrayList而不是Vector,因为同步完全可以由程序员自己来控制。
2. Vector每次扩容请求其大小的2倍空间，而ArrayList是1.5倍。
3. Vector还有一个子类Stack. 这是干嘛用的？

参考了尚学堂高淇的集合视频15集，以及下面相关博客，代码示例均为手敲，jdk1.8下编译通过；

参考：
JDK1.8源码  
[Java集合干货系列-（一）ArrayList源码解析](http://tengj.top/2016/04/13/javajh1arraylist/#ArrayList定义)  
[JCFInternals-2-ArrayList](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/2-ArrayList.md)  


2017年9月-by zhang.xx
