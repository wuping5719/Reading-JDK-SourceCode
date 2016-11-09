* LinkedHashMap类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
   &nbsp;&nbsp; Map 接口的哈希表和链接列表实现，具有可预知的迭代顺序。此实现与 HashMap 的不同之处在于，后者维护着一个运行于所有条目的双重链接列表。此链接列表定义了迭代顺序，该迭代顺序通常就是将键插入到映射中的顺序（插入顺序）。注意，如果在映射中重新插入 键，则插入顺序不受影响。（如果在调用 m.put(k, v) 前 m.containsKey(k) 返回了 true，则调用时会将键 k 重新插入到映射 m 中。）

   &nbsp;&nbsp; 此实现可以让客户避免未指定的、由 HashMap（及 Hashtable）所提供的通常为杂乱无章的排序工作，同时无需增加与 TreeMap 相关的成本。使用它可以生成一个与原来顺序相同的映射副本，而与原映射的实现无关：
     void foo(Map m) {
         Map copy = new LinkedHashMap(m);
         ...
     }
 
  &nbsp;&nbsp; 如果模块通过输入得到一个映射，复制这个映射，然后返回由此副本确定其顺序的结果，这种情况下这项技术特别有用。（客户通常期望返回的内容与其出现的顺序相同。）
提供特殊的构造方法来创建链接哈希映射，该哈希映射的迭代顺序就是最后访问其条目的顺序，从近期访问最少到近期访问最多的顺序（访问顺序）。这种映射很适合构建 LRU 缓存。调用 put 或 get 方法将会访问相应的条目（假定调用完成后它还存在）。putAll 方法以指定映射的条目集迭代器提供的键-值映射关系的顺序，为指定映射的每个映射关系生成一个条目访问。任何其他方法均不生成条目访问。特别是，collection 视图上的操作不 影响底层映射的迭代顺序。

  &nbsp;&nbsp; 可以重写 removeEldestEntry(Map.Entry) 方法来实施策略，以便在将新映射关系添加到映射时自动移除旧的映射关系。

  &nbsp;&nbsp; 此类提供所有可选的 Map 操作，并且允许 null 元素。与 HashMap 一样，它可以为基本操作（add、contains 和 remove）提供稳定的性能，假定哈希函数将元素正确分布到桶中。由于增加了维护链接列表的开支，其性能很可能比 HashMap 稍逊一筹，不过这一点例外：LinkedHashMap 的 collection 视图迭代所需时间与映射的大小 成比例。HashMap 迭代时间很可能开支较大，因为它所需要的时间与其容量 成比例。

  &nbsp;&nbsp; 链接的哈希映射具有两个影响其性能的参数：初始容量和加载因子。它们的定义与 HashMap 极其相似。要注意，为初始容量选择非常高的值对此类的影响比对 HashMap 要小，因为此类的迭代时间不受容量的影响。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问链接的哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须 保持外部同步。这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射的意外的非同步访问：

    Map m = Collections.synchronizedMap(new LinkedHashMap(...));
  &nbsp;&nbsp; 结构修改是指添加或删除一个或多个映射关系，或者在按访问顺序链接的哈希映射中影响迭代顺序的任何操作。在按插入顺序链接的哈希映射中，仅更改与映射中已包含键关联的值不是结构修改。 在按访问顺序链接的哈希映射中，仅利用 get 查询映射不是结构修改。）
Collection（由此类的所有 collection 视图方法所返回）的 iterator 方法返回的迭代器都是快速失败 的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器自身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒将来不确定的时间任意发生不确定行为的风险。

  &nbsp;&nbsp; 注意：迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的方式是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class LinkedHashMap<K,V> extends HashMap<K,V>
    implements Map<K,V> {
    private static final long serialVersionUID = 3801124242820219131L;
    
    //
    private transient Entry<K,V> header;
  }
```
