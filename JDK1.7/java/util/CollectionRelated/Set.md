* Set接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 一个不包含重复元素的 collection。更确切地讲，set 不包含满足 e1.equals(e2) 的元素对 e1 和 e2，并且最多包含一个 null 元素。正如其名称所暗示的，此接口模仿了数学上的 set 抽象。

  &nbsp;&nbsp; 在所有构造方法以及 add、equals 和 hashCode 方法的协定上，Set 接口还加入了其他规定，这些规定超出了从 Collection 接口所继承的内容。出于方便考虑，它还包括了其他继承方法的声明（这些声明的规范已经专门针对 Set 接口进行了修改，但是没有包含任何其他的规定）。

  &nbsp;&nbsp; 对这些构造方法的其他规定是（不要奇怪），所有构造方法必须创建一个不包含重复元素的 set（正如上面所定义的）。

  &nbsp;&nbsp; 注：如果将可变对象用作 set 元素，那么必须极其小心。如果对象是 set 中某个元素，以一种影响 equals 比较的方式改变对象的值，那么 set 的行为就是不确定的。此项禁止的一个特殊情况是不允许某个 set 包含其自身作为元素。

  &nbsp;&nbsp; 某些 set 实现对其所包含的元素有所限制。例如，某些实现禁止 null 元素，而某些则对其元素的类型所有限制。试图添加不合格的元素会抛出未经检查的异常，通常是 NullPointerException 或 ClassCastException。试图查询不合格的元素是否存在可能会抛出异常，也可能简单地返回 false；某些实现会采用前一种行为，而某些则采用后者。概括地说，试图对不合格元素执行操作时，如果完成该操作后不会导致在 set 中插入不合格的元素，则该操作可能抛出一个异常，也可能成功，这取决于实现的选择。此接口的规范中将这样的异常标记为“可选”。

  &nbsp;&nbsp; 此接口是 Java Collections Framework 的成员。
 
```java
  public interface Set<E> extends Collection<E> {
    //返回 set 中的元素数（其容量）。如果 set 包含超过 Integer.MAX_VALUE 个元素，则返回 Integer.MAX_VALUE。
    int size();
    
    //如果 set 不包含元素，则返回 true。
    boolean isEmpty();
    
    //如果 set 包含指定的元素，则返回 true。
    //更确切地讲，当且仅当 set 包含满足 (o==null ? e==null : o.equals(e)) 的元素 e 时返回 true。
    boolean contains(Object o);
    
    //返回在此 set 中的元素上进行迭代的迭代器。返回的元素没有特定的顺序（除非此 set 是某个提供顺序保证的类的实例）。
    Iterator<E> iterator();
    
    //返回一个包含 set 中所有元素的数组。
    //如果此 set 对其迭代器返回的元素的顺序作出了某些保证，那么此方法也必须按相同的顺序返回这些元素。
    //由于此 set 不维护对返回数组的任何引用，因而它是安全的。
    //（换句话说，即使此 set 受到数组的支持，此方法也必须分配一个新的数组）。因此，调用者可以随意修改返回的数组。
    //此方法充当基于数组的 API 与基于 collection 的 API 之间的桥梁。
    Object[] toArray();
    
    //返回一个包含此 set 中所有元素的数组；返回数组的运行时类型是指定数组的类型。
    //如果指定的数组能容纳该 set，则它将在其中返回。否则，将分配一个具有指定数组的运行时类型和此 set 大小的新数组。
    //如果指定的数组能容纳此set,并有剩余的空间(即该数组的元素比此set多),那么会将列表中紧接该set尾部的元素设置为null
    //（只有在调用者知道此 set 不包含任何 null 元素时才能用此方法确定此 set 的长度）。
    //如果此 set 对其迭代器返回的元素的顺序作出了某些保证，那么此方法也必须按相同的顺序返回这些元素。
    //像 toArray() 方法一样，此方法充当基于数组的 API 与基于 collection 的 API 之间的桥梁。
    //更进一步说，此方法允许对输出数组的运行时类型上进行精确控制，在某些情况下，可以用来节省分配开销。
    //假定 x 是只包含字符串的一个已知 set。以下代码用来将该 set 转储到一个新分配的 String 数组：
    //   String[] y = x.toArray(new String[0]);
    //注意： toArray(new Object[0]) 和 toArray() 在功能上是相同的。
    <T> T[] toArray(T[] a);
    
    //如果 set 中尚未存在指定的元素，则添加此元素（可选操作）。
    //更确切地讲，如果此set没有包含满足(e==null ? e2==null : e.equals(e2))的元素e2，则向该set中添加指定的元素e
    //如果此set已经包含该元素，则该调用不改变此set并返回false。结合构造方法上的限制，这就可以确保set永远不包含重复的元素
    //上述规定并未暗示 set 必须接受所有元素；set 可以拒绝添加任意特定的元素，包括 null，并抛出异常，
    //这与 Collection.add 规范中所描述的一样。每个 set 实现应该明确地记录对其可能包含元素的所有限制。
    boolean add(E e);
    
    //如果 set 中存在指定的元素，则将其移除（可选操作）。
    //更确切地讲，如果此 set 中包含满足 (o==null ? e==null : o.equals(e)) 的元素 e，则移除它。
    //如果此set包含指定的元素（或者此set由于调用而发生更改），则返回true（一旦调用返回，则此set不再包含指定的元素）。
    boolean remove(Object o);
    
    //如果此 set 包含指定 collection 的所有元素，则返回 true。
    //如果指定的 collection 也是一个 set，那么当该 collection 是此 set 的子集时返回 true。
    boolean containsAll(Collection<?> c);
    
    //如果 set 中没有指定 collection 中的所有元素，则将其添加到此 set 中（可选操作）。
    //如果指定的 collection 也是一个 set，则 addAll 操作会实际修改此 set，这样其值是两个 set 的一个并集。
    //如果操作正在进行的同时修改了指定的 collection，则此操作的行为是不确定的。
    boolean addAll(Collection<? extends E> c);
    
    //仅保留 set 中那些包含在指定 collection 中的元素（可选操作）。
    //换句话说，移除此 set 中所有未包含在指定 collection 中的元素。
    //如果指定的 collection 也是一个 set，则此操作会实际修改此 set，这样其值是两个 set 的一个交集。
    boolean retainAll(Collection<?> c);
    
    //移除 set 中那些包含在指定 collection 中的元素（可选操作）。
    //如果指定的 collection 也是一个 set，则此操作会实际修改此 set，这样其值是两个 set 的一个不对称差集。
    boolean removeAll(Collection<?> c);
    
    //移除此 set 中的所有元素（可选操作）。此调用返回后该 set 将是空的。
    void clear();
    
    //比较指定对象与此 set 的相等性。
    //如果指定的对象也是一个 set，两个 set 的大小相同，
    //并且指定set的所有成员都包含在此set中（或者，此set的所有成员都包含在指定的set中也一样），则返回true。
    //此定义确保了 equals 方法可在不同的 set 接口实现间正常工作。
    boolean equals(Object o);
    
    //返回 set 的哈希码值。一个set的哈希码定义为此set中所有元素的哈希码和，其中null元素的哈希码定义为零。
    //这就确保对于任意两个 set s1 和 s2 而言， s1.equals(s2) 就意味着 s1.hashCode()==s2.hashCode()，
    //正如 Object.hashCode() 的常规协定所要求的那样。
    int hashCode();
  }
```
