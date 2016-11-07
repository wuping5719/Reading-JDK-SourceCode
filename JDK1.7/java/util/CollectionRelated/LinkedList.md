* LinkedList类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp; &nbsp; List 接口的链接列表实现。实现所有可选的列表操作，并且允许所有元素（包括 null）。除了实现 List 接口外，LinkedList 类还为在列表的开头及结尾 get、remove 和 insert 元素提供了统一的命名方法。这些操作允许将链接列表用作堆栈、队列或双端队列。

  &nbsp; &nbsp; 此类实现 Deque 接口，为 add、poll 提供先进先出队列操作，以及其他堆栈和双端队列操作。

  &nbsp; &nbsp; 所有操作都是按照双重链接列表的需要执行的。在列表中编索引的操作将从开头或结尾遍历列表（从靠近指定索引的一端）。

  &nbsp; &nbsp; 注意：此实现不是同步的。如果多个线程同时访问一个链接列表，而其中至少一个线程从结构上修改了该列表，则它必须保持外部同步。（结构修改指添加或删除一个或多个元素的任何操作；仅设置元素的值不是结构修改。）这一般通过对自然封装该列表的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedList方法来“包装”该列表。最好在创建时完成这一操作，以防止对列表进行意外的不同步访问，如下所示：
   List list = Collections.synchronizedList(new LinkedList(...));   
   
  &nbsp; &nbsp; 此类的 iterator 和 listIterator 方法返回的迭代器是快速失败的：在迭代器创建之后，如果从结构上对列表进行修改，除非通过迭代器自身的 remove 或 add 方法，其他任何时间任何方式的修改，迭代器都将抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒将来不确定的时间任意发生不确定行为的风险。

  &nbsp; &nbsp; 注意：迭代器的快速失败行为不能得到保证，一般来说，存在不同步的并发修改时，不可能作出任何硬性保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的方式是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

  &nbsp; &nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class LinkedList<E> extends AbstractSequentialList<E>
       implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
     transient int size = 0;
     
     //指向第一个节点的指针
     transient Node<E> first;
     //指向最后一个节点的指针
     transient Node<E> last;
     
     //构造一个空列表
     public LinkedList() {
     }
     
     //构造一个包含指定collection中的元素的列表，这些元素按其collection的迭代器返回的顺序排列
     public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
     }
     
     //链接E作为第一个元素
     private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
     }
     
     //链接E作为最后一个元素
     void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
     }
     
     //在非空节点succ之前插入元素
     void linkBefore(E e, Node<E> succ) {
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
     }
     
     //置空非空的第一结点f的链接
     private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
    
    //置空非空的最后一个结点l的链接
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
    
    //置空非空结点x的链接
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
    
    //返回此列表的第一个元素
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
    
    //返回此列表的最后一个元素
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
    
    //移除并返回此列表的第一个元素
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
    
    //移除并返回此列表的最后一个元素
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    
    //将指定元素插入此列表的开头
    public void addFirst(E e) {
        linkFirst(e);
    }
    
    //将指定元素添加到此列表的结尾
    public void addLast(E e) {
        linkLast(e);
    }
    
    //如果此列表包含指定元素，则返回true。
    //更确切地讲，当且仅当此列表包含至少一个满足(o==null ? e==null : o.equals(e))的元素e时返回true
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }
    
    //返回此列表的元素数
    public int size() {
        return size;
    }
    
    //将指定元素添加到此列表的结尾。此方法等效于addLast(E)。
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    //从此列表中移除首次出现的指定元素（如果存在）。如果列表不包含该元素，则不作更改。
    //更确切地讲，移除具有满足(o==null ? get(i)==null : o.equals(get(i)))的最低索引i的元素（如果存在这样的元素）
    //如果此列表已包含指定元素（或者此列表由于调用而发生更改），则返回 true。
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    //添加指定collection中的所有元素到此列表的结尾，顺序是指定collection的迭代器返回这些元素的顺序。
    //如果指定的 collection 在操作过程中被修改，则此操作的行为是不确定的。
    //（注意: 如果指定 collection 就是此列表并且非空，则此操作的行为是不确定的）
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
    
    //将指定 collection 中的所有元素从指定位置开始插入此列表。
    //移动当前在该位置上的元素（如果有），所有后续元素都向右移（增加其索引）。
    //新元素将按由指定 collection 的迭代器返回的顺序在列表中显示。
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
    
    //从此列表中移除所有元素
    public void clear() {
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
    
    //返回此列表中指定位置处的元素
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
    //将此列表中指定位置的元素替换为指定的元素
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
    
    //在此列表中指定的位置插入指定的元素。移动当前在该位置处的元素（如果有），所有后续元素都向右移（在其索引中添加1）
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
    
    //移除此列表中指定位置处的元素。将任何后续元素向左移（从索引中减1）。返回从列表中删除的元素
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
    
    //判断索引是否越界
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }
    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    //返回指定元素索引中的（非空）结点
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
    
    //返回此列表中首次出现的指定元素的索引，如果此列表中不包含该元素，则返回-1。
    //更确切地讲，返回满足(o==null ? get(i)==null : o.equals(get(i)))的最低索引i；如果没有此索引，则返回-1
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
    
    //返回此列表中最后出现的指定元素的索引，如果此列表中不包含该元素，则返回-1。
    //更确切地讲，返回满足(o==null ? get(i)==null : o.equals(get(i)))的最高索引i；如果没有此索引，则返回-1
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
    
    //获取但不移除此列表的头（第一个元素）
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
    
    //获取但不移除此列表的头（第一个元素）
    public E element() {
        return getFirst();
    }
    
    //获取并移除此列表的头（第一个元素）
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    
    //获取并移除此列表的头（第一个元素）
    public E remove() {
        return removeFirst();
    }
    
    //将指定元素添加到此列表的末尾（最后一个元素）
    public boolean offer(E e) {
        return add(e);
    }
    
    //在此列表的开头插入指定的元素
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }
    
    //在此列表末尾插入指定的元素
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }
    
    //获取但不移除此列表的第一个元素；如果此列表为空，则返回null
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
    
    //获取但不移除此列表的最后一个元素；如果此列表为空，则返回null
    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
    
    //获取并移除此列表的第一个元素；如果此列表为空，则返回null
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    
    //获取并移除此列表的最后一个元素；如果此列表为空，则返回null
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }
    
    //将元素推入此列表所表示的堆栈。换句话说，将该元素插入此列表的开头。此方法等效于addFirst(E)。
    public void push(E e) {
        addFirst(e);
    }
    
    //从此列表所表示的堆栈处弹出一个元素。换句话说，移除并返回此列表的第一个元素。此方法等效于removeFirst()。
    public E pop() {
        return removeFirst();
    }
    
    //从此列表中移除第一次出现的指定元素（从头部到尾部遍历列表时）。如果列表不包含该元素，则不作更改
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }
    
    //从此列表中移除最后一次出现的指定元素（从头部到尾部遍历列表时）。如果列表不包含该元素，则不作更改
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    //返回此列表中的元素的列表迭代器（按适当顺序），从列表中指定位置开始。遵守List.listIterator(int)的常规协定。
    //列表迭代器是快速失败的：在迭代器创建之后，如果从结构上对列表进行修改，除非通过列表迭代器自身的remove或add方法，
    //其他任何时间任何方式的修改，列表迭代器都将抛出ConcurrentModificationException。
    //因此，面对并发的修改，迭代器很快就会完全失败，而不冒将来不确定的时间任意发生不确定行为的风险。
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }
    private class ListItr implements ListIterator<E> {...}
    
    //返回以逆向顺序在此双端队列的元素上进行迭代的迭代器。元素将按从最后一个（尾部）到第一个（头部）的顺序返回。
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }
    private class DescendingIterator implements Iterator<E> {...}
    
    @SuppressWarnings("unchecked")
    private LinkedList<E> superClone() {
        try {
            return (LinkedList<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }
    //返回此LinkedList的浅副本。（这些元素本身没有复制）
    public Object clone() {
        LinkedList<E> clone = superClone();

        // Put clone into "virgin" state
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // Initialize clone with our elements
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        return clone;
    }
    
    //返回以适当顺序（从第一个元素到最后一个元素）包含此列表中所有元素的数组。
    //由于此列表不维护对返回数组的任何引用，因而它将是“安全的”。
    //（换句话说，此方法必须分配一个新数组）。因此，调用者可以随意修改返回的数组。
    //此方法充当基于数组的 API 与基于 collection 的 API 之间的桥梁。
    public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }
    
    //返回以适当顺序（从第一个元素到最后一个元素）包含此列表中所有元素的数组；返回数组的运行时类型为指定数组的类型。
    //如果指定数组能容纳列表，则在其中返回该列表。否则，分配具有指定数组的运行时类型和此列表大小的新数组。
    //如果指定数组能容纳列表，并有剩余空间（即数组比列表元素多），则紧跟在列表末尾的数组元素会被设置为 null。
    //（只有在调用者知道列表不包含任何 null 元素时，才可使用此方法来确定列表的长度。）
    //像toArray()方法一样，此方法充当基于数组的 API 与基于 collection 的 API 之间的桥梁。
    //更进一步说，此方法允许对输出数组的运行时类型上进行精确控制，在某些情况下，可以用来节省分配开销。
    //假定 x 是只包含字符串的一个已知列表。以下代码可用来将该列表转储到一个新分配的 String 数组：
    //    String[] y = x.toArray(new String[0]);
    //注意： toArray(new Object[0]) 和 toArray() 在功能上是相同的。
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            a = (T[])java.lang.reflect.Array.newInstance(a.getClass().getComponentType(), size);
        int i = 0;
        Object[] result = a;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;

        if (a.length > size)
            a[size] = null;

        return a;
    }
    
    private static final long serialVersionUID = 876323262645176354L;
    
    private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException {...}
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {...}
  }
```
