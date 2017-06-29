---
title: HashMap解析
date: 2017-06-28 22:54:53
category: summarize
tags: [java,HashMap]
---

### HashMap数据结构分析
进入HashMap的Class能看到如下部分（为什么length必须是2的倍数呢?)    
    
```java
    /**
     * An empty table instance to share when the table is not inflated.
     */
    static final Entry<?,?>[] EMPTY_TABLE = {};
    .  .  .
    

    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
    .  .  .
    
    
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
        .  .  .
    }

```      

由此我们可以看出HashMap由一个table数组组成，这个table数组中包含有Key，Value，Next的一个Entry。    
再次抽象一下：
HashTable总体由一个数组组成。而这些数组的每一个元素又是一个单向链表。
于是当我们put数据时，程序根据HashCode索引到具体的数组位置，得到一个Entry 链表，然后放入链表的首位。同理查询、删除也是如此。    

```java
 public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)   //HashMap与HashTable的区别点之一
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);//计算出对应的数组索引位置
        //遍历对比链表中的值
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            //如果key相同 则替换旧值
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        //key不存在 则直接添加至链表队尾
        modCount++;//修正hashMap容量
        addEntry(hash, key, value, i);
        return null;
    }
```      


### Hash索引
  大家注意看HashMap的put代码，出现了`int i = indexFor(hash, table.length);`而之后，利用返回结果取出了table中对应的Entry。
由此我们可知，该方法是根据HashCode的值计算对应的数组中的值。    

```java
    /**
     * Returns index for hash code h.
     */
    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }

```     
  方法中，传入的HashCode与table的长度有一个按位与的操作，从而计算出table数组中的索引。前面已经提到，操作HashMap中的元素时先获取位置
然后遍历链表，执行相应的操作。遍历链表此时显得尤为的耗时。所以，为了保证HashMap的高效，要求每个键上的链表长度尽可能的小即为1。[table长度为2时重复概率最小、且空间浪费最少](http://stackoverflow.com/questions/22935616/why-hash-method-in-hashmap)。我们再来看看构造函数。。。发现HashMap初始化时均需要指定table的长度，未指定的则默认为16。大家会发现HashMap初始化的时候，还有一`loadFactor`的参数
该参数的默认值为0.75f。这个参数有什么用呢？

### HashMap的长度调整
随着HashMap存储数据的增多，由于table[]的定长，索引重复率提高。所以为了提高执行效率，此时需要扩容！    
扩容需要重新数组长度，然后再将现在的数据重新分配进新的数组。想想就知道此时的效率是多么的低下，由此初始化一个合理的长度是多么的重要！
那么什么时候进行扩容操作呢？这时候就要用到上面提到的那个参数了。    
扩容及计算新HashMap的扩容阀值    
    
```java    
    /**
     * Inflates the table.
     */
    private void inflateTable(int toSize) {
        // Find a power of 2 >= toSize
        int capacity = roundUpToPowerOf2(toSize);

        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }

```      

当HashMap的size大于threhold时及对HashMap进行扩容操作（代码见*public void putAll(Map<? extends K, ? extends V> m)*
及*void addEntry(int hash, K key, V value, int bucketIndex)*）。threhold的最大值为2^30+1
那么扩容后的old value是怎么处理的呢？  

```java
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            //遍历所有的旧元素
            while(null != e) {
                Entry<K,V> next = e.next;
                //是否需要重新计算hash值
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                //获取新的索引
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```  

由以上代码我们可以看出，在HashMap扩容过后，会依次遍历旧元素，重新计算索引位置，这样的操作是相当费时的。所以在执行HashMap初始化时，如果存储容量已知，我们可以指定
存储容量，防止HashMap扩容。  
  
### [HashMap与LinkedHashMap,TreeMap的区别](http://www.geeksforgeeks.org/differences-treemap-hashmap-linkedhashmap-java/)
 
 LinkedHashMap是HashMap子类，保存了记录的插入顺序，
 在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.
 也可以在构造时用带参数，按照应用次数排序。相对于HashMap中的Entry LinkedHashMap中多了header和after构成链表记录插入顺序  
 
 TreeMap实现SortMap接口，能够把它保存的记录根据键排序,默认是按键值的升序排序，
 当用Iterator 遍历TreeMap时，得到的记录是排过序的。TreeMap中的key要求存储对像实现了Comparable接口即是可比较的
 
 

