* TreeSet类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)       
  &nbsp;&nbsp; 基于 TreeMap 的 NavigableSet 实现。使用元素的自然顺序对元素进行排序，或者根据创建 set 时提供的 Comparator 进行排序，具体取决于使用的构造方法。

  &nbsp;&nbsp; 此实现为基本操作（add、remove 和 contains）提供受保证的 log(n) 时间开销。

  &nbsp;&nbsp; 注意：如果要正确实现 Set 接口，则 set 维护的顺序（无论是否提供了显式比较器）必须与 equals 一致。（关于与 equals 一致的精确定义，请参阅 Comparable 或 Comparator。）这是因为 Set 接口是按照 equals 操作定义的，但 TreeSet 实例使用它的 compareTo（或 compare）方法对所有元素进行比较，因此从 set 的观点来看，此方法认为相等的两个元素就是相等的。即使 set 的顺序与 equals 不一致，其行为也是定义良好的；它只是违背了 Set 接口的常规协定。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问一个 TreeSet，而其中至少一个线程修改了该 set，那么它必须外部同步。这一般是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSortedSet 方法来“包装”该 set。此操作最好在创建时进行，以防止对 set 的意外非同步访问：    
  &nbsp;&nbsp; &nbsp;&nbsp;  SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));
  
  &nbsp;&nbsp; 此类的 iterator 方法返回的迭代器是快速失败 的：在创建迭代器之后，如果从结构上对 set 进行修改，除非通过迭代器自身的 remove 方法，否则在其他任何时间以任何方式进行修改都将导致迭代器抛出 ConcurrentModificationException。因此，对于并发的修改，迭代器很快就完全失败，而不会冒着在将来不确定的时间发生不确定行为的风险。

  &nbsp; &nbsp; 注意：迭代器的快速失败行为无法得到保证，一般来说，存在不同步的并发修改时，不可能作出任何肯定的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测 bug。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
    //实际的NavigableMap存储结构
    private transient NavigableMap<E,Object> m;
    
    //与实际存储结构NavigableMap关联的虚值对象
    private static final Object PRESENT = new Object();
    
    //构造一个以NavigableMap为存储结构的Set
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
    
    //构造一个新的空 set，该 set 根据其元素的自然顺序进行排序。
    //插入该 set 的所有元素都必须实现 Comparable 接口。
    //另外，所有这些元素都必须是可互相比较的：对于 set 中的任意两个元素 e1 和 e2，
    //执行 e1.compareTo(e2) 都不得抛出 ClassCastException。
    //如果用户试图将违反此约束的元素添加到 set
    //（例如，用户试图将字符串元素添加到其元素为整数的 set 中），则 add 调用将抛出 ClassCastException。
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }
    
    //构造一个新的空 TreeSet，它根据指定比较器进行排序。
    //插入到该 set 的所有元素都必须能够由指定比较器进行相互比较：对于 set 中的任意两个元素 e1 和 e2，
    //执行 comparator.compare(e1, e2) 都不得抛出 ClassCastException。
    //如果用户试图将违反此约束的元素添加到 set 中，则 add 调用将抛出 ClassCastException。
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
    
    //构造一个包含指定 collection 元素的新 TreeSet，它按照其元素的自然顺序进行排序。
    //插入该 set 的所有元素都必须实现 Comparable 接口。另外，所有这些元素都必须是可互相比较的：
    //对于 set 中的任意两个元素 e1 和 e2，执行 e1.compareTo(e2) 都不得抛出 ClassCastException。
    public TreeSet(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    
    //构造一个与指定有序 set 具有相同映射关系和相同排序的新 TreeSet。
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }
    
    //返回在此 set 中的元素上按升序进行迭代的迭代器。
    public Iterator<E> iterator() {
        return m.navigableKeySet().iterator();
    }
    
    //返回在此 set 元素上按降序进行迭代的迭代器。
    public Iterator<E> descendingIterator() {
        return m.descendingKeySet().iterator();
    }
    
    //返回此 set 中所包含元素的逆序视图。
    //降序 set 受此 set 的支持，因此对此 set 的更改将反映在降序 set 中，反之亦然。
    //如果在对任一set进行迭代的同时修改了任一set（通过迭代器自己的remove操作除外），则迭代结果是不确定的。
    //返回 set 的顺序等于 Collections.reverseOrder(comparator())。
    //表达式 s.descendingSet().descendingSet() 返回的 s 视图基本等于 s。
    public NavigableSet<E> descendingSet() {
        return new TreeSet<>(m.descendingMap());
    }
    
    //返回 set 中的元素数（set 的容量）。
    public int size() {
        return m.size();
    }
    
    //如果此 set 不包含任何元素，则返回 true。
    public boolean isEmpty() {
        return m.isEmpty();
    }
    
    //如果此 set 包含指定的元素，则返回 true。
    //更确切地讲，当且仅当此set包含满足(o==null ? e==null : o.equals(e))的元素e时，返回true。
    public boolean contains(Object o) {
        return m.containsKey(o);
    }
    
    //将指定的元素添加到此 set（如果该元素尚未存在于 set 中）。
    //更确切地讲，如果该 set 不包含满足 (e==null ? e2==null : e.equals(e2)) 的元素 e2，
    //则将指定元素 e 添加到此 set 中。如果此 set 已经包含这样的元素，则该调用不改变此 set 并返回false。
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
    
    //将指定的元素从 set 中移除（如果该元素存在于此 set 中）。
    //更确切地讲，如果 set 中包含满足 (o==null ? e==null : o.equals(e)) 的元素 e，
    //则移除一个这样的元素。如果此 set 包含这样的元素（或者此 set 由于调用而发生更改），则返回 true。
    //（一旦调用返回，则此 set 不再包含这样的元素。）
    public boolean remove(Object o) {
        return m.remove(o)==PRESENT;
    }
    
    //移除此 set 中的所有元素。在此调用返回之后，set 将为空。
    public void clear() {
        m.clear();
    }
    
    //将指定 collection 中的所有元素添加到此 set 中。
    public  boolean addAll(Collection<? extends E> c) {
        if (m.size()==0 && c.size() > 0 
          && c instanceof SortedSet && m instanceof TreeMap) {
            SortedSet<? extends E> set = (SortedSet<? extends E>) c;
            TreeMap<E,Object> map = (TreeMap<E, Object>) m;
            Comparator<? super E> cc = (Comparator<? super E>) set.comparator();
            Comparator<? super E> mc = map.comparator();
            if (cc==mc || (cc != null && cc.equals(mc))) {
                map.addAllForTreeSet(set, PRESENT);
                return true;
            }
        }
        return super.addAll(c);
    }
    
    //返回此 set 的部分视图，其元素范围从 fromElement 到 toElement。
    //如果fromElement和toElement相等，则返回的set为空，除非fromExclusive和toExclusive都为true。
    //返回的 set 受此 set 支持，所以在返回 set 中的更改将反映在此 set 中，反之亦然。
    //返回 set 支持此 set 支持的所有可选 set 操作。
    //如果试图在返回 set 的范围之外插入元素，则返回的 set 将抛出 IllegalArgumentException。
    public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                                  E toElement,   boolean toInclusive) {
        return new TreeSet<>(m.subMap(fromElement, fromInclusive,
                                       toElement,   toInclusive));
    }
    
    //返回此 set 的部分视图，其元素小于（或等于，如果 inclusive 为 true） toElement。
    //返回的 set 受此 set 支持，所以在返回 set 中的更改将反映在此 set 中，反之亦然。
    //返回 set 支持此 set 支持的所有可选 set 操作。
    //如果试图在返回 set 的范围之外插入元素，则返回的 set 将抛出 IllegalArgumentException。
    public NavigableSet<E> headSet(E toElement, boolean inclusive) {
        return new TreeSet<>(m.headMap(toElement, inclusive));
    }
    
    //返回此 set 的部分视图，其元素大于（或等于，如果 inclusive 为 true） fromElement。
    //返回的 set 受此 set 支持，所以在返回 set 中的更改将反映在此 set 中，反之亦然。
    //返回 set 支持此 set 支持的所有可选 set 操作。
    //如果试图在返回 set 的范围之外插入元素，则返回的 set 将抛出 IllegalArgumentException。
    public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
        return new TreeSet<>(m.tailMap(fromElement, inclusive));
    }
    
    //返回此 set 的部分视图，其元素从 fromElement（包括）到 toElement（不包括）。
    //（如果 fromElement 和 toElement 相等，则返回空的 set）。
    //返回的 set 受此 set 支持，所以在返回 set 中的更改将反映在此 set 中，反之亦然。
    //返回的 set 支持此 set 支持的所有可选 set 操作。
    //如果试图在返回 set 的范围之外插入元素，则返回的 set 将抛出 IllegalArgumentException。
    //等效于 subSet(fromElement, true, toElement, false)。
    public SortedSet<E> subSet(E fromElement, E toElement) {
        return subSet(fromElement, true, toElement, false);
    }
    
    //返回此 set 的部分视图，其元素严格小于 toElement。
    //返回的 set 受此 set 支持，所以在返回 set 中的更改将反映在此 set 中，反之亦然。
    //返回的 set 支持此 set 支持的所有可选 set 操作。
    //如果试图在返回 set 的范围之外插入元素，则返回的 set 将抛出 IllegalArgumentException。
    //等效于 headSet(toElement, false)。
    public SortedSet<E> headSet(E toElement) {
        return headSet(toElement, false);
    }
    
    //返回此 set 的部分视图，其元素大于等于 fromElement。
    //返回的 set 受此 set 支持，所以在返回 set 中的更改将反映在此 set 中，反之亦然。
    //返回的 set 支持此 set 支持的所有可选 set 操作。
    //如果试图在返回 set 的范围之外插入元素，则返回的 set 将抛出 IllegalArgumentException。
    //等效于 tailSet(fromElement, true)。
    public SortedSet<E> tailSet(E fromElement) {
        return tailSet(fromElement, true);
    }
    
    //返回对此 set 中的元素进行排序的比较器；如果此 set 使用其元素的自然顺序，则返回 null。
    public Comparator<? super E> comparator() {
        return m.comparator();
    }
    
    //返回此 set 中当前第一个（最低）元素。
    public E first() {
        return m.firstKey();
    }
    
    //返回此 set 中当前最后一个（最高）元素。
    public E last() {
        return m.lastKey();
    }
    
    //返回此 set 中严格小于给定元素的最大元素；如果不存在这样的元素，则返回 null。
    public E lower(E e) {
        return m.lowerKey(e);
    }
    
    //返回此 set 中小于等于给定元素的最大元素；如果不存在这样的元素，则返回 null。
    public E floor(E e) {
        return m.floorKey(e);
    }
    
    //返回此 set 中大于等于给定元素的最小元素；如果不存在这样的元素，则返回 null。
    public E ceiling(E e) {
        return m.ceilingKey(e);
    }
    
    //返回此 set 中严格大于给定元素的最小元素；如果不存在这样的元素，则返回 null。
    public E higher(E e) {
        return m.higherKey(e);
    }
    
    //获取并移除第一个（最低）元素；如果此 set 为空，则返回 null。
    public E pollFirst() {
        Map.Entry<E,?> e = m.pollFirstEntry();
        return (e == null) ? null : e.getKey();
    }
    
    //获取并移除最后一个（最高）元素；如果此 set 为空，则返回 null。
    public E pollLast() {
        Map.Entry<E,?> e = m.pollLastEntry();
        return (e == null) ? null : e.getKey();
    }
    
    //返回 TreeSet 实例的浅副本（这些元素本身不被复制）
    public Object clone() {
        TreeSet<E> clone = null;
        try {
            clone = (TreeSet<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }

        clone.m = new TreeMap<>(m);
        return clone;
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {...}
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {...}
    private static final long serialVersionUID = -2479143000061671589L;
  }
```
