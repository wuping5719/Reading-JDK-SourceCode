* Collection类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Collection层次结构中的根接口。Collection表示一组对象，这些对象也称为collection的元素。一些collection允许有重复的元素，
  而另一些则不允许。一些collection是有序的，而另一些则是无序的。JDK不提供此接口的任何直接实现：它提供更具体的子接口（如Set和List）实现。
  此接口通常用来传递collection，并在需要最大普遍性的地方操作这些collection。
  包 (bag) 或多集合 (multiset)（可能包含重复元素的无序 collection）应该直接实现此接口。
  
  &nbsp;&nbsp; 所有通用的Collection实现类（通常通过它的一个子接口间接实现Collection）应该提供两个“标准”构造方法：
  一个是void（无参数）构造方法，用于创建空collection；
  另一个是带有Collection类型单参数的构造方法，用于创建一个具有与其参数相同元素新的collection。实际上，
  后者允许用户复制任何collection，以生成所需实现类型的一个等效collection。
  尽管无法强制执行此约定（因为接口不能包含构造方法），但是Java平台库中所有通用的Collection实现都遵从它。
  
  &nbsp;&nbsp; 此接口中包含的“破坏性”方法，是指可修改其所操作的collection的那些方法，如果此collection不支持该操作，
  则指定这些方法抛出UnsupportedOperationException。如果是这样，那么在调用对该collection无效时，这些方法可能，
  但并不一定抛出UnsupportedOperationException。例如，如果要添加的collection为空且不可修改，
  则对该collection调用addAll(Collection)方法时，可能但并不一定抛出异常。
  
  &nbsp;&nbsp; 一些collection实现对它们可能包含的元素有所限制。例如，某些实现禁止null元素，而某些实现则对元素的类型有限制。
  试图添加不合格的元素将抛出一个未经检查的异常，通常是NullPointerException或ClassCastException。
  试图查询是否存在不合格的元素可能抛出一个异常，或者只是简单地返回false；
  某些实现将表现出前一种行为，而某些实现则表现后一种。
  较为常见的是，试图对某个不合格的元素执行操作且该操作的完成不会导致将不合格的元素插入collection中，将可能抛出一个异常，
  也可能操作成功，这取决于实现本身。这样的异常在此接口的规范中标记为“可选”。
  
  &nbsp;&nbsp; 由每个collection来确定其自身的同步策略。在没有实现的强烈保证的情况下，调用由另一进程正在更改的collection的方法可能会出现不确定行为；
  这包括直接调用，将collection传递给可能执行调用的方法，以及使用现有迭代器检查collection。
  
  &nbsp;&nbsp; Collections Framework接口中的很多方法是根据equals方法定义的。
  例如，contains(Object o)方法的规范声明：“当且仅当此collection包含至少一个满足(o==null ? e==null :o.equals(e))的元素e时，返回 true。”
  不应将此规范理解为它暗指调用具有非空参数o的Collection.contains方法会导致为任意的e元素调用o.equals(e)方法。
  可随意对各种实现执行优化，只要避免调用equals即可，例如，通过首先比较两个元素的哈希码。
  （Object.hashCode() 规范保证哈希码不相等的两个对象不会相等）。较为常见的是，
  各种Collections Framework接口的实现可随意利用底层Object方法的指定行为，而不管实现程序认为它是否合适。
  此接口是Java Collections Framework的一个成员。
  
  <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_Collections.gif" /></p>
  上述类图中，实线边框的是实现类，比如ArrayList，LinkedList，HashMap等；折线边框的是抽象类，比如AbstractCollection，AbstractList，AbstractMap等；而点线边框的是接口，比如Collection，Iterator，List等。

```java
  public interface Collection<E> extends Iterable<E> {
     //返回此collection中的元素数。如果此collection包含的元素大于Integer.MAX_VALUE，则返回Integer.MAX_VALUE。
     int size();
     
     //如果此collection不包含元素，则返回true。
     boolean isEmpty();
     
     //如果此collection包含指定的元素，则返回true。
     //更确切地讲，当且仅当此collection至少包含一个满足(o==null ? e==null : o.equals(e))的元素e时，返回true。
     boolean contains(Object o);
     
     //返回在此collection的元素上进行迭代的迭代器。
     //关于元素返回的顺序没有任何保证（除非此collection是某个能提供保证顺序的类实例）。
     Iterator<E> iterator();
     
     //返回包含此collection中所有元素的数组。
     //如果collection对其迭代器返回的元素顺序做出了某些保证，那么此方法必须以相同的顺序返回这些元素。
     //返回的数组将是“安全的”，因为此collection并不维护对返回数组的任何引用。
     //（换句话说，即使collection受到数组的支持，此方法也必须分配一个新的数组）。因此，调用者可以随意修改返回的数组。
     //此方法充当了基于数组的API与基于collection的API之间的桥梁。
     Object[] toArray();
     
     //返回包含此collection中所有元素的数组；返回数组的运行时类型与指定数组的运行时类型相同。
     //如果指定的数组能容纳该collection，则返回包含此collection元素的数组。
     //否则，将分配一个具有指定数组的运行时类型和此collection大小的新数组。
     //如果指定的数组能容纳collection，并有剩余空间（即数组的元素比collection的元素多），
     //那么会将数组中紧接collection尾部的元素设置为null。
     //（只有在调用者知道此collection没有包含任何null元素时才能用此方法确定collection的长度。）
     //如果此collection对其迭代器返回的元素顺序做出了某些保证，那么此方法必须以相同的顺序返回这些元素。
     //像toArray()方法一样，此方法充当基于数组的API与基于collection的API之间的桥梁。
     //更进一步说，此方法允许对输出数组的运行时类型进行精确控制，并且在某些情况下，可以用来节省分配开销。
     //假定x是只包含字符串的一个已知collection。以下代码用来将collection转储到一个新分配的String数组：
     //  String[] y = x.toArray(new String[0]); 
     //注意: toArray(new Object[0])和toArray()在功能上是相同的。
     <T> T[] toArray(T[] a);
     
     //确保此collection包含指定的元素（可选操作）。如果此collection由于调用而发生更改，则返回true。
     //（如果此collection不允许有重复元素，并且已经包含了指定的元素，则返回false。）
     //支持此操作的collection可以限制哪些元素能添加到此collection中来。
     //需要特别指出的是，一些collection拒绝添加null元素，其他一些collection将对可以添加的元素类型强加限制。
     //Collection类应该在其文档中清楚地指定能添加哪些元素方面的所有限制。
     //如果collection由于某些原因（已经包含该元素的原因除外）拒绝添加特定的元素，那么它必须抛出一个异常（而不是返回 false）。
     //这确保了在此调用返回后，collection总是包含指定的元素。
     boolean add(E e);
     
     //从此collection中移除指定元素的单个实例，如果存在的话（可选操作）。
     //更确切地讲，如果此collection包含一个或多个满足(o==null ? e==null : o.equals(e))的元素e，则移除这样的元素。
     //如果此collection包含指定的元素（或者此collection由于调用而发生更改），则返回true 。
     boolean remove(Object o);
     
     //如果此collection包含指定collection中的所有元素，则返回true。
     boolean containsAll(Collection<?> c);
     
     //将指定collection中的所有元素都添加到此collection中（可选操作）。
     //如果在进行此操作的同时修改指定的collection，那么此操作行为是不确定的。
     //（这意味着如果指定的collection是此collection，并且此collection为非空，那么此调用的行为是不确定的。）
     boolean addAll(Collection<? extends E> c);
     
     //移除此collection中那些也包含在指定collection中的所有元素（可选操作）。
     //此调用返回后，collection中将不包含任何与指定collection相同的元素。
     boolean removeAll(Collection<?> c);
     
     //仅保留此collection中那些也包含在指定collection的元素（可选操作）。
     //换句话说，移除此collection中未包含在指定collection中的所有元素。
     boolean retainAll(Collection<?> c);
     
     //移除此collection中的所有元素（可选操作）。此方法返回void，除非抛出一个异常。
     void clear();
     
     //比较此collection与指定对象是否相等。
     //当Collection接口没有对Object.equals的常规协定添加任何约定时，“直接”实现该Collection接口
     //（换句话说，创建一个Collection，但它不是Set或List的类）的程序员选择重写Object.equals方法时必须小心。
     //没必要这样做，最简单的方案是依靠Object的实现，然而实现者可能希望实现“值比较”，而不是默认的“引用比较”。
     //（List和Set接口要求进行这样的值比较。）
     //Object.equals方法的常规协定声称相等必须是对称的（换句话说，当且仅当存在b.equals(a)时，才存在a.equals(b)）。
     //List.equals和Set.equals的协定声称列表只能与列表相等，set只能与set相等。
     //因此，对于一个既不实现List又不实现Set接口的collection类，
     //当将此collection与任何列表或set进行比较时，常规的equals方法必须返回false。
     //（按照相同的逻辑，不可能编写一个同时正确实现Set和List接口的类。）
     boolean equals(Object o);
     
     //返回此collection的哈希码值。当Collection接口没有为Object.hashCode方法的常规协定添加任何约束时，
     //为了满足Object.hashCode方法的常规协定，程序员应该注意任何重写Object.equals方法的类必须重写Object.hashCode方法。
     //需要特别指出的是， c1.equals(c2)暗示着c1.hashCode()==c2.hashCode()。
     int hashCode();
  }
```
