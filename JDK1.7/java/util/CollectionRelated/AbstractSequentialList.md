* AbstractSequentialList类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; AbstractSequentialList类提供了List接口的骨干实现，从而最大限度地减少了实现受“连续访问”数据存储（如链接列表）支持的此接口所需的工作。对于随机访问数据（如数组），应该优先使用 AbstractList，而不是先使用此类。

  &nbsp;&nbsp; 从某种意义上说，此类与在列表的列表迭代器上实现“随机访问”方法（get(int index)、set(int index, E element)、add(int index, E element) 和 remove(int index)）的 AbstractList 类相对立，而不是其他关系。

  &nbsp;&nbsp; 要实现一个列表，程序员只需要扩展此类，并提供 listIterator 和 size 方法的实现即可。对于不可修改的列表，程序员只需要实现列表迭代器的 hasNext、next、hasPrevious、previous 和 index 方法即可。

  &nbsp;&nbsp; 对于可修改的列表，程序员应该再另外实现列表迭代器的 set 方法。对于可变大小的列表，程序员应该再另外实现列表迭代器的 remove 和 add 方法。

  &nbsp;&nbsp; 按照 Collection 接口规范中的推荐，程序员通常应该提供一个 void（无参数）构造方法和 collection 构造方法。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public abstract class AbstractSequentialList<E> extends AbstractList<E> {
     //单独的构造方法（由子类构造方法调用，通常是隐式的）
     protected AbstractSequentialList() {
     }
     
     //返回此列表中指定位置上的元素。
     //此实现首先获得一个指向索引元素的列表迭代器（通过 listIterator(index) 方法）。
     //然后它使用 ListIterator.next 获得该元素并返回它。
     public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
     }
     
     //用指定的元素替代此列表中指定位置上的元素（可选操作）。
     //此实现首先获得一个指向索引元素的列表迭代器（通过 listIterator(index) 方法）。
     //然后它使用 ListIterator.next 获得当前元素，并使用 ListIterator.set 替代它。
     //注意：如果该列表迭代器没有实现 set 操作，则此实现将抛出 UnsupportedOperationException。
     public E set(int index, E element) {
        try {
            ListIterator<E> e = listIterator(index);
            E oldVal = e.next();
            e.set(element);
            return oldVal;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
     }
     
     //在此列表中的指定位置上插入指定的元素（可选操作）。
     //向右移动当前位于该位置上的元素（如果有）以及所有后续元素（将其索引加 1）。
     //此实现首先获得一个指向索引元素的列表迭代器（通过 listIterator(index)）。
     //然后它使用 ListIterator.add 插入指定的元素。
     //注意：如果列表迭代器没有实现 add 操作，则此实现将抛出 UnsupportedOperationException。
     public void add(int index, E element) {
        try {
            listIterator(index).add(element);
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
     }
     
     //移除此列表中指定位置上的元素（可选操作）。
     //向左移动所有后续元素（将其索引减 1）。返回从列表中移除的元素。
     //此实现首先获得一个指向索引元素的列表迭代器（通过 listIterator(index) 方法）。
     //然后它使用 ListIterator.remove 移除该元素。
     //注意：如果该列表迭代器没有实现 remove 操作，则此实现将抛出 UnsupportedOperationException。
     public E remove(int index) {
        try {
            ListIterator<E> e = listIterator(index);
            E outCast = e.next();
            e.remove();
            return outCast;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
     }
     
     //在此列表中指定的位置上插入指定 collection 中的所有元素（可选操作）。
     //向右移动当前位于该位置上的元素（如果有）以及所有后续元素（增加其索引）。
     //新元素将按指定 collection 的迭代器所返回的顺序出现在列表中。
     //如果正在进行此操作时修改指定的 collection，则此操作行为是未指定的。
     //（注意：如果指定的 collection 是此列表并且是非空的，则会发生这种情况。）
     //此实现获得指定 collection 的迭代器，以及此列表指向索引元素的列表迭代器（通过listIterator(index)方法）。
     //然后，它在指定的collection上进行迭代，通过使用ListIterator.next之后紧接着使用ListIterator.add方法（以跳过添加的元素），
     //把从迭代器中获得的元素逐个插入此列表中。
     //注意：如果listIterator方法返回的列表迭代器没有实现 add 操作，则此实现将会抛出 UnsupportedOperationException。
     public boolean addAll(int index, Collection<? extends E> c) {
        try {
            boolean modified = false;
            ListIterator<E> e1 = listIterator(index);
            Iterator<? extends E> e2 = c.iterator();
            while (e2.hasNext()) {
                e1.add(e2.next());
                modified = true;
            }
            return modified;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }
    
    //返回在此列表中的元素上进行迭代的迭代器（按适当顺序）。
    //此实现仅返回列表的一个列表迭代器。
    public Iterator<E> iterator() {
        return listIterator();
    }
    
    //返回在此列表中的元素上进行迭代的列表迭代器（按适当顺序）。
    public abstract ListIterator<E> listIterator(int index);
  }
```
