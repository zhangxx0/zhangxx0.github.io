---
layout:     post
title:      "Java集合（三）-LinkedList"
subtitle:   ""
date:       2017-09-24 04:00:00
author:     "zhang.xx"
header-img: "img/post-everydayguess.jpg"
catalog: true
tags:
    - Java进阶
---

> If more of us valued food and cheer and song above hoarded gold, it would be a merrier world. ——**John·Ronald·Reuel·Tolkien**

---

## 前言
LinkedList是List接口链表的实现。基于链表实现的方式使得LinkedList在插入和删除时更优于ArrayList，而随机访问则比ArrayList逊色些。  
LinkedList 同时实现了 List 接口和 Deque 接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（Queue），同时又可以看作一个栈（Stack）

## 概览
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{}
```
LinkedList 底层通过**双向链表**实现，双向链表的每个节点用内部类 Node 表示。LinkedList 通过first和last引用分别指向链表的第一个和最后一个元素。注意这里没有所谓的哑元，当链表为空的时候first和last都指向null。

LinkedList 的实现方式决定了所有跟下标相关的操作都是线性时间，而在首段或者末尾删除元素只需要常数时间。为追求效率 LinkedList 没有实现同步（synchronized），如果需要多个线程并发访问，可以先采用Collections.synchronizedList()方法对其进行包装。

## 源码解析(基于jdk1.8.0_91)

#### Node()
Node内部类
```java
  private static class Node<E> {
      E item;
      Node<E> next;
      Node<E> prev;

      Node(Node<E> prev, E element, Node<E> next) {
          this.item = element;
          this.next = next;
          this.prev = prev;
      }
  }
```

#### 构造函数
```java
  public LinkedList() {
  }
  public LinkedList(Collection<? extends E> c) {
      this();
      addAll(c);
  }
```

#### add()
**改变前后的互相指向关系**
```java
    // 在 LinkedList 的末尾插入元素
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    // 在指定下表处插入元素
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            // 添加到末尾
            linkLast(element);
        else
            // 添加到node(index)前面
            linkBefore(element, node(index));
    }

    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode; //原来链表为空，这是插入的第一个元素
        else
            l.next = newNode;
        size++;
        modCount++;
    }    
    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null) //插入位置为0
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);
        // 根据条件 index < (size >> 1) 判断查找的方向，提高效率
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

![](https://gitee.com/zhangxx0/blog_image/raw/master/java/148526704726154_LinkedList_add.png)

#### remove()
remove()方法也有两个版本，一个是删除跟指定元素相等的第一个元素remove(Object o)，另一个是删除指定下标处的元素remove(int index)。

```java
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }

    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) { //删除的是第一个元素
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) { //删除的是最后一个元素
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;  //let GC work
        size--;
        modCount++;
        return element;
    }
```
两个删除操作都要 1. 先找到要删除元素的引用，2. 修改相关引用，完成删除操作。在寻找被删元素引用的时候remove(Object o)调用的是元素的equals方法，而remove(int index)使用的是下标计数，两种方式都是线性时间复杂度。在步骤 2 中，两个revome()方法都是通过unlink(Node<E> x)方法完成的。这里需要考虑删除元素是第一个或者最后一个时的边界情况。

![](https://gitee.com/zhangxx0/blog_image/raw/master/java/148526706581863_LinkedList_remove.png)

#### get()
```java
  public E get(int index) {
      checkElementIndex(index);
      return node(index).item;
  }
```

#### set()
先通过node(int index)找到对应下表元素的引用，然后修改Node中item的值

```java
  public E set(int index, E element) {
      checkElementIndex(index);
      Node<E> x = node(index);
      E oldVal = x.item;
      x.item = element;
      return oldVal;
  }
```


## 总结与参考

**ArrayList使用最普通的for循环遍历比较快，LinkedList使用foreach循环比较快。**  
使用各自遍历效率最高的方式，ArrayList的遍历效率会比LinkedList的遍历效率高一些

有些说法认为LinkedList做插入和删除更快，这种说法其实是不准确的：  
LinkedList做插入、删除的时候，慢在寻址，快在只需要改变前后Entry的引用地址  
ArrayList做插入、删除的时候，慢在数组元素的批量copy，快在寻址  

如果待插入、删除的元素是在数据结构的前半段尤其是非常靠前的位置的时候，LinkedList的效率将大大快过ArrayList，因为ArrayList将批量copy大量的元素；越往后，对于LinkedList来说，因为它是双向链表，所以在第2个元素后面插入一个数据和在倒数第2个元素后面插入一个元素在效率上基本没有差别，但是ArrayList由于要批量copy的元素越来越少，操作速度必然追上乃至超过LinkedList

从这个分析看出，如果你十分确定你插入、删除的元素是在前半段，那么就使用LinkedList；如果你十分确定你删除、删除的元素在比较靠后的位置，那么可以考虑使用ArrayList。如果你不能确定你要做的插入、删除是在哪儿呢？那还是建议你使用LinkedList吧，因为一来LinkedList整体插入、删除的执行效率比较稳定，没有ArrayList这种越往后越快的情况；二来插入元素的时候，弄得不好ArrayList就要进行一次扩容，记住，ArrayList底层数组扩容是一个既消耗时间又消耗空间的操作。



参考了尚学堂高淇的集合视频15集，以及下面相关博客，代码示例均为手敲，jdk1.8下编译通过；

参考：  
JDK1.8源码
[Java集合干货系列-（二）LinkedList源码解析](http://tengj.top/2016/04/13/javajh2linklist/)  
[JCFInternals-3-LinkedList](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/3-LinkedList.md)  


2017年9月-by zhang.xx
