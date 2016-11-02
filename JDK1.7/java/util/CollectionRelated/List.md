* List类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp; &nbsp;&nbsp; 有序的collection（也称为序列）。此接口的用户可以对列表中每个元素的插入位置进行精确地控制。用户可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。
  
  &nbsp; &nbsp;&nbsp; 与 set 不同，列表通常允许重复的元素。更确切地讲，列表通常允许满足 e1.equals(e2) 的元素对 e1 和 e2，并且如果列表本身允许 null 元素的话，通常它们允许多个 null 元素。难免有人希望通过在用户尝试插入重复元素时抛出运行时异常的方法来禁止重复的列表，但我们希望这种用法越少越好。
  
  &nbsp; &nbsp;&nbsp; List 接口在 iterator、add、remove、equals 和 hashCode 方法的协定上加了一些其他约定，超过了 Collection 接口中指定的约定。为方便起见，这里也包括了其他继承方法的声明。

  &nbsp; &nbsp;&nbsp; List 接口提供了 4 种对列表元素进行定位（索引）访问方法。列表（像 Java 数组一样）是基于 0 的。注意，这些操作可能在和某些实现（例如 LinkedList 类）的索引值成比例的时间内执行。因此，如果调用者不知道实现，那么在列表元素上迭代通常优于用索引遍历列表。

  &nbsp; &nbsp;&nbsp; List 接口提供了特殊的迭代器，称为 ListIterator，除了允许 Iterator 接口提供的正常操作外，该迭代器还允许元素插入和替换，以及双向访问。还提供了一个方法来获取从列表中指定位置开始的列表迭代器。

  &nbsp; &nbsp;&nbsp; List 接口提供了两种搜索指定对象的方法。从性能的观点来看，应该小心使用这些方法。在很多实现中，它们将执行高开销的线性搜索。

  &nbsp; &nbsp;&nbsp; List 接口提供了两种在列表的任意位置高效插入和移除多个元素的方法。

  &nbsp; &nbsp;&nbsp; 注意：尽管列表允许把自身作为元素包含在内，但建议要特别小心：在这样的列表上，equals 和 hashCode 方法不再是定义良好的。

  &nbsp; &nbsp;&nbsp; 某些列表实现对列表可能包含的元素有限制。例如，某些实现禁止 null 元素，而某些实现则对元素的类型有限制。试图添加不合格的元素会抛出未经检查的异常，通常是 NullPointerException 或 ClassCastException。试图查询不合格的元素是否存在可能会抛出异常，也可能简单地返回 false；某些实现会采用前一种行为，而某些则采用后者。概括地说，试图对不合格元素执行操作时，如果完成该操作后不会导致在列表中插入不合格的元素，则该操作可能抛出一个异常，也可能成功，这取决于实现的选择。此接口的规范中将这样的异常标记为“可选”。

  &nbsp; &nbsp;&nbsp; 此接口是Java Collections Framework的成员。
 
```java
  public interface List<E> extends Collection<E> {
     //返回列表中的元素数。如果列表包含多于Integer.MAX_VALUE个元素，则返回Integer.MAX_VALUE
     int size();
     
     //如果列表不包含元素，则返回true
     boolean isEmpty();
     
     //如果列表包含指定的元素，则返回true。
     //更确切地讲，当且仅当列表包含满足(o==null ? e==null : o.equals(e))的元素e时才返回true
     boolean contains(Object o);
     
     //返回按适当顺序在列表的元素上进行迭代的迭代器
     Iterator<E> iterator();
     
     //返回按适当顺序包含列表中的所有元素的数组（从第一个元素到最后一个元素）
     Object[] toArray();
     
     //返回按适当顺序（从第一个元素到最后一个元素）包含列表中所有元素的数组；
     //返回数组的运行时类型是指定数组的运行时类型。如果指定数组能容纳列表，则在其中返回该列表。
     //否则，分配具有指定数组的运行时类型和此列表大小的新数组。
     //如果指定数组能容纳列表，并剩余空间（即数组的元素比列表的多），那么会将数组中紧随列表尾部的元素设置为null。
     //（只有在调用者知道列表不包含任何null元素时此方法才能用于确定列表的长度）。
     //像 toArray() 方法一样，此方法充当基于数组的API与基于collection的API之间的桥梁。
     //更进一步说，此方法允许对输出数组的运行时类型进行精确控制，在某些情况下，可以用来节省分配开销。
     //假定 x 是只包含字符串的一个已知列表。以下代码用来将该列表转储到一个新分配的 String 数组：
     //String[] y = x.toArray(new String[0]);
     //注意， toArray(new Object[0]) 和 toArray() 在功能上是相同的。
     <T> T[] toArray(T[] a);
    
     //向列表的尾部添加指定的元素（可选操作）。
     //支持该操作的列表可能对列表可以添加的元素有一些限制。
     //特别是某些列表将拒绝添加 null 元素，其他列表将在可能添加的元素类型上施加限制。
     //List 类应该在它们的文档中明确指定有关添加元素的所有限制。
     boolean add(E e);
     
     //从此列表中移除第一次出现的指定元素（如果存在）（可选操作）。
     //如果列表不包含元素，则不更改列表。
     //更确切地讲，移除满足(o==null ? get(i)==null : o.equals(get(i)))的最低索引i的元素（如果存在这样的元素）。
     //如果此列表已包含指定元素（或者此列表由于调用而发生更改），则返回true。
     boolean remove(Object o);
     
     //如果列表包含指定collection的所有元素，则返回true。
     boolean containsAll(Collection<?> c);
     
     //添加指定collection中的所有元素到此列表的结尾，顺序是指定collection的迭代器返回这些元素的顺序（可选操作）。
     //如果在操作正在进行中修改了指定的collection，那么此操作的行为是不确定的.
     //（注意，如果指定的collection是此列表，并且它是非空的，则会发生这种情况）
     boolean addAll(Collection<? extends E> c);
     
     //将指定collection中的所有元素都插入到列表中的指定位置（可选操作）。
     //将当前处于该位置的元素（如果有的话）和所有后续元素向右移动（增加其索引）。
     //新元素将按照它们通过指定collection的迭代器所返回的顺序出现在此列表中。
     //如果在操作正在进行中修改了指定的collection，那么该操作的行为是不确定的.
     //（注意，如果指定的collection是此列表，并且它是非空的，则会发生这种情况）
     boolean addAll(int index, Collection<? extends E> c);
     
     //从列表中移除指定 collection 中包含的其所有元素（可选操作）
     boolean removeAll(Collection<?> c);
     
     //仅在列表中保留指定 collection 中所包含的元素（可选操作）。
     //换句话说，该方法从列表中移除未包含在指定 collection 中的所有元素。
     boolean retainAll(Collection<?> c);
     
     //从列表中移除所有元素（可选操作）。此调用返回后该列表将是空的。
     void clear();
     
     //比较指定的对象与列表是否相等。当且仅当指定的对象也是一个列表、两个列表有相同的大小，
     //并且两个列表中的所有相应的元素对 相等 时才返回 true（ 如果 (e1==null ? e2==null :e1.equals(e2))，则两个元素 e1 和 e2 是 相等 的）   
     //换句话说，如果所定义的两个列表以相同的顺序包含相同的元素，那么它们是相等的。该定义确保了 equals 方法在 List 接口的不同实现间正常工作。
     boolean equals(Object o);
     
     //返回列表的哈希码值。列表的哈希码定义为以下计算的结果：
     //    int hashCode = 1;
     //    Iterator<E> i = list.iterator();
     //    while (i.hasNext()) {
     //        E obj = i.next();
     //        hashCode = 31*hashCode + (obj==null ? 0 : obj.hashCode());
     //    }
     //这确保了 list1.equals(list2) 意味着对于任何两个列表 list1 和 list2 而言，可实现 list1.hashCode()==list2.hashCode()，
     //正如 Object.hashCode() 的常规协定所要求的。
     int hashCode();
     
     //返回列表中指定位置的元素。
     E get(int index);
     
     //用指定元素替换列表中指定位置的元素（可选操作）。
     E set(int index, E element);
     
     //在列表的指定位置插入指定元素（可选操作）。将当前处于该位置的元素（如果有的话）和所有后续元素向右移动（在其索引中加 1）。
     void add(int index, E element);
     
     //移除列表中指定位置的元素（可选操作）。将所有的后续元素向左移动（将其索引减 1）。返回从列表中移除的元素。
     E remove(int index);
     
     //返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1。
     //更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最低索引 i；如果没有这样的索引，则返回 -1。
     int indexOf(Object o);
     
     //返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1。
     //更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最高索引 i；如果没有这样的索引，则返回 -1。
     int lastIndexOf(Object o);
     
     //返回此列表元素的列表迭代器（按适当顺序）。
     ListIterator<E> listIterator();
     
     //返回列表中元素的列表迭代器（按适当顺序），从列表的指定位置开始。
     //指定的索引表示 next 的初始调用所返回的第一个元素。 previous 方法的初始调用将返回索引比指定索引少 1 的元素。
     ListIterator<E> listIterator(int index);
     
     //返回列表中指定的 fromIndex（包括 ）和 toIndex（不包括）之间的部分视图。（如果 fromIndex 和 toIndex 相等，则返回的列表为空）。
     //返回的列表由此列表支持，因此返回列表中的非结构性更改将反映在此列表中，反之亦然。返回的列表支持此列表支持的所有可选列表操作。
     //此方法省去了显式范围操作（此操作通常针对数组存在）。通过传递 subList 视图而非整个列表，期望列表的任何操作可用作范围操作。
     //例如，下面的语句从列表中移除了元素的范围：
     //     list.subList(from, to).clear();
     //可以对 indexOf 和 lastIndexOf 构造类似的语句，而且 Collections 类中的所有算法都可以应用于 subList。
     //如果支持列表（即此列表）通过任何其他方式（而不是通过返回的列表）从结构上修改，
     //则此方法返回的列表语义将变为未定义（从结构上修改是指更改列表的大小，或者以其他方式打乱列表，使正在进行的迭代产生错误的结果）。
     //参数：
     //    fromIndex - subList 的低端（包括）
     //    toIndex - subList 的高端（不包括）
     List<E> subList(int fromIndex, int toIndex);
  }
```
