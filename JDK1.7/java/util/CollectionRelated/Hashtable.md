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
    
    //
    private transient int modCount = 0;
  }
```
