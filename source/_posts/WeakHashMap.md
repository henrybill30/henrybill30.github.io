---
title: WeakHashMap是什么？
date: 2021-09-15 11:07:55
tags: [Java, Map]
categories: Java学习
---
# WeakHashMap是什么？

# 1. 四大引用

在介绍WeakHashMap之前，必须先清楚Weak代表了什么？在java中，有四种对象的引用方式，俗称“强软弱虚”，分别对应强引用、软引用、弱引用和虚引用，这四种引用有不同的使用场景，理解它们是理解WeakHashMap的前提，四种引用的解释与使用场景如下表格所示：
<style>
    table th:nth-of-type(1){
    width: 20%;
    }
    table th:nth-of-type(2){
    width: 40%
    ;
    }
    table th:nth-of-type(3){
    width: 40%;
    }
</style>


| 引用名 | 解释 | 使用场景 |
|:-----:|:-----:|:-----:|
| 强引用 | 最普通的引用，只要对象的强引用一直存在，该对象就永远不会被JVM垃圾回收 | 只要不是特殊场景，都应该使用强引用 |
| 软引用 | 存在内存不足时，软引用的对象会被垃圾回收 | 适合做缓存，当内存足够可以拿到缓存，内存紧张时，直接回收 |
| 弱引用 | 无论内存足够与否，只要发生GC，就会被回收 | ThreadLocal、WeakHashMap |
| 虚引用 | 发生GC时，会回收虚引用对象，并且将回收通知记录在ReferenceQueue中 | 在NIO中，虚引用可以用来管理堆外内存 |

除了强引用外，其他三种引用方法的创建方法如下，

- 软引用

```java
SoftReference<T> softTObject = new SoftReference<T>(new T());
```

- 弱引用

```java
WeakReference<T> weakTObject = new WeakReference<T>(new T());
```

- 虚引用

```java
ReferenceQueue queue = new ReferenceQueue();
PhantomReference<T> phantomTObject = new PhantomReference<T>(new T(), queue);
```

特别注意创建虚引用时需要加一个ReferenceQueue对象

这里只是简单介绍四大引用，会在以后另起一篇文章详细介绍四大引用的原理和使用。

# 2. WeakHashMap

了解了四大引用后，再看WeakHashMap就会比较简单了，其实WeakHashMap就是加了弱引用的HashMap，其用法与HashMap几乎相同，唯一的区别就是：**Map的key是通过弱引用建立的。**虽然value没有使用弱引用，但是当key被回收时，value也会同时被清除。直接看源码中的解释：

> Hash table based implementation of the Map interface, with weak keys. An entry in a WeakHashMap will automatically be removed when its key is no longer in ordinary use. More precisely, the presence of a mapping for a given key will not prevent the key from being discarded by the garbage collector, that is, made finalizable, finalized, and then reclaimed. When a key has been discarded its entry is effectively removed from the map, so this class behaves somewhat differently from other Map implementations.
Both null values and the null key are supported. This class has performance characteristics similar to those of the HashMap class, and has the same efficiency parameters of initial capacity and load factor.
Like most collection classes, this class is not synchronized. A synchronized WeakHashMap may be constructed using the Collections.synchronizedMap method.

总结下来有以下几点：

- 实现了Map接口，有弱引用的key
- entry将被回收清除当key不被正常使用
- 支持null值key和null值value
- 线程不安全，想要线程安全时要使用Collections.synchronizedMap

核心代码，源码中大部分与HashMap实现相同，不同点在于Entry对象的构造上，Entry<K, V>对象继承了WeakReference类并实现了Map.Entry接口，如下所示。

```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;

        /**
         * Creates new entry.
         */
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }
}
```

当key被GC时，就会被放入ReferenceQueue中，每次get时根据ReferenceQueue中的值删除对应的value，从而实现对key和value的自动回收。删除的具体代码如下。

```java
private void expungeStaleEntries() {
        for (Object x; (x = queue.poll()) != null; ) {
            synchronized (queue) {
                @SuppressWarnings("unchecked")
                    Entry<K,V> e = (Entry<K,V>) x;
                int i = indexFor(e.hash, table.length);

                Entry<K,V> prev = table[i];
                Entry<K,V> p = prev;
                while (p != null) {
                    Entry<K,V> next = p.next;
                    if (p == e) {
                        if (prev == e)
                            table[i] = next;
                        else
                            prev.next = next;
                        // Must not null out e.next;
                        // stale entries may be in use by a HashIterator
                        e.value = null; // Help GC
                        size--;
                        break;
                    }
                    prev = p;
                    p = next;
                }
            }
        }
    }
```

在put方法中，可以看出当发生哈希冲突时，WeakHashMap使用的是拉链法解决。

```java
public V put(K key, V value) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int i = indexFor(h, tab.length);

        for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
            if (h == e.hash && eq(k, e.get())) {
                V oldValue = e.value;
                if (value != oldValue)
                    e.value = value;
                return oldValue;
            }
        }

        modCount++;
        Entry<K,V> e = tab[i];
        tab[i] = new Entry<>(k, value, queue, h, e);
        if (++size >= threshold)
            resize(tab.length * 2);
        return null;
    }
```

# 3. 使用场景

实际中，WeakHashMap主要会用作缓存，在这篇[文章](https://blog.csdn.net/kaka0509/article/details/73459419)中提到，tomcat中使用WeakHashMap做eden和longterm的分代缓存。

# 4. 实际使用

如下代码执行后，结果如下

```java
public static void main(String[] args) {
        WeakHashMap<Object ,String> hashMap = new WeakHashMap<>();
        Object a = new Object();
        Object b = new Object();
        Object c = new Object();
        hashMap.put(a, "a");
        hashMap.put(b, "b");
        hashMap.put(c, "c");
        System.out.println("a != null 时：map size = " + hashMap.size());
        System.gc();
        System.out.println("gc后：map size = " + hashMap.size());
        a = null;
        System.out.println("a = null 时：map size = " + hashMap.size());
        System.gc();
        System.out.println("gc后：map size = " + hashMap.size());
    }
```

![HashMap](Untitled.png)

换成HashMap后结果如下：

![WeakHashMap](Untitled1.png)

可以看到WeakHashMap与HashMap的区别