* Hashtable类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)        
  &nbsp;&nbsp; 此类实现一个哈希表，该哈希表将键映射到相应的值。任何非 null 对象都可以用作键或值。

  &nbsp;&nbsp; 为了成功地在哈希表中存储和获取对象，用作键的对象必须实现 hashCode 方法和 equals 方法。

  &nbsp;&nbsp; Hashtable 的实例有两个参数影响其性能：初始容量和加载因子。容量是哈希表中桶的数量，初始容量就是哈希表创建时的容量。注意：哈希表的状态为 open：在发生“哈希冲突”的情况下，单个桶会存储多个条目，这些条目必须按顺序搜索。加载因子是对哈希表在其容量自动增加之前可以达到多满的一个尺度。初始容量和加载因子这两个参数只是对该实现的提示。关于何时以及是否调用 rehash 方法的具体细节则依赖于该实现。
  
  &nbsp;&nbsp; 通常，默认加载因子(0.75)在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查找某个条目的时间（在大多数 Hashtable 操作中，包括 get 和 put 操作，都反映了这一点）。
  
  &nbsp;&nbsp; 初始容量主要控制空间消耗与执行 rehash 操作所需要的时间损耗之间的平衡。如果初始容量大于 Hashtable 所包含的最大条目数除以加载因子，则永远不会发生 rehash 操作。但是，将初始容量设置太高可能会浪费空间。
  
  &nbsp;&nbsp; 如果很多条目要存储在一个 Hashtable 中，那么与根据需要执行自动 rehashing 操作来增大表的容量的做法相比，使用足够大的初始容量创建哈希表或许可以更有效地插入条目。
  
  &nbsp;&nbsp; 下面这个示例创建了一个数字的哈希表。它将数字的名称用作键：      
  &nbsp;&nbsp;&nbsp;&nbsp; Hashtable<String, Integer> numbers = new Hashtable<String, Integer>();      
  &nbsp;&nbsp;&nbsp;&nbsp; numbers.put("one", 1);       
  &nbsp;&nbsp;&nbsp;&nbsp; numbers.put("two", 2);     
  &nbsp;&nbsp;&nbsp;&nbsp; numbers.put("three", 3);   
  
  &nbsp;&nbsp; 要获取一个数字，可以使用以下代码：     
  &nbsp;&nbsp;&nbsp;&nbsp; Integer n = numbers.get("two");     
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (n != null) {       
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println("two = " + n);       
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }      
  &nbsp;&nbsp;&nbsp;&nbsp; }
  
  &nbsp;&nbsp; 由所有类的“collection 视图方法”返回的 collection 的 iterator 方法返回的迭代器都是快速失败的：在创建 Iterator 之后，如果从结构上对 Hashtable 进行修改，除非通过 Iterator 自身的 remove 方法，否则在任何时间以任何方式对其进行修改，Iterator 都将抛出ConcurrentModificationException。因此，面对并发的修改，Iterator 很快就会完全失败，而不冒在将来某个不确定的时间发生任意不确定行为的风险。由 Hashtable 的键和元素方法返回的 Enumeration 不是快速失败的。

  &nbsp;&nbsp; 注意：迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误做法：迭代器的快速失败行为应该仅用于检测程序错误。

  &nbsp;&nbsp; 从Java 2 平台 v1.2起，此类就被改进以实现 Map 接口，使它成为 Java Collections Framework 中的一个成员。不像新的 collection 实现，Hashtable 是同步的。
 
```java
  public class Hashtable<K,V> extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    //哈希表数据
    private transient Entry<K,V>[] table;
    
    //哈希表中的条目总数
    private transient int count;
    
    //阈值：哈希表大小超过此阈值时 rehash (int)(初始容量 * 加载因子)
    private int threshold;
    
    //加载因子
    private float loadFactor;
    
    //哈希表已被结构性修改(改变哈希表项目数或其他内部结构修改)的次数，用于迭代哈希表时快速失败
    private transient int modCount = 0;
    
    private static final long serialVersionUID = 1421746759512286392L;
    
    //用于字符串键的map容量的默认阈值。替代散列降低由于计算字符串键弱哈希码的产生碰撞的几率
    static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
    
    //在虚拟机启动之后才能初始化该值
    private static class Holder {
        static final int ALTERNATIVE_HASHING_THRESHOLD;

        static {
            String altThreshold = java.security.AccessController.doPrivileged(
                new sun.security.action.GetPropertyAction("jdk.map.althashing.threshold"));

            int threshold;
            try {
                threshold = (null != altThreshold) 
                   ? Integer.parseInt(altThreshold) : ALTERNATIVE_HASHING_THRESHOLD_DEFAULT;
                        
                // disable alternative hashing if -1
                if (threshold == -1) {
                    threshold = Integer.MAX_VALUE;
                }

                if (threshold < 0) {
                    throw new IllegalArgumentException("value must be positive integer.");
                }
            } catch(IllegalArgumentException failed) {
                throw new Error("Illegal value for 'jdk.map.althashing.threshold'", failed);
            }

            ALTERNATIVE_HASHING_THRESHOLD = threshold;
        }
    }
    
    //哈希种子
    transient int hashSeed;
    
    //初始化哈希码
    final boolean initHashSeedAsNeeded(int capacity) {
        boolean currentAltHashing = hashSeed != 0;
        boolean useAltHashing = sun.misc.VM.isBooted() &&
                (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        boolean switching = currentAltHashing ^ useAltHashing;
        if (switching) {
            hashSeed = useAltHashing ? sun.misc.Hashing.randomHashSeed(this) : 0;
        }
        return switching;
    }
    
    private int hash(Object k) {
        // 如果禁用替代散列，哈希种子hashSeed为0
        return hashSeed ^ k.hashCode();
    }
    
    //用指定初始容量和指定加载因子构造一个新的空哈希表
    public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        initHashSeedAsNeeded(initialCapacity);
    }
    
    //用指定初始容量和默认的加载因子(0.75)构造一个新的空哈希表
    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }
    
    //用默认的初始容量(11)和加载因子(0.75)构造一个新的空哈希表
    public Hashtable() {
        this(11, 0.75f);
    }
    
    //构造一个与给定的 Map 具有相同映射关系的新哈希表。
    //该哈希表是用足以容纳给定 Map 中映射关系的初始容量和默认的加载因子(0.75)创建的。
    public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
    }
    
    //返回此哈希表中的键的数量。
    public synchronized int size() {
        return count;
    }
    
    //测试此哈希表是否没有键映射到值(是否为空)。
    public synchronized boolean isEmpty() {
        return count == 0;
    }
    
    //返回哈希表中键的Enumeration枚举
    public synchronized Enumeration<K> keys() {
        return this.<K>getEnumeration(KEYS);
    }
    
    //返回哈希表中值的枚举
    //使用返回对象上的枚举方法，依次提取元素。
    public synchronized Enumeration<V> elements() {
        return this.<V>getEnumeration(VALUES);
    }
    
    //测试此映射表中是否存在与指定值关联的键。此操作比 containsKey 方法的开销更大。
    //注意：此方法在功能上等同于containsValue方法，containValue是collection框架中Map接口的一部分
    public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }

        Entry tab[] = table;
        for (int i = tab.length ; i-- > 0 ;) {
            for (Entry<K,V> e = tab[i] ; e != null ; e = e.next) {
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }
    
    //如果此 Hashtable 将一个或多个键映射到此值，则返回 true。
    //注意：此方法在功能上等同于contains（它先于 Map 接口）。
    public boolean containsValue(Object value) {
        return contains(value);
    }
    
    //测试指定对象是否为此哈希表中的键。
    public synchronized boolean containsKey(Object key) {
        Entry tab[] = table;
        int hash = hash(key);
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return true;
            }
        }
        return false;
    }
    
    //返回指定键所映射到的值，如果此映射不包含此键的映射，则返回 null.
    //更确切地讲，如果此映射包含满足 (key.equals(k)) 的从键 k 到值 v 的映射，则此方法返回 v；
    //否则，返回 null。（最多只能有一个这样的映射。）
    public synchronized V get(Object key) {
        Entry tab[] = table;
        int hash = hash(key);
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return e.value;
            }
        }
        return null;
    }
    
    //可分配的最大数组大小
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    
    //增加此哈希表的容量并在内部对其进行重组，以便更有效地容纳和访问其元素。
    //当哈希表中的键的数量超出哈希表的容量和加载因子时，自动调用此方法。
    protected void rehash() {
        int oldCapacity = table.length;
        Entry<K,V>[] oldMap = table;

        int newCapacity = (oldCapacity << 1) + 1;
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // 确保最大尺寸的桶运行
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        Entry<K,V>[] newMap = new Entry[newCapacity];

        modCount++;
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        boolean rehash = initHashSeedAsNeeded(newCapacity);

        table = newMap;

        for (int i = oldCapacity ; i-- > 0 ;) {
            for (Entry<K,V> old = oldMap[i] ; old != null ; ) {
                Entry<K,V> e = old;
                old = old.next;

                if (rehash) {
                    e.hash = hash(e.key);
                }
                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                e.next = newMap[index];
                newMap[index] = e;
            }
        }
    }
    
    //将指定 key 映射到此哈希表中的指定 value。键和值都不可以为 null。
    //通过使用与原来的键相同的键调用 get 方法，可以获取相应的值。
    public synchronized V put(K key, V value) {
        // 确保值不为空
        if (value == null) {
            throw new NullPointerException();
        }

        // 确保键不在哈希表中
        Entry tab[] = table;
        int hash = hash(key);
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                V old = e.value;
                e.value = value;
                return old;
            }
        }

        modCount++;
        if (count >= threshold) {
            // 超过阈值Rehash
            rehash();

            tab = table;
            hash = hash(key);
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // 创建新条目
        Entry<K,V> e = tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
        return null;
    }
    
    //从哈希表中移除该键及其相应的值。如果该键不在哈希表中，则此方法不执行任何操作。
    public synchronized V remove(Object key) {
        Entry tab[] = table;
        int hash = hash(key);
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<K,V> e = tab[index], prev = null ; e != null ; prev = e, e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                modCount++;
                if (prev != null) {
                    prev.next = e.next;
                } else {
                    tab[index] = e.next;
                }
                count--;
                V oldValue = e.value;
                e.value = null;
                return oldValue;
            }
        }
        return null;
    }
    
    //将指定映射的所有映射关系复制到此哈希表中，这些映射关系将替换此哈希表拥有的、
    //针对当前指定映射中所有键的所有映射关系。
    public synchronized void putAll(Map<? extends K, ? extends V> t) {
        for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
            put(e.getKey(), e.getValue());
    }
    
    //将此哈希表清空，使其不包含任何键。
    public synchronized void clear() {
        Entry tab[] = table;
        modCount++;
        for (int index = tab.length; --index >= 0; )
            tab[index] = null;
        count = 0;
    }
    
    //创建此哈希表的浅副本。
    //复制哈希表自身的所有结构，但不复制它的键和值。这是一个开销相对较大的操作。
    public synchronized Object clone() {
        try {
            Hashtable<K,V> t = (Hashtable<K,V>) super.clone();
            t.table = new Entry[table.length];
            for (int i = table.length ; i-- > 0 ; ) {
                t.table[i] = (table[i] != null)
                    ? (Entry<K,V>) table[i].clone() : null;
            }
            t.keySet = null;
            t.entrySet = null;
            t.values = null;
            t.modCount = 0;
            return t;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }
    
    //返回此Hashtable对象的字符串表示形式，其形式为ASCII字符","(逗号加空格)分隔开的、括在括号中的一组条目。
    //每个条目都按以下方式呈现：键，一个等号 = 和相关元素，其中 toString 方法用于将键和元素转换为字符串。
    public synchronized String toString() {
        int max = size() - 1;
        if (max == -1)
            return "{}";

        StringBuilder sb = new StringBuilder();
        Iterator<Map.Entry<K,V>> it = entrySet().iterator();

        sb.append('{');
        for (int i = 0; ; i++) {
            Map.Entry<K,V> e = it.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key.toString());
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value.toString());

            if (i == max)
                return sb.append('}').toString();
            sb.append(", ");
        }
    }
    
    private <T> Enumeration<T> getEnumeration(int type) {
        if (count == 0) {
            return Collections.emptyEnumeration();
        } else {
            return new Enumerator<>(type, false);
        }
    }

    private <T> Iterator<T> getIterator(int type) {
        if (count == 0) {
            return Collections.emptyIterator();
        } else {
            return new Enumerator<>(type, true);
        }
    }
    
    //视图
    private transient volatile Set<K> keySet = null;
    private transient volatile Set<Map.Entry<K,V>> entrySet = null;
    private transient volatile Collection<V> values = null;
    
    //返回哈希表的键视图
    public Set<K> keySet() {
        if (keySet == null)
            keySet = Collections.synchronizedSet(new KeySet(), this);
        return keySet;
    }
    private class KeySet extends AbstractSet<K> {...}
    
    //返回哈希表的键值对视图
    public Set<Map.Entry<K,V>> entrySet() {
        if (entrySet==null)
            entrySet = Collections.synchronizedSet(new EntrySet(), this);
        return entrySet;
    }
    private class EntrySet extends AbstractSet<Map.Entry<K,V>> {...}
    
    //返回哈希表的值视图
    public Collection<V> values() {
        if (values==null)
            values = Collections.synchronizedCollection(new ValueCollection(), this);
        return values;
    }
    private class ValueCollection extends AbstractCollection<V> {...}
    
    //按照 Map 接口的定义，比较指定 Object 与此 Map 是否相等
    public synchronized boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<K,V> t = (Map<K,V>) o;
        if (t.size() != size())
            return false;

        try {
            Iterator<Map.Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Map.Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(t.get(key)==null && t.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(t.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }
    
    //按照 Map 接口的定义，返回此 Map 的哈希码值
    //负加载因子表明: 哈希代码计算正在进行中。
    public synchronized int hashCode() {
        int h = 0;
        if (count == 0 || loadFactor < 0)
            return h;  // 返回0

        loadFactor = -loadFactor;  // 哈希码计算进行中
        Entry[] tab = table;
        for (Entry<K,V> entry : tab)
            while (entry != null) {
                h += entry.hashCode();
                entry = entry.next;
            }
        loadFactor = -loadFactor;  // 哈希码计算完成

        return h;
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
            throws IOException {...}
    private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException {...}
    
    //被readObject方法调用
    private void reconstitutionPut(Entry<K,V>[] tab, K key, V value)
        throws StreamCorruptedException {...}
    
    //哈希桶碰撞列表条目
    private static class Entry<K,V> implements Map.Entry<K,V> {
        int hash;
        final K key;
        V value;
        Entry<K,V> next;

        protected Entry(int hash, K key, V value, Entry<K,V> next) {
            this.hash = hash;
            this.key =  key;
            this.value = value;
            this.next = next;
        }

        protected Object clone() {
            return new Entry<>(hash, key, value, (next==null ? null : (Entry<K,V>) next.clone()));
        }

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        public V setValue(V value) {
            if (value == null)
                throw new NullPointerException();

            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry)o;

            return key.equals(e.getKey()) && value.equals(e.getValue());
        }

        public int hashCode() {
            return (Objects.hashCode(key) ^ Objects.hashCode(value));
        }

        public String toString() {
            return key.toString()+"="+value.toString();
        }
    }
    
    //枚举/迭代类型
    private static final int KEYS = 0;
    private static final int VALUES = 1;
    private static final int ENTRIES = 2;
    
    private class Enumerator<T> implements Enumeration<T>, Iterator<T> {...}
    
  }
```
