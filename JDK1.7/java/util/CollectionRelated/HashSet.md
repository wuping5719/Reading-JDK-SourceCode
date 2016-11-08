* HashSet类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)       
  &nbsp;&nbsp; 此类实现 Set 接口，由哈希表（实际上是一个 HashMap 实例）支持。它不保证 set 的迭代顺序；特别是它不保证该顺序恒久不变。此类允许使用 null 元素。

  &nbsp;&nbsp; 此类为基本操作提供了稳定性能，这些基本操作包括 add、remove、contains 和 size，假定哈希函数将这些元素正确地分布在桶中。对此 set 进行迭代所需的时间与 HashSet 实例的大小（元素的数量）和底层 HashMap 实例（桶的数量）的“容量”的和成比例。因此，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问一个哈希 set，而其中至少一个线程修改了该 set，那么它必须 保持外部同步。这通常是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSet 方法来“包装” set。最好在创建时完成这一操作，以防止对该 set 进行意外的不同步访问：      
  &nbsp;&nbsp; Set s = Collections.synchronizedSet(new HashSet(...));    
   
 &nbsp;&nbsp; 此类的 iterator 方法返回的迭代器是快速失败的：在创建迭代器之后，如果对 set 进行修改，除非通过迭代器自身的 remove 方法，否则在任何时间以任何方式对其进行修改，Iterator 都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒将来在某个不确定时间发生任意不确定行为的风险。

 &nbsp;&nbsp; 注意：迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器在尽最大努力抛出 ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误做法：迭代器的快速失败行为应该仅用于检测 bug。

 &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    static final long serialVersionUID = -5024744406713321676L;
    
    //实际用HashMap存储
    private transient HashMap<E,Object> map;
    
    //在背后的存储结构HashMap中关联的虚值对象
    private static final Object PRESENT = new Object();
    
    //构造一个新的空 set，其底层 HashMap 实例的默认初始容量是 16，加载因子是 0.75。
    public HashSet() {
        map = new HashMap<>();
    }
    
    //构造一个包含指定 collection 中的元素的新 set。
    //使用默认的加载因子 0.75 和足以包含指定 collection 中所有元素的初始容量来创建 HashMap。
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
    
    //构造一个新的空 set，其底层 HashMap 实例具有指定的初始容量和指定的加载因子。
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
    
    //构造一个新的空 set，其底层 HashMap 实例具有指定的初始容量和默认的加载因子（0.75）。
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
    
    //构造一个新的空 linked hash set.(该构造函数仅被LinkedHashSet使用)
    //其底层 LinkedHashMap 实例具有指定的初始容量和和指定的加载因子。
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    
    //返回对此 set 中元素进行迭代的迭代器。返回元素的顺序并不是特定的。
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
    
    //返回此 set 中的元素的数量（set 的容量）。
    public int size() {
        return map.size();
    }
    
    //如果此 set 不包含任何元素，则返回 true。
    public boolean isEmpty() {
        return map.isEmpty();
    }
    
    //如果此 set 包含指定元素，则返回 true。 
    //更确切地讲，当且仅当此set包含一个满足(o==null ? e==null : o.equals(e))的e元素时，返回true。
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
    
    //如果此 set 中尚未包含指定元素，则添加指定元素。
    //更确切地讲，如果此set没有包含满足(e==null ? e2==null : e.equals(e2))的元素e2，
    //则向此set添加指定的元素 e。如果此 set 已包含该元素，则该调用不更改 set 并返回 false。
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    
    //如果指定元素存在于此 set 中，则将其移除。
    //更确切地讲，如果此set包含一个满足(o==null ? e==null : o.equals(e))的元素e，则将其移除。
    //如果此 set 已包含该元素，则返回 true（或者：如果此 set 因调用而发生更改，则返回 true）。
    //（一旦调用返回，则此 set 不再包含该元素）。
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
    
    //从此 set 中移除所有元素。此调用返回后，该 set 将为空。
    public void clear() {
        map.clear();
    }
    
    //返回此 HashSet 实例的浅副本：并没有复制这些元素本身。
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E, Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        s.defaultWriteObject();

        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());

        s.writeInt(map.size());

        for (E e : map.keySet())
            s.writeObject(e);
    }
    
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {...}
  }
```
