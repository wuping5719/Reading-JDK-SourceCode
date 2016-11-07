* AbstractList抽象类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  AbstractList抽象类提供 List 接口的骨干实现，以最大限度地减少实现“随机访问”数据存储（如数组）支持的该接口所需的工作。对于连续的访问数据（如链表），应优先使用 AbstractSequentialList，而不是此类。

  要实现不可修改的列表，编程人员只需扩展此类，并提供 get(int) 和 size() 方法的实现。

  要实现可修改的列表，编程人员必须另外重写 set(int, E) 方法（否则将抛出 UnsupportedOperationException）。如果列表为可变大小，则编程人员必须另外重写 add(int, E) 和 remove(int) 方法。

  按照 Collection 接口规范中的建议，编程人员通常应该提供一个 void（无参数）和 collection 构造方法。

  与其他抽象 collection 实现不同，编程人员不必提供迭代器实现；迭代器和列表迭代器由此类在以下“随机访问”方法上实现：get(int)、set(int, E)、add(int, E) 和 remove(int)。

  此类中每个非抽象方法的文档详细描述了其实现。如果要实现的 collection 允许更有效的实现，则可以重写所有这些方法。

  此类是Java Collections Framework的成员。
 
```java
  public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
     //唯一的构造方法。（由子类构造方法调用，通常是隐式的）
     protected AbstractList() {}
     
     //将指定的元素添加到此列表的尾部（可选操作）。
     //支持此操作的列表可能对列表可以添加的元素做出一些限制。具体来说，某些列表将拒绝添加 null 元素，另一些列表将在可以添加的元素类型上施加限制。
     //List 类应该在它们的文档中明确指定可以添加何种元素的所有限制。
     //此实现调用 add(size(), e)。  注意：除非重写 add(int, E)，否则此实现将抛出 UnsupportedOperationException。
     public boolean add(E e) {
        add(size(), e);
        return true;
     }
     
     //返回列表中指定位置的元素。
     abstract public E get(int index);
     
     //用指定元素替换列表中指定位置的元素（可选操作）。此实现始终抛出 UnsupportedOperationException。
     public E set(int index, E element) {
        throw new UnsupportedOperationException();
     }
     
     //在列表的指定位置插入指定元素（可选操作）。将当前处于该位置的元素（如果有的话）和所有后续元素向右移动（在其索引中加 1）。
     //此实现始终抛出 UnsupportedOperationException。
     public void add(int index, E element) {
        throw new UnsupportedOperationException();
     }
     
     //移除列表中指定位置的元素（可选操作）。将所有的后续元素向左移动（将其索引减 1）。返回从列表中移除的元素。
     //此实现始终抛出 UnsupportedOperationException。
     public E remove(int index) {
        throw new UnsupportedOperationException();
     }
     
     //返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1。
     //更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最低索引 i；如果没有这样的索引，则返回 -1。
     //此实现首先获取一个列表迭代器（使用 listIterator()）。然后它迭代列表，直至找到指定的元素，或者到达列表的末尾。
     public int indexOf(Object o) {
        ListIterator<E> it = listIterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return it.previousIndex();
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return it.previousIndex();
        }
        return -1;
     }
     
     //返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1。
     //更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最高索引 i；如果没有这样的索引，则返回 -1。
     //此实现首先获取一个指向列表末尾的列表迭代器（使用 listIterator(size())）。然后它逆向迭代列表，直至找到指定的元素，或者到达列表的开头。
     public int lastIndexOf(Object o) {
        ListIterator<E> it = listIterator(size());
        if (o==null) {
            while (it.hasPrevious())
                if (it.previous()==null)
                    return it.nextIndex();
        } else {
            while (it.hasPrevious())
                if (o.equals(it.previous()))
                    return it.nextIndex();
        }
        return -1;
     }
     
     //从此列表中移除所有元素（可选操作）。此调用返回后，列表将为空。
     //此实现调用 removeRange(0, size())。
     //注意：除非重写 remove(int index) 或 removeRange(int fromIndex, int toIndex)，否则此实现将抛出 UnsupportedOperationException。
     public void clear() {
        removeRange(0, size());
     }
     
     //将指定 collection 中的所有元素都插入到列表中的指定位置（可选操作）。
     //将当前处于该位置的元素（如果有的话）和所有后续元素向右移动（增加其索引）。
     //新元素将按照它们通过指定 collection 的迭代器所返回的顺序出现在此列表中。
     //如果在操作正在进行中修改了指定的 collection，那么该操作的行为是不确定的
     //（注意，如果指定的 collection 是此列表，并且它是非空的，则会发生这种情况）
     //此实现获取指定 collection 上的迭代器并迭代它，使用 add(int, E) 将迭代器获取的元素插入此列表的适当位置，一次一个。
     //为了提高效率，多数实现将重写此方法。
     //注意：除非重写 add(int, E)，否则此实现将抛出 UnsupportedOperationException。
     public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);
            modified = true;
        }
        return modified;
     }
     
     //返回以恰当顺序在此列表的元素上进行迭代的迭代器。
     //此实现返回 iterator 接口的一个直接实现，具体取决于底层实现列表的 size()、get(int) 和 remove(int) 方法。
     //注意，除非重写列表的 remove(int) 方法，否则此方法返回的迭代器将抛出一个 UnsupportedOperationException 来响应其 remove 方法。
     //根据 (protected) modCount 字段规范中的描述，在面临并发修改时，可以使此实现抛出运行时异常。
     public Iterator<E> iterator() {
        return new Itr();
     }
     
     //返回此列表元素的列表迭代器（按适当顺序）。
     public ListIterator<E> listIterator() {
        return listIterator(0);
     }
     
     //返回列表中元素的列表迭代器（按适当顺序），从列表的指定位置开始。指定的索引表示 next 的初始调用所返回的第一个元素。 
     //previous 方法的初始调用将返回索引比指定索引少 1 的元素。
     //此实现返回 ListIterator 接口的直接实现，扩展了由 iterator() 方法返回的 Iterator 接口的实现。
     //ListIterator 实现依赖于底层实现列表的 get(int)、set(int, E)、add(int, E) 和 remove(int) 方法。
     //注意：除非重写列表的 remove(int)、set(int, E) 和 add(int, E) 方法，
     //否则此实现返回的列表迭代器将抛出 UnsupportedOperationException 来响应其 remove、set 和 add 方法。
     //根据 (protected) modCount 字段规范中的描述，在面临并发修改时，可以使此实现抛出运行时异常。
     public ListIterator<E> listIterator(final int index) {
        rangeCheckForAdd(index);

        return new ListItr(index);
     }
     
     private class Itr implements Iterator<E> {
       int cursor = 0;
       int lastRet = -1;
       int expectedModCount = modCount;

       public boolean hasNext() {
           return cursor != size();
       }
       ...
     }
     
     private class ListItr extends Itr implements ListIterator<E> {...}
     
     //返回列表中指定的 fromIndex（包括 ）和 toIndex（不包括）之间的部分视图。（如果 fromIndex 和 toIndex 相等，则返回的列表为空）。
     //返回的列表由此列表支持，因此返回列表中的非结构性更改将反映在此列表中，反之亦然。返回的列表支持此列表支持的所有可选列表操作。
     //此方法省去了显式范围操作（此操作通常针对数组存在）。通过传递 subList 视图而非整个列表，期望列表的任何操作可用作范围操作。
     //例如，下面的语句从列表中移除了元素的范围：
     //      list.subList(from, to).clear();
     //可以对 indexOf 和 lastIndexOf 构造类似的语句，而且 Collections 类中的所有算法都可以应用于 subList。
     //如果支持列表（即此列表）通过任何其他方式（而不是通过返回的列表）从结构上修改，则此方法返回的列表语义将变为未定义
     //（从结构上修改是指更改列表的大小，或者以其他方式打乱列表，使正在进行的迭代产生错误的结果）。
     //此实现返回一个子类化 AbstractList 的列表。
     //子类在 private 字段中存储底层实现列表中 subList 的偏移量、subList 的大小（随其生存期变化）以及底层实现列表的预期 modCount 值。
     //子类有两个变体，其中一个实现 RandomAccess。如果此列表实现 RandomAccess，则返回的列表将是实现 RandomAccess 的子类实例。
     //子类的 set(int, E)、get(int)、add(int, E)、remove(int)、addAll(int, Collection) 
     //和 removeRange(int, int) 方法在对索引进行边界检查和调整偏移量之后，都委托给底层实现抽象列表上的相应方法。
     //addAll(Collection c) 方法返回 addAll(size, c)。
     //listIterator(int) 方法返回底层实现列表的列表迭代器上的“包装器对象”，使用底层实现列表上的相应方法可创建该迭代器。
     //iterator 方法返回 listIterator()，size 方法返回子类的 size 字段。
     //所有方法都将首先检查底层实现列表的实际 modCount 是否与其预期的值相等，并且在不相等时将抛出 ConcurrentModificationException。
     public List<E> subList(int fromIndex, int toIndex) {
        return (this instanceof RandomAccess ?
                new RandomAccessSubList<>(this, fromIndex, toIndex) :
                new SubList<>(this, fromIndex, toIndex));
     }
     
     //将指定的对象与此列表进行相等性比较。
     //当且仅当指定的对象也是一个列表，两个列表具有相同的大小，而且两个列表中所有相应的元素对都相等时，才返回 true。
     //（如果 (e1==null ? e2==null : e1.equals(e2))，则元素 e1 和 e2 相等。）
     //换句话说，如果两个列表包含相同的元素，且元素的顺序也相同，才将它们定义为相等。
     //此实现首先检查指定的对象是否为此列表。如果是，则返回 true；否则，它将检查指定的对象是否为一个列表。
     //如果不是，它将返回 false；如果是，它将迭代两个列表，比较相应的元素对。如果有任何比较结果返回 false，则此方法将返回 false。
     //如果某中某个迭代器在另一迭代器之前迭代完元素，则会返回 false（因为列表是不等长的）；否则，在迭代完成时将返回 true。
     public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator e2 = ((List) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
     }
     
     //返回此列表的哈希码值。
     //此实现使用在 List.hashCode() 方法的文档中用于定义列表哈希函数的代码。
     public int hashCode() {
        int hashCode = 1;
        for (E e : this)
            hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
        return hashCode;
     }
     
     //从此列表中移除索引在 fromIndex（包括）和 toIndex（不包括）之间的所有元素。向左移动所有后续元素（减小其索引）。
     //此调用缩短了 ArrayList，将其减少了 (toIndex - fromIndex) 个元素。（如果 toIndex==fromIndex，则此操作无效。）
     //此方法由此列表及其 subList 上的 clear 操作调用。重写此方法以利用内部列表实现可以极大地改进此列表及其 subList 上 clear 操作的性能。
     //此实现获取一个位于 fromIndex 之前的列表迭代器，并在移除该范围内的元素前重复调用 ListIterator.next（后跟 ListIterator.remove）。
     //注：如果 ListIterator.remove 需要的时间与元素数呈线性关系，那么此实现需要的时间与元素数的平方呈线性关系。
     protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
     }
     
     //已从结构上修改此列表的次数。从结构上修改是指更改列表的大小，或者打乱列表，从而使正在进行的迭代产生错误的结果。
     //此字段由 iterator 和 listIterator 方法返回的迭代器和列表迭代器实现使用。
     //如果意外更改了此字段中的值，则迭代器（或列表迭代器）将抛出 ConcurrentModificationException 
     //来响应 next、remove、previous、set 或 add 操作。在迭代期间面临并发修改时，它提供了快速失败行为，而不是非确定性行为。
     //子类是否使用此字段是可选的。如果子类希望提供快速失败迭代器（和列表迭代器），
     //则它只需在其 add(int, E) 和 remove(int) 方法（以及它所重写的、导致列表结构上修改的任何其他方法）中增加此字段。
     //对 add(int, E) 或 remove(int) 的单个调用向此字段添加的数量不得超过 1，
     //否则迭代器（和列表迭代器）将抛出虚假的 ConcurrentModificationExceptions。
     //如果某个实现不希望提供快速失败迭代器，则可以忽略此字段。
     protected transient int modCount = 0;
     
     private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
     }

     private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size();
     }
  }
  
  class SubList<E> extends AbstractList<E> {
    ...
  }
  
  class RandomAccessSubList<E> extends SubList<E> implements RandomAccess {
    ...
  }
```
