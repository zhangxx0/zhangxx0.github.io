---
layout:     post
title:      "Java集合（四）-HashMap & HashSet"
subtitle:   ""
date:       2017-09-24 05:00:00
author:     "zhang.xx"
header-img: "img/post-everydayguess.jpg"
catalog: true
tags:
    - Java进阶
---

> If more of us valued food and cheer and song above hoarded gold, it would be a merrier world. ——**John·Ronald·Reuel·Tolkien**

---

## 前言
HashMap 实现了 Map 接口，即允许放入key为null的元素，也允许插入value为null的元素；除该类未实现同步外，其余跟Hashtable大致相同；此外，HashMap中的映射不是有序的。  
跟 TreeMap 不同，该容器不保证元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散，因此不同时间迭代同一个 HashMap 的顺序可能会不同。  

## 概览
1.7版本的JDK：  
HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。  
根据对冲突的处理方式不同，哈希表有两种实现方式，一种开放地址方式（Open addressing），另一种是冲突链表方式（Separate chaining with linked lists）。Java HashMap 采用的是 **冲突链表方式**。  
HashMap结构图：  
![](http://owqdwc45t.bkt.clouddn.com/148526920663924_HashMap_base.png)  
从上图容易看出，如果选择合适的哈希函数，put()和get()方法可以在常数时间内完成。但在对 HashMap 进行迭代时，需要遍历整个 table 以及后面跟的冲突链表。因此对于迭代比较频繁的场景，不宜将 HashMap 的初始大小设的过大。

有两个参数可以影响 HashMap 的性能：初始容量（inital capacity）和负载系数（load factor）。初始容量指定了初始table的大小，负载系数用来指定自动扩容的临界值。当entry的数量超过capacity*load_factor时，容器将自动扩容并重新哈希。对于插入元素较多的场景，将初始容量设大可以减少重新哈希的次数。  

将对象放入到 HashMap 或 HashSet 中时，有两个方法需要特别关心：hashCode()和equals()。
**hashCode()方法决定了对象会被放到哪个bucket里，当多个对象的哈希值冲突时，equals()方法决定了这些对象是否是 “同一个对象”。**
所以，如果要将自定义的对象放入到HashMap或HashSet中，需要 @OverridehashCode()和equals()方法。

浅显易懂的说明：  
> 1.hashcode是用来查找的，如果你学过数据结构就应该知道，在查找和排序这一章有
> 例如内存中有这样的位置
> 0 1 2 3 4 5 6 7
> 而我有个类，这个类有个字段叫ID,我要把这个类存放在以上8个位置之一，如果不用hashcode而任意存放，那么当查找时就需要到这八个位置里挨个去找，或者用二分法一类的算法。
> 但如果用hashcode那就会使效率提高很多。
> 我们这个类中有个字段叫ID,那么我们就定义我们的hashcode为ID％8，然后把我们的类存放在取得得余数那个位置。比如我们的ID为9，9除8的余数为1，那么我们就把该类存在1这个位置，如果ID是13，求得的余数是5，那么我们就把该类放在5这个位置。这样，以后在查找该类时就可以通过ID除 > 8求余数直接找到存放的位置了。
> 2.但是如果两个类有相同的hashcode怎么办那（我们假设上面的类的ID不是唯一的），例如9除以8和17除以8的余数都是1，那么这是不是合法的，回答是：可以这样。那么如何判断呢？在这个时候就需要定义 equals了。
> 也就是说，我们先通过 hashcode来判断两个类是否存放某个桶里，但这个桶里可能有很多类，那么我们就需要再通过 equals 来在这个桶里找到我们要的类。
> 那么。重写了equals()，为什么还要重写hashCode()呢？
> 想想，你要在一个桶里找东西，你必须先要找到这个桶啊，你不通过重写hashcode()来找到桶，光重写equals()有什么用啊

1.8版本的JDK：   
- (1)JDK 1.8 以前 HashMap 的实现是 数组+链表，即使哈希函数取得再好，也很难达到元素百分百均匀分布。
- (2)当 HashMap 中有大量的元素都存放到同一个桶中时，这个桶下有一条长长的链表，这个时候 HashMap 就相当于一个单链表，假如单链表有 n 个元素，遍历的时间复杂度就是 O(n)，完全失去了它的优势。
- (3)针对这种情况，JDK 1.8 中引入了红黑树（查找时间复杂度为 O(logn)）来优化这个问题  

HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现  
![](http://owqdwc45t.bkt.clouddn.com/hashMap%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84%E5%9B%BE.png)

HashMap通常作为桶式哈希表，当桶变得很大的时候就转化为树结点，和TreeMap中比较类似，一般达到过量数据的时机比较少，所以在桶式哈希表中会尽量推迟树形结点的检测  

对于红黑树，不是很熟悉，需进一步的学习完善。


HashMap定义：
```Java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {}
```



## 源码解析(基于jdk1.8.0_91)

#### Node<K,V>
基本哈希容器结点Node（实现Map.Entry接口）
```Java
transient Node<K,V>[] table;

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next; // 下一个节点

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        // Entry对象的hashCode()由关键字key的hashCode()与值value的hashCode()做异或运算
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    // 对象相同或同类型且key-value均相同，则返回true
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

#### get()
返回指定关键字的值value，没有则返回null  
```java
  public V get(Object key) {
      Node<K,V> e;
      return (e = getNode(hash(key), key)) == null ? null : e.value;
  }

  // 核心方法
  final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node 始终检查第一个结点
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 1.7 版本使用的是for循环 依次遍历冲突链表中的每个entry
                // 依据equals()方法判断是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### put()
将指定关键字和指定value关联在一起 如果Map中之前就已经包含该关键字，则新值替换旧值
```java
  public V put(K key, V value) {
      return putVal(hash(key), key, value, false, true);
  }

  //
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) // 如果table数组尚未创建（第一次调用put），则新建table数组
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null); // table[i]中没有结点则创建新节点
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k)))) // 如果p=table[i]的关键字与给定关键字key相同，则替换旧值
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value); // 如果结点类型是TreeNode，则向红黑树中插入新节点
        else {
            // 遍历链表，查找给定关键字
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // static final int TREEIFY_THRESHOLD = 8; 如果大于8，则使用红黑树来替换链表，从而提高速度
                    // 创建新节点后若超出树形化阈值，则转换为树形存储
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        // 树形化
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) // 如果找到关键字相同的结点
                    break;
                p = e;
            }
        }
        // 相同的key，若找到则使用新的value替换掉原来的oldValue并返回oldValue
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize(); // 如果超出重构阈值，需要重新分配空间
    afterNodeInsertion(evict);
    return null;
}
```
树形化方法：  
```java
  // 将桶内所有的 链表节点 替换成 红黑树节点
  final void treeifyBin(Node<K,V>[] tab, int hash) {
      int n, index; Node<K,V> e;
      if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
          resize();
      else if ((e = tab[index = (n - 1) & hash]) != null) {
          TreeNode<K,V> hd = null, tl = null;
          do {
              TreeNode<K,V> p = replacementTreeNode(e, null);
              if (tl == null)
                  hd = p;
              else {
                  p.prev = tl;
                  tl.next = p;
              }
              tl = p;
          } while ((e = e.next) != null);
          if ((tab[index] = hd) != null)
              hd.treeify(tab);
      }
  }
```

#### remove()
如果存在就删除关键字对应的映射  
```java
  public V remove(Object key) {
      Node<K,V> e;
      return (e = removeNode(hash(key), key, null, false, true)) == null ?
          null : e.value;
  }

  final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable); // 删除树中的一个结点
            else if (node == p)
                tab[index] = node.next; // 删除容器中第一个结点
            else
                p.next = node.next; // 删除链表中的某一个结点  
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```


## HashSet
HashSet 是对 HashMap 的简单包装，对 HashSet 的函数调用都会转换成合适的 HashMap 方法，源码行数354。
```java
  public class HashSet<E>
      extends AbstractSet<E>
      implements Set<E>, Cloneable, java.io.Serializable
  {
    // Dummy value to associate with an Object in the backing Map
    // 一个Object
    private static final Object PRESENT = new Object();

    public HashSet() {
      map = new HashMap<>();
    }

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
  }
```

## 1.8的效率提升
待完成；

## 总结与参考
临写博客，才发现jdk1.8的HashMap改动不小，导致自己准备不足，因此还需要继续完善这篇博客。

参考：

JDK1.7：  
[Java集合干货系列-（三）HashMap源码解析](http://tengj.top/2016/04/15/javajh3hashmap/#)
[JCFInternals-6-HashSet and HashMap](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/6-HashSet%20and%20HashMap.md)
JDK1.8：  
[{美团技术团队}Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)
[深入分析hashmap](http://blog.csdn.net/lianhuazy167/article/details/66967698)  
[从源码理解HashMap](http://blog.csdn.net/ymrfzr/article/details/51244052)  


2017年9月-by zhang.xx
