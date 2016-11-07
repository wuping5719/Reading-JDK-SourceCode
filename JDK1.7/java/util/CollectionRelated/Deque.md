* Deque接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
   &nbsp;&nbsp; 一个线性 collection，支持在两端插入和移除元素。名称 deque 是“double ended queue（双端队列）”的缩写，通常读为“deck”。大多数 Deque 实现对于它们能够包含的元素数没有固定限制，但此接口既支持有容量限制的双端队列，也支持没有固定大小限制的双端队列。

   &nbsp;&nbsp; 此接口定义在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或 false，具体取决于操作）。插入操作的后一种形式是专为使用有容量限制的 Deque 实现设计的；在大多数实现中，插入操作不能失败。

   &nbsp;&nbsp; 下表总结了上述 12 种方法：    
   &nbsp;&nbsp;&nbsp;&nbsp; 第一个元素（头部）	最后一个元素（尾部）     
   &nbsp;&nbsp;&nbsp;&nbsp; 抛出异常	特殊值	抛出异常	特殊值    
   &nbsp;&nbsp;&nbsp;&nbsp; 插入	addFirst(e)	offerFirst(e)	addLast(e)	offerLast(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; 移除	removeFirst()	pollFirst()	removeLast()	pollLast()     
   &nbsp;&nbsp;&nbsp;&nbsp; 检查	getFirst()	peekFirst()	getLast()	peekLast()     
   &nbsp;&nbsp; 此接口扩展了 Queue 接口。在将双端队列用作队列时，将得到 FIFO（先进先出）行为。将元素添加到双端队列的末尾，从双端队列的开头移除元素。从 Queue 接口继承的方法完全等效于 Deque 方法，如下表所示：

   &nbsp;&nbsp;&nbsp;&nbsp; Queue 方法	等效 Deque 方法     
   &nbsp;&nbsp;&nbsp;&nbsp; add(e)	addLast(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; offer(e)	offerLast(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; remove()	removeFirst()      
   &nbsp;&nbsp;&nbsp;&nbsp; poll()	pollFirst()     
   &nbsp;&nbsp;&nbsp;&nbsp; element()	getFirst()     
   &nbsp;&nbsp;&nbsp;&nbsp; peek()	peekFirst()     
   &nbsp;&nbsp; 双端队列也可用作 LIFO（后进先出）堆栈。应优先使用此接口而不是遗留 Stack 类。在将双端队列用作堆栈时，元素被推入双端队列的开头并从双端队列开头弹出。堆栈方法完全等效于 Deque 方法，如下表所示：

   &nbsp;&nbsp;&nbsp;&nbsp; 堆栈方法	等效 Deque 方法    
   &nbsp;&nbsp;&nbsp;&nbsp; push(e)	addFirst(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; pop()	removeFirst()   
   &nbsp;&nbsp;&nbsp;&nbsp; peek()	peekFirst()    
   &nbsp;&nbsp; 注意：在将双端队列用作队列或堆栈时，peek 方法同样正常工作；无论哪种情况下，都从双端队列的开头抽取元素。

   &nbsp;&nbsp; 此接口提供了两种移除内部元素的方法：removeFirstOccurrence 和 removeLastOccurrence。

   &nbsp;&nbsp; 与 List 接口不同，此接口不支持通过索引访问元素。

   &nbsp;&nbsp; 虽然 Deque 实现没有严格要求禁止插入 null 元素，但建议最好这样做。建议任何事实上允许 null 元素的 Deque 实现用户最好不 要利用插入 null 的功能。这是因为各种方法会将 null 用作特殊的返回值来指示双端队列为空。

   &nbsp;&nbsp; Deque 实现通常不定义基于元素的 equals 和 hashCode 方法，而是从 Object 类继承基于身份的 equals 和 hashCode 方法。

   &nbsp;&nbsp; 此接口是 Java Collections Framework 的成员。
 
```java
  public interface Deque<E> extends Queue<E> {
    
  }
```
