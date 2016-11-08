* TreeSet类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)       
  &nbsp;&nbsp; 基于 TreeMap 的 NavigableSet 实现。使用元素的自然顺序对元素进行排序，或者根据创建 set 时提供的 Comparator 进行排序，具体取决于使用的构造方法。

  &nbsp;&nbsp; 此实现为基本操作（add、remove 和 contains）提供受保证的 log(n) 时间开销。

  &nbsp;&nbsp; 注意：如果要正确实现 Set 接口，则 set 维护的顺序（无论是否提供了显式比较器）必须与 equals 一致。（关于与 equals 一致 的精确定义，请参阅 Comparable 或 Comparator。）这是因为 Set 接口是按照 equals 操作定义的，但 TreeSet 实例使用它的 compareTo（或 compare）方法对所有元素进行比较，因此从 set 的观点来看，此方法认为相等的两个元素就是相等的。即使 set 的顺序与 equals 不一致，其行为也是 定义良好的；它只是违背了 Set 接口的常规协定。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问一个 TreeSet，而其中至少一个线程修改了该 set，那么它必须 外部同步。这一般是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSortedSet 方法来“包装”该 set。此操作最好在创建时进行，以防止对 set 的意外非同步访问：
  &nbsp;&nbsp; &nbsp;&nbsp;  SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));
  
  &nbsp;&nbsp; 此类的 iterator 方法返回的迭代器是快速失败 的：在创建迭代器之后，如果从结构上对 set 进行修改，除非通过迭代器自身的 remove 方法，否则在其他任何时间以任何方式进行修改都将导致迭代器抛出 ConcurrentModificationException。因此，对于并发的修改，迭代器很快就完全失败，而不会冒着在将来不确定的时间发生不确定行为的风险。

  &nbsp;&nbsp： 注意：迭代器的快速失败行为无法得到保证，一般来说，存在不同步的并发修改时，不可能作出任何肯定的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测 bug。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
    //实际的NavigableMap存储结构
    private transient NavigableMap<E,Object> m;
    
    //与实际存储结构NavigableMap关联的虚值对象
    private static final Object PRESENT = new Object();
    
    //构造一个新的空 set，该 set 根据其元素的自然顺序进行排序。
    //插入该 set 的所有元素都必须实现 Comparable 接口。
    //另外，所有这些元素都必须是可互相比较的：对于 set 中的任意两个元素 e1 和 e2，
    //执行 e1.compareTo(e2) 都不得抛出 ClassCastException。
    //如果用户试图将违反此约束的元素添加到 set
    //（例如，用户试图将字符串元素添加到其元素为整数的 set 中），则 add 调用将抛出 ClassCastException。
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
    
    //
    
  }
```
