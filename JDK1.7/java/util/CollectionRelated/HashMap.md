* HashMap类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)       
  &nbsp;&nbsp; 基于哈希表的 Map 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。（除了非同步和允许使用 null 之外，HashMap 类与 Hashtable 大致相同）此类不保证映射的顺序，特别是它不保证该顺序恒久不变。     
  &nbsp;&nbsp; HashMap的存储结构图如下：
  <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_hashmap.JPG" /></p>
  
  &nbsp;&nbsp; 此实现假定哈希函数将元素适当地分布在各桶之间，可为基本操作（get 和 put）提供稳定的性能。迭代 collection 视图所需的时间与 HashMap 实例的“容量”（桶的数量）及其大小（键-值映射关系数）成比例。所以，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。

  &nbsp;&nbsp; HashMap 的实例有两个参数影响其性能：初始容量和加载因子。容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。

  &nbsp;&nbsp; 通常，默认加载因子 (0.75) 在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

  &nbsp;&nbsp; 如果很多映射关系要存储在 HashMap 实例中，则相对于按需执行自动的 rehash 操作以增大表的容量来说，使用足够大的初始容量创建它将使得映射关系能更有效地存储。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改）这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，如下所示:     &nbsp;&nbsp; Map m = Collections.synchronizedMap(new HashMap(...));
  
  &nbsp;&nbsp; 由所有此类的“collection 视图方法”所返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。

  &nbsp;&nbsp; 注意：迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //默认的初始容量为16---必须是2的2的正数次方
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    
    //最大容量---2的30次方
    static final int MAXIMUM_CAPACITY = 1 << 30;
    
    //默认加载因子(0.75)
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    //共享的空表实例
    static final Entry<?,?>[] EMPTY_TABLE = {};
    
    //可调整大小的表实例，长度必须是2的正数次方
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
    
    //map中包含的键-值对的数量
    transient int size;
    
    //阈值：下一次可调整到的Size大小(容量 * 加载因子)
    int threshold;
    
    //加载因子
    final float loadFactor;
    
    //HashMap结构被修改的次数(映射数变化或其他内部结构修改，例如：rehash)，用于集合视图遍历时使迭代器快速失败
    transient int modCount;
    
    //字符串键-值映射容量的默认阈值，替代Hash以降低计算字符串键弱哈希码发生碰撞的几率
    static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
    
    //在虚拟机启动后才能初始化
    private static class Holder {
      //切换使用替代散列时哈希表的容量
      static final int ALTERNATIVE_HASHING_THRESHOLD;
      static {
            String altThreshold = java.security.AccessController.doPrivileged(
                new sun.security.action.GetPropertyAction("jdk.map.althashing.threshold"));

            int threshold;
            try {
                threshold = (null != altThreshold)
                        ? Integer.parseInt(altThreshold)
                        : ALTERNATIVE_HASHING_THRESHOLD_DEFAULT;

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
    
    //哈希种子(如果是0，则禁用可选择的散列)
    transient int hashSeed = 0;
    
    //构造一个带指定初始容量和加载因子的空HashMap
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();
    }
    
    //构造一个带指定初始容量和默认加载因子(0.75)的空HashMap
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    //构造一个具有默认初始容量(16)和默认加载因子(0.75)的空HashMap
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }
    
    //构造一个映射关系与指定Map相同的新HashMap
    //所创建的HashMap具有默认加载因子(0.75)和足以容纳指定Map中映射关系的初始容量
    public HashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        inflateTable(threshold);

        putAllForCreate(m);
    }
    
    private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        int rounded = number >= MAXIMUM_CAPACITY ? MAXIMUM_CAPACITY
                : (rounded = Integer.highestOneBit(number)) != 0
                    ? (Integer.bitCount(number) > 1) ? rounded << 1 : rounded : 1;

        return rounded;
    }
    
    //扩大表的容量(给表"充气")
    private void inflateTable(int toSize) {
        // Find a power of 2 >= toSize
        int capacity = roundUpToPowerOf2(toSize);

        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }
    
    //子类的初始化钩子。该方法在所有的构造函数和伪构造函数（克隆，readObject）中，
    //在HashMap已被初始化，但还没有插入任何数据的情况下被调用。
    void init() {
    }
    
    //初始化哈希掩码值(延迟初始化)
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
    
    //检索对象哈希代码，并将一个补充的哈希函数应用到结果哈希中，以防质量差的哈希函数
    final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
    
    //返回哈希码的索引
    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
    
    //返回此映射中的键-值映射关系数
    public int size() {
        return size;
    }
    
    //如果此映射不包含键-值映射关系，则返回true
    public boolean isEmpty() {
        return size == 0;
    }
    
    //返回指定键所映射的值；如果对于该键来说，此映射不包含任何映射关系，则返回null
    //更确切地讲，如果此映射包含一个满足(key==null ? k==null : key.equals(k))的从k键到v值的映射关系，
    //则此方法返回v；否则返回 null。（最多只能有一个这样的映射关系。）
    //返回 null 值并不一定 表明该映射不包含该键的映射关系；也可能该映射将该键显示地映射为 null。
    //可使用 containsKey 操作来区分这两种情况。
    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
    
    //get()的查null键版本
    private V getForNullKey() {
        if (size == 0) {
            return null;
        }
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }
    
    //如果此映射包含对于指定键的映射关系，则返回true。
    public boolean containsKey(Object key) {
        return getEntry(key) != null;
    }
    
    //返回指定键的关联条目。如果HashMap没有包含指定键的映射，则返回null
    final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
    
    //在此映射中关联指定值与指定键。如果该映射以前包含了一个该键的映射关系，则旧值被替换
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
    
    //put()的null键版本
    private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
    
    //该方法用来替代构造方法和pseudoconstructors (clone, readObject)
    //它不调整表，检查comodification
    private void putForCreate(K key, V value) {
        int hash = null == key ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        //查找键对应的条目
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                e.value = value;
                return;
            }
        }

        createEntry(hash, key, value, i);
    }
    private void putAllForCreate(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            putForCreate(e.getKey(), e.getValue());
    }
    
    //Rehash
    //将map中的内容转存到一个有更大容量的新Entry[]数组中
    //当map中键的数量达到阈值时，自动调用此方法
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
    
    //将旧表Entry[]中的数据搬到新表Entry[]
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
    
    //将指定映射的所有映射关系复制到此映射中，这些映射关系将替换此映射目前针对指定映射中所有键的所有映射关系
    public void putAll(Map<? extends K, ? extends V> m) {
        int numKeysToBeAdded = m.size();
        if (numKeysToBeAdded == 0)
            return;

        if (table == EMPTY_TABLE) {
            inflateTable((int) Math.max(numKeysToBeAdded * loadFactor, threshold));
        }

        //自动扩容
        if (numKeysToBeAdded > threshold) {
            int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
            if (targetCapacity > MAXIMUM_CAPACITY)
                targetCapacity = MAXIMUM_CAPACITY;
            int newCapacity = table.length;
            while (newCapacity < targetCapacity)
                newCapacity <<= 1;
            if (newCapacity > table.length)
                resize(newCapacity);
        }

        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }
    
    //从此映射中移除指定键的映射关系（如果存在）,返回映射值
    public V remove(Object key) {
        Entry<K,V> e = removeEntryForKey(key);
        return (e == null ? null : e.value);
    }
    
    //从此映射中移除指定键的映射关系（如果存在），返回整个映射键-值对
    final Entry<K,V> removeEntryForKey(Object key) {
        if (size == 0) {
            return null;
        }
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }
    
    //使用Map.Entry.equals()匹配的移除键-值对的特别版
    final Entry<K,V> removeMapping(Object o) {
        if (size == 0 || !(o instanceof Map.Entry))
            return null;

        Map.Entry<K,V> entry = (Map.Entry<K,V>) o;
        Object key = entry.getKey();
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            if (e.hash == hash && e.equals(entry)) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }
    
    //从此映射中移除所有映射关系。此调用返回后，映射将为空
    public void clear() {
        modCount++;
        Arrays.fill(table, null);
        size = 0;
    }
    
    //如果此映射将一个或多个键映射到指定值，则返回true
    public boolean containsValue(Object value) {
        if (value == null)
            return containsNullValue();

        Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (value.equals(e.value))
                    return true;
        return false;
    }
    //参数为空的containsValue特例版
    private boolean containsNullValue() {
        Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (e.value == null)
                    return true;
        return false;
    }
    
    //返回此HashMap实例的浅副本：并不复制键和值本身
    public Object clone() {
        HashMap<K,V> result = null;
        try {
            result = (HashMap<K,V>)super.clone();
        } catch (CloneNotSupportedException e) { 
            // assert false;
        }
        if (result.table != EMPTY_TABLE) {
            result.inflateTable(Math.min(
                (int) Math.min(size * Math.min(1 / loadFactor, 4.0f), 
                HashMap.MAXIMUM_CAPACITY), table.length));
        }
        result.entrySet = null;
        result.modCount = 0;
        result.size = 0;
        result.init();
        result.putAllForCreate(this);

        return result;
    }
    
    static class Entry<K,V> implements Map.Entry<K,V> {...}
    
    //将带有指定键、值和哈希码的新条目添加到指定的桶中。该方法会适时调整表的大小。
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
    
    //类似addEntry，创建一个新条目
    void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
    
    private abstract class HashIterator<E> implements Iterator<E> {...}
    private final class ValueIterator extends HashIterator<V> {...}
    private final class KeyIterator extends HashIterator<K> {...}
    private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {...}
    
    //子类重写这些迭代方法
    Iterator<K> newKeyIterator() {...}
    Iterator<V> newValueIterator() {...}
    Iterator<Map.Entry<K,V>> newEntryIterator() {...}
    //视图
    private transient Set<Map.Entry<K,V>> entrySet = null;
    
    //返回此映射中所包含的键的 Set 视图。
    //该 set 受映射的支持，所以对映射的更改将反映在该 set 中，反之亦然。
    //如果在对set进行迭代的同时修改了映射（通过迭代器自己的remove操作除外），则迭代结果是不确定的。
    //该 set 支持元素的移除，通过 Iterator.remove、 Set.remove、 removeAll、 
    //retainAll 和 clear 操作可从该映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
    public Set<K> keySet() {
        Set<K> ks = keySet;
        return (ks != null ? ks : (keySet = new KeySet()));
    }
    private final class KeySet extends AbstractSet<K> {...}
    
    //返回此映射所包含的值的 Collection 视图。
    //该 collection 受映射的支持，所以对映射的更改将反映在该 collection 中，反之亦然。
    //如果在对collection进行迭代的同时修改了映射（通过迭代器自己的remove操作除外），则迭代结果是不确定的
    //该 collection 支持元素的移除，通过 Iterator.remove、 Collection.remove、 removeAll、 
    //retainAll 和 clear 操作可从该映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
    public Collection<V> values() {
        Collection<V> vs = values;
        return (vs != null ? vs : (values = new Values()));
    }
    private final class Values extends AbstractCollection<V> {...}
    
    //返回此映射所包含的映射关系的 Set 视图。 
    //该 set 受映射支持，所以对映射的更改将反映在此 set 中，反之亦然。
    //如果在对 set 进行迭代的同时修改了映射（通过迭代器自己的remove操作，
    //或者通过在该迭代器返回的映射项上执行 setValue 操作除外），则迭代结果是不确定的。
    //该 set 支持元素的移除，通过 Iterator.remove、 Set.remove、 removeAll、 
    // retainAll 和 clear 操作可从该映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
    public Set<Map.Entry<K,V>> entrySet() {
        return entrySet0();
    }
    private Set<Map.Entry<K,V>> entrySet0() {
        Set<Map.Entry<K,V>> es = entrySet;
        return es != null ? es : (entrySet = new EntrySet());
    }
    private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {...}
    
    //序列化和反序列化
    private void writeObject(java.io.ObjectOutputStream s)
        throws IOException {...}
    private static final long serialVersionUID = 362498820763181265L;
    private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException {...}
    int   capacity()     { return table.length; }
    float loadFactor()   { return loadFactor;   }
  }
```
