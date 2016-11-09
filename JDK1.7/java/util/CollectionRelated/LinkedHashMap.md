* LinkedHashMap类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
   &nbsp;&nbsp; Map 接口的哈希表和链接列表实现，具有可预知的迭代顺序。此实现与 HashMap 的不同之处在于，后者维护着一个运行于所有条目的双重链接列表。此链接列表定义了迭代顺序，该迭代顺序通常就是将键插入到映射中的顺序（插入顺序）。注意：如果在映射中重新插入键，则插入顺序不受影响（如果在调用 m.put(k, v) 前 m.containsKey(k) 返回了 true，则调用时会将键 k 重新插入到映射 m 中）    
   &nbsp;&nbsp; LinkedHashMap原理图：<http://www.cnblogs.com/chenpi/p/5294077.html>
   <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_LinkedHashMap.jpg" /></p>
   <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_LinkedHashMap2.jpg" /></p>
   
   &nbsp;&nbsp; 此实现可以让客户避免未指定的、由 HashMap（及 Hashtable）所提供的通常为杂乱无章的排序工作，同时无需增加与 TreeMap 相关的成本。使用它可以生成一个与原来顺序相同的映射副本，而与原映射的实现无关：    
   &nbsp;&nbsp;   void foo(Map m) {      
   &nbsp;&nbsp; &nbsp;&nbsp;        Map copy = new LinkedHashMap(m);     
   &nbsp;&nbsp; &nbsp;&nbsp;       ...      
   &nbsp;&nbsp;   }
 
  &nbsp;&nbsp; 如果模块通过输入得到一个映射，复制这个映射，然后返回由此副本确定其顺序的结果，这种情况下这项技术特别有用（客户通常期望返回的内容与其出现的顺序相同）。提供特殊的构造方法来创建链接哈希映射，该哈希映射的迭代顺序就是最后访问其条目的顺序，从近期访问最少到近期访问最多的顺序（访问顺序）。这种映射很适合构建 LRU 缓存。调用 put 或 get 方法将会访问相应的条目（假定调用完成后它还存在）。putAll 方法以指定映射的条目集迭代器提供的键-值映射关系的顺序，为指定映射的每个映射关系生成一个条目访问。任何其他方法均不生成条目访问。特别是，collection 视图上的操作不影响底层映射的迭代顺序。

  &nbsp;&nbsp; 可以重写 removeEldestEntry(Map.Entry) 方法来实施策略，以便在将新映射关系添加到映射时自动移除旧的映射关系。

  &nbsp;&nbsp; 此类提供所有可选的 Map 操作，并且允许 null 元素。与 HashMap 一样，它可以为基本操作（add、contains 和 remove）提供稳定的性能，假定哈希函数将元素正确分布到桶中。由于增加了维护链接列表的开支，其性能很可能比 HashMap 稍逊一筹，不过这一点例外：LinkedHashMap 的 collection 视图迭代所需时间与映射的大小成比例。HashMap 迭代时间很可能开支较大，因为它所需要的时间与其容量成比例。

  &nbsp;&nbsp; 链接的哈希映射具有两个影响其性能的参数：初始容量和加载因子。它们的定义与 HashMap 极其相似。要注意：为初始容量选择非常高的值对此类的影响比对 HashMap 要小，因为此类的迭代时间不受容量的影响。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问链接的哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射的意外的非同步访问：Map m = Collections.synchronizedMap(new LinkedHashMap(...));
  
  &nbsp;&nbsp; 结构修改是指添加或删除一个或多个映射关系，或者在按访问顺序链接的哈希映射中影响迭代顺序的任何操作。在按插入顺序链接的哈希映射中，仅更改与映射中已包含键关联的值不是结构修改。 在按访问顺序链接的哈希映射中，仅利用 get 查询映射不是结构修改。Collection（由此类的所有 collection 视图方法所返回）的 iterator 方法返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器自身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒将来不确定的时间任意发生不确定行为的风险。

  &nbsp;&nbsp; 注意：迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的方式是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class LinkedHashMap<K,V> extends HashMap<K,V>
    implements Map<K,V> {
    private static final long serialVersionUID = 3801124242820219131L;
    
    //双链表的头结点
    private transient Entry<K,V> header;
    
    //迭代排序模式：true，实际存储顺序；false，插入顺序。
    private final boolean accessOrder;
    
    //构造一个带指定初始容量和加载因子的空插入顺序LinkedHashMap实例。
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
    
    //构造一个带指定初始容量和默认加载因子(0.75)的空插入顺序LinkedHashMap实例。
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
    
    //构造一个带默认初始容量(16)和加载因子(0.75)的空插入顺序LinkedHashMap实例。
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }
    
    //构造一个映射关系与指定映射相同的插入顺序LinkedHashMap实例。
    //所创建的LinkedHashMap实例具有默认的加载因子(0.75)和足以容纳指定映射中映射关系的初始容量。
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super(m);
        accessOrder = false;
    }
    
    //构造一个带指定初始容量、加载因子和排序模式的空LinkedHashMap实例。
    public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
    
    //在插入元素前初始化双链表
    @Override
    void init() {
        header = new Entry<>(-1, null, null, null);
        header.before = header.after = header;
    }
    
    //rehash，将所有条目转移到新的表数组，超类调整，为了更快的迭代链表，以提升性能.
    @Override
    void transfer(HashMap.Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e = header.after; e != header; e = e.after) {
            if (rehash)
                e.hash = (e.key == null) ? 0 : hash(e.key);
            int index = indexFor(e.hash, newCapacity);
            e.next = newTable[index];
            newTable[index] = e;
        }
    }
    
    //如果此映射将一个或多个键映射到指定值，则返回true。
    public boolean containsValue(Object value) {
        // 利用更快的迭代器
        if (value==null) {
            for (Entry e = header.after; e != header; e = e.after)
                if (e.value==null)
                    return true;
        } else {
            for (Entry e = header.after; e != header; e = e.after)
                if (value.equals(e.value))
                    return true;
        }
        return false;
    }
    
    //返回此映射到指定键的值。如果此映射中没有该键的映射关系，则返回null 。
    //更确切地讲，如果此映射包含满足(key==null ? k==null : key.equals(k))的从键k到值v的映射关系，
    //则此方法返回v；否则，返回 null。（最多只能有一个这样的映射关系。）
    //返回 null 值并不一定表明此映射不包含该键的映射关系；
    //也可能此映射将该键显式地映射为 null。可使用 containsKey 操作来区分这两种情况。
    public V get(Object key) {
        Entry<K,V> e = (Entry<K,V>)getEntry(key);
        if (e == null)
            return null;
        e.recordAccess(this);
        return e.value;
    }
    
    //从该映射中移除所有映射关系。此调用返回后映射将为空。
    public void clear() {
        super.clear();
        header.before = header.after = header;
    }
    
    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // 用于迭代双链接列表的字段
        Entry<K,V> before, after;

        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }

        //从链接列表中删除此项
        private void remove() {
            before.after = after;
            after.before = before;
        }

        // 在列表中指定的现有条目之前插入此项
        private void addBefore(Entry<K,V> existingEntry) {
            after  = existingEntry;
            before = existingEntry.before;
            before.after = this;
            after.before = this;
        }

        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            if (lm.accessOrder) {
                lm.modCount++;
                remove();
                addBefore(lm.header);
            }
        }

        void recordRemoval(HashMap<K,V> m) {
            remove();
        }
    }
    
    private abstract class LinkedHashIterator<T> implements Iterator<T> {...}
    private class KeyIterator extends LinkedHashIterator<K> {...}
    private class ValueIterator extends LinkedHashIterator<V> {...}
    private class EntryIterator extends LinkedHashIterator<Map.Entry<K,V>> {...}
    
    Iterator<K> newKeyIterator() {...}
    Iterator<V> newValueIterator() {...}
    Iterator<Map.Entry<K,V>> newEntryIterator() {...}
    
    //它会导致新分配的条目插入链接列表的结尾，并在合适时删除最旧的条目。
    void addEntry(int hash, K key, V value, int bucketIndex) {
        super.addEntry(hash, key, value, bucketIndex);

        Entry<K,V> eldest = header.after;
        if (removeEldestEntry(eldest)) {
            removeEntryForKey(eldest.key);
        }
    }
    
    //不同于addEntry方法，它不调整表或删除最旧的条目。
    void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMap.Entry<K,V> old = table[bucketIndex];
        Entry<K,V> e = new Entry<>(hash, key, value, old);
        table[bucketIndex] = e;
        e.addBefore(header);
        size++;
    }
    
    //在将新条目插入到映射后， put 和 putAll 将调用此方法。
    //此方法可以提供在每次添加新条目时移除最旧条目的实现程序。
    //如果映射表示缓存，则此方法非常有用：它允许映射通过删除旧条目来减少内存损耗。
    //此方法始终返回false
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
  }
```
