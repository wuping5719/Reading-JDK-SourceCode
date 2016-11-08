* Deque接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
   &nbsp;&nbsp; 一个线性 collection，支持在两端插入和移除元素。名称 deque 是“double ended queue（双端队列）”的缩写，通常读为“deck”。大多数 Deque 实现对于它们能够包含的元素数没有固定限制，但此接口既支持有容量限制的双端队列，也支持没有固定大小限制的双端队列。

   &nbsp;&nbsp; 此接口定义在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或 false，具体取决于操作）。插入操作的后一种形式是专为使用有容量限制的 Deque 实现设计的；在大多数实现中，插入操作不能失败。

   &nbsp;&nbsp; 下表总结了上述 12 种方法：    
   &nbsp;&nbsp;&nbsp;&nbsp; 第一个元素（头部）	 最后一个元素（尾部）        
   &nbsp;&nbsp;&nbsp;&nbsp; 抛出异常	特殊值	   抛出异常	特殊值    
   &nbsp;&nbsp;&nbsp;&nbsp; 插入	addFirst(e)	offerFirst(e)	addLast(e)	offerLast(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; 移除	removeFirst()	pollFirst()	removeLast()	pollLast()     
   &nbsp;&nbsp;&nbsp;&nbsp; 检查	getFirst()	peekFirst()	getLast()	peekLast()     
   
   &nbsp;&nbsp; 此接口扩展了 Queue 接口。在将双端队列用作队列时，将得到 FIFO（先进先出）行为。将元素添加到双端队列的末尾，从双端队列的开头移除元素。从 Queue 接口继承的方法完全等效于 Deque 方法，如下表所示：     
   &nbsp;&nbsp; Queue 方法	等效 Deque 方法     
   &nbsp;&nbsp;&nbsp;&nbsp; add(e)	addLast(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; offer(e)	offerLast(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; remove()	removeFirst()      
   &nbsp;&nbsp;&nbsp;&nbsp; poll()	pollFirst()     
   &nbsp;&nbsp;&nbsp;&nbsp; element()	getFirst()     
   &nbsp;&nbsp;&nbsp;&nbsp; peek()	peekFirst()    
   
   &nbsp;&nbsp; 双端队列也可用作 LIFO（后进先出）堆栈。应优先使用此接口而不是遗留 Stack 类。在将双端队列用作堆栈时，元素被推入双端队列的开头并从双端队列开头弹出。堆栈方法完全等效于 Deque 方法，如下表所示：      
   &nbsp;&nbsp; 堆栈方法	等效 Deque 方法    
   &nbsp;&nbsp;&nbsp;&nbsp; push(e)	addFirst(e)    
   &nbsp;&nbsp;&nbsp;&nbsp; pop()	removeFirst()   
   &nbsp;&nbsp;&nbsp;&nbsp; peek()	peekFirst()    
   &nbsp;&nbsp; 注意：在将双端队列用作队列或堆栈时，peek 方法同样正常工作；无论哪种情况下，都从双端队列的开头抽取元素。

   &nbsp;&nbsp; 此接口提供了两种移除内部元素的方法：removeFirstOccurrence 和 removeLastOccurrence。

   &nbsp;&nbsp; 与 List 接口不同，此接口不支持通过索引访问元素。

   &nbsp;&nbsp; 虽然 Deque 实现没有严格要求禁止插入 null 元素，但建议最好这样做。建议任何事实上允许 null 元素的 Deque 实现用户最好不要利用插入 null 的功能。这是因为各种方法会将 null 用作特殊的返回值来指示双端队列为空。

   &nbsp;&nbsp; Deque 实现通常不定义基于元素的 equals 和 hashCode 方法，而是从 Object 类继承基于身份的 equals 和 hashCode 方法。

   &nbsp;&nbsp; 此接口是 Java Collections Framework 的成员。
 
```java
  public interface Deque<E> extends Queue<E> {
     //将指定元素插入此双端队列的开头（如果可以直接这样做而不违反容量限制）。
     //在使用有容量限制的双端队列时，通常首选 offerFirst(E) 方法。
     void addFirst(E e);
     
     //将指定元素插入此双端队列的末尾（如果可以直接这样做而不违反容量限制）。
     //在使用有容量限制的双端队列时，通常首选 offerLast(E) 方法。
     //此方法等效于 add(E)。
     void addLast(E e);
     
     //在不违反容量限制的情况下，将指定的元素插入此双端队列的开头。
     //当使用有容量限制的双端队列时，此方法通常优于 addFirst(E) 方法，后者可能无法插入元素，而只是抛出一个异常。
     boolean offerFirst(E e);
     
     //在不违反容量限制的情况下，将指定的元素插入此双端队列的末尾。
     //当使用有容量限制的双端队列时，此方法通常优于 addLast(E) 方法，后者可能无法插入元素，而只是抛出一个异常。
     boolean offerLast(E e);
     
     //获取并移除此双端队列第一个元素。此方法与 pollFirst 唯一的不同在于：如果此双端队列为空，它将抛出一个异常。
     E removeFirst();
     
     //获取并移除此双端队列的最后一个元素。此方法与 pollLast 唯一的不同在于：如果此双端队列为空，它将抛出一个异常。
     E removeLast();
     
     //获取并移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。
     E pollFirst();
     
     //获取并移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。
     E pollLast();
     
     //获取，但不移除此双端队列的第一个元素。 此方法与 peekFirst 唯一的不同在于：如果此双端队列为空，它将抛出一个异常。
     E getFirst();
     
     //获取，但不移除此双端队列的最后一个元素。此方法与 peekLast 唯一的不同在于：如果此双端队列为空，它将抛出一个异常。
     E getLast();
     
     //获取，但不移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。
     E peekFirst();
     
     //获取，但不移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。
     E peekLast();
     
     //从此双端队列移除第一次出现的指定元素。如果此双端队列不包含该元素，则不作更改。
     //更确切地讲，移除第一个满足 (o==null ? e==null : o.equals(e)) 的元素 e（如果存在这样的元素）。
     //如果此双端队列包含指定的元素（或者此双端队列由于调用而发生了更改），则返回 true。
     boolean removeFirstOccurrence(Object o);
     
     //从此双端队列移除最后一次出现的指定元素。如果此双端队列不包含该元素，则不作更改。
     //更确切地讲，移除最后一个满足 (o==null ? e==null : o.equals(e)) 的元素 e（如果存在这样的元素）。
     //如果此双端队列包含指定的元素（或者此双端队列由于调用而发生了更改），则返回 true。
     boolean removeLastOccurrence(Object o);
     
     //将指定元素插入此双端队列所表示的队列（换句话说，此双端队列的尾部），如果可以直接这样做而不违反容量限制的话；
     //如果成功，则返回 true，如果当前没有可用空间，则抛出 IllegalStateException。
     //当使用有容量限制的双端队列时，通常首选 offer。
     //此方法等效于 addLast(E)。
     boolean add(E e);
     
     //将指定元素插入此双端队列所表示的队列（换句话说，此双端队列的尾部），如果可以直接这样做而不违反容量限制的话；
     //如果成功，则返回 true，如果当前没有可用的空间，则返回 false。
     //当使用有容量限制的双端队列时，此方法通常优于 add(E) 方法，后者可能无法插入元素，而只是抛出一个异常。
     //此方法等效于 offerLast(E)。
     boolean offer(E e);
     
     //获取并移除此双端队列所表示的队列的头部（换句话说，此双端队列的第一个元素）。
     //此方法与 poll 的唯一不同在于：如果此双端队列为空，它将抛出一个异常。
     //此方法等效于 removeFirst()。
     E remove();
     
     //获取并移除此双端队列所表示的队列的头部（换句话说，此双端队列的第一个元素）；
     //如果此双端队列为空，则返回 null。
     //此方法等效于 pollFirst()。
     E poll();
     
     //获取，但不移除此双端队列所表示的队列的头部（换句话说，此双端队列的第一个元素）。
     //此方法与 peek 唯一的不同在于：如果此双端队列为空，它将抛出一个异常。
     //此方法等效于 getFirst()。
     E element();
     
     //获取，但不移除此双端队列所表示的队列的头部（换句话说，此双端队列的第一个元素）；
     //如果此双端队列为空，则返回 null。
     //此方法等效于 peekFirst()。
     E peek();
     
     //将一个元素推入此双端队列所表示的堆栈（换句话说，此双端队列的头部），如果可以直接这样做而不违反容量限制的话；
     //如果成功，则返回 true，如果当前没有可用空间，则抛出 IllegalStateException。
     //此方法等效于 addFirst(E)。
     void push(E e);
     
     //从此双端队列所表示的堆栈中弹出一个元素。换句话说，移除并返回此双端队列第一个元素。
     //此方法等效于 removeFirst()。
     E pop();
     
     //从此双端队列中移除第一次出现的指定元素。如果此双端队列不包含该元素，则不作更改。
     //更确切地讲，移除第一个满足 (o==null ? e==null : o.equals(e)) 的元素 e（如果存在这样的元素）。
     //如果此双端队列包含指定的元素（或者此双端队列由于调用而发生了更改），则返回 true。
     //此方法等效于 removeFirstOccurrence(java.lang.Object)。
     boolean remove(Object o);
     
     //如果此双端队列包含指定元素，则返回 true。
     //更确切地讲，当且仅当此双端队列至少包含一个满足 (o==null ? e==null : o.equals(e)) 的元素 e 时，返回 true。
     boolean contains(Object o);
     
     //返回此双端队列的元素数。
     public int size();
     
     //返回以恰当顺序在此双端队列的元素上进行迭代的迭代器。元素将按从第一个（头部）到最后一个（尾部）的顺序返回。
     Iterator<E> iterator();
     
     //返回以逆向顺序在此双端队列的元素上进行迭代的迭代器。元素将按从最后一个（尾部）到第一个（头部）的顺序返回。
     Iterator<E> descendingIterator();
  }
```
