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

```java

```
