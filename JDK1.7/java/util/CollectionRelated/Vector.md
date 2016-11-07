* Vector类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Vector类可以实现可增长的对象数组。与数组一样，它包含可以使用整数索引进行访问的组件。但是，Vector的大小可以根据需要增大或缩小，以适应创建 Vector 后进行添加或移除项的操作。

  &nbsp;&nbsp; 每个向量会试图通过维护 capacity 和 capacityIncrement 来优化存储管理。capacity 始终至少应与向量的大小相等；这个值通常比后者大些，因为随着将组件添加到向量中，其存储将按 capacityIncrement 的大小增加存储块。应用程序可以在插入大量组件前增加向量的容量；这样就减少了增加的重分配的量。

  &nbsp;&nbsp; 由 Vector 的 iterator 和 listIterator 方法所返回的迭代器是快速失败的：如果在迭代器创建后的任意时间从结构上修改了向量（通过迭代器自身的 remove 或 add 方法之外的任何其他方式），则迭代器将抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就完全失败，而不是冒着在将来不确定的时间任意发生不确定行为的风险。Vector 的 elements 方法返回的 Enumeration 不是快速失败的。

  &nbsp;&nbsp; 注意: 迭代器的快速失败行为不能得到保证，一般来说，存在不同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的方式是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测 bug。

  &nbsp;&nbsp; 从 Java 2 平台 v1.2 开始，此类改进为可以实现 List 接口，使它成为 Java Collections Framework 的成员。与新 collection 实现不同，Vector 是同步的。
 
```java
  public class Vector<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
     //存储向量组件的数组缓冲区。vector的容量就是此数据缓冲区的长度，该长度至少要足以包含向量的所有元素。
     //Vector中的最后一个元素后的任何数组元素都为 null。
     protected Object[] elementData;
     
     //Vector对象中的有效组件数。从elementData[0]到elementData[elementCount-1]的组件均为实际项。
     protected int elementCount;
     
     //向量的大小大于其容量时，容量自动增加的量。如果容量的增量小于等于零，则每次需要增大容量时，向量的容量将增大一倍。
     protected int capacityIncrement;
     
     private static final long serialVersionUID = -2767605614048989439L;
     
    //使用指定的初始容量和容量增量构造一个空的向量。
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    
    //使用指定的初始容量和等于零的容量增量构造一个空向量。
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
    
    //构造一个空向量，使其内部数据数组的大小为10，其标准容量增量为零。
    public Vector() {
        this(10);
    }
    
    //构造一个包含指定 collection 中的元素的向量，这些元素按其 collection 的迭代器返回元素的顺序排列
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray()可能不会返回对象数组
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
    
    //将此向量的组件复制到指定的数组中。此向量中索引k处的项将复制到anArray的组件k中。
    public synchronized void copyInto(Object[] anArray) {
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }
    
    //对此向量的容量进行微调，使其等于向量的当前大小。
    //如果此向量的容量大于其当前大小，则通过将其内部数据数组（保存在字段elementData中）替换为一个较小的数组，
    //从而将容量更改为等于当前大小。应用程序可以使用此操作最小化向量的存储。
    public synchronized void trimToSize() {
        modCount++;
        int oldCapacity = elementData.length;
        if (elementCount < oldCapacity) {
            elementData = Arrays.copyOf(elementData, elementCount);
        }
    }
    
    //增加此向量的容量（如有必要），以确保其至少能够保存最小容量参数指定的组件数。
    //如果此向量的当前容量小于minCapacity，则通过将其内部数据数组（保存在字段elementData中）替换为一个较大的数组来增加其容量。
    //新数据数组的大小将为原来的大小加上capacityIncrement，除非capacityIncrement的值小于等于零，
    //在后一种情况下，新的容量将为原来容量的两倍，不过，如果此大小仍然小于 minCapacity，则新容量将为 minCapacity。
    public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
            modCount++;
            ensureCapacityHelper(minCapacity);
        }
    }
    private void ensureCapacityHelper(int minCapacity) {
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    //可以分配的最大数组大小。一些虚拟机在数组中保留头单词。试图分配较大的数组可能导致OutOfMemoryError：要求数组大小超过VM的限制
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // 溢出
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
    }
    
    //设置此向量的大小。如果新大小大于当前大小，则会在向量的末尾添加相应数量的 null 项。
    //如果新大小小于当前大小，则丢弃索引 newSize 处及其之后的所有项。
    public synchronized void setSize(int newSize) {
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }
    
    //返回此向量的当前容量。
    public synchronized int capacity() {
        return elementData.length;
    }
    
    //返回此向量中的组件数。
    public synchronized int size() {
        return elementCount;
    }
    
    //测试此向量是否不包含组件。
    public synchronized boolean isEmpty() {
        return elementCount == 0;
    }
    
    //返回此向量的组件的枚举。返回的 Enumeration 对象将生成此向量中的所有项。
    //生成的第一项为索引 0 处的项，然后是索引 1 处的项，依此类推。
    public Enumeration<E> elements() {
        return new Enumeration<E>() {
            int count = 0;

            public boolean hasMoreElements() {
                return count < elementCount;
            }

            public E nextElement() {
                synchronized (Vector.this) {
                    if (count < elementCount) {
                        return elementData(count++);
                    }
                }
                throw new NoSuchElementException("Vector Enumeration");
            }
        };
    }
    
    //如果此向量包含指定的元素，则返回true。
    //更确切地讲，当且仅当此向量至少包含一个满足 (o==null ? e==null : o.equals(e)) 的元素e时，返回true。
    public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }
    
    //返回此向量中第一次出现的指定元素的索引，如果此向量不包含该元素，则返回 -1。
    //更确切地讲，返回满足 (o==null ? get(i)==null : o.equals(get(i))) 的最低索引i；如果没有这样的索引，则返回-1。
    public int indexOf(Object o) {
        return indexOf(o, 0);
    }
    
    //返回此向量中第一次出现的指定元素的索引，从index处正向搜索，如果未找到该元素，则返回-1。
    //更确切地讲，返回满足(i >= index && (o==null ? get(i)==null : o.equals(get(i))))的最低索引i；如果没有这样的索引，则返回-1
    public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    
    //返回此向量中最后一次出现的指定元素的索引；如果此向量不包含该元素，则返回-1。
    //更确切地讲，返回满足(o==null ? get(i)==null : o.equals(get(i)))的最高索引i；如果没有这样的索引，则返回-1
    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    }
    
    //返回此向量中最后一次出现的指定元素的索引，从index处逆向搜索，如果未找到该元素，则返回-1。
    //更确切地讲，返回满足(i <= index && (o==null ? get(i)==null : o.equals(get(i))))的最高索引i；如果没有这样的索引，则返回-1
    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    
    //返回指定索引处的组件。此方法的功能与get(int)方法的功能完全相同（后者是List接口的一部分）
    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }
        
        return elementData(index);
    }
    
    //返回此向量的第一个组件(位于索引0)处的项
    public synchronized E firstElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(0);
    }
    
    //返回此向量的最后一个组件
    public synchronized E lastElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(elementCount - 1);
    }
    
    //将此向量指定index处的组件设置为指定的对象。丢弃该位置以前的组件。
    //索引必须为一个大于等于0且小于向量当前大小的值。
    //此方法的功能与set(int, E)方法的功能完全相同(后者是List 接口的一部分). 
    //注意: set方法将反转参数的顺序，与数组用法更为匹配。另外还要注意，set方法将返回以前存储在指定位置的旧值。
    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }
        elementData[index] = obj;
    }
    
    //删除指定索引处的组件。此向量中的每个索引大于等于指定index的组件都将下移，使其索引值变成比以前小1的值。此向量的大小将减1。
    //索引必须为一个大于等于0且小于向量当前大小的值。
    //此方法的功能与remove(int)方法的功能完全相同(后者是List接口的一部分)。注意: remove方法将返回存储在指定位置的旧值。
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* 让gc做它的工作 */
    }
    
    //将指定对象作为此向量中的组件插入到指定的 index 处。
    //此向量中的每个索引大于等于指定index的组件都将向上移位，使其索引值变成比以前大 1 的值。
    //索引必须为一个大于等于 0 且小于等于向量当前大小的值（如果索引等于向量的当前大小，则将新元素添加到向量）。
    //此方法的功能与add(int, E)方法的功能完全相同(后者是List接口的一部分)。注意: add方法将反转参数的顺序，与数组用法更为匹配
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }
    
    //将指定的组件添加到此向量的末尾，将其大小增加1。如果向量的大小比容量大，则增大其容量。
    //此方法的功能与add(E)方法的功能完全相同(后者是 List 接口的一部分)。
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }
    
    //从此向量中移除变量的第一个（索引最小的）匹配项。
    //如果在此向量中找到该对象，那么向量中索引大于等于该对象索引的每个组件都会下移，使其索引值变成比以前小1的值。
    //此方法的功能与remove(Object)方法的功能完全相同（后者是 List 接口的一部分）。
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }
    
    //从此向量中移除全部组件，并将其大小设置为零。
    //此方法的功能与clear()方法的功能完全相同（后者是 List 接口的一部分）。
    public synchronized void removeAllElements() {
        modCount++;
        // 让gc做它的工作
        for (int i = 0; i < elementCount; i++)
            elementData[i] = null;

        elementCount = 0;
    }
    
    //返回向量的一个副本。副本中将包含一个对内部数据数组副本的引用，而非对此Vector对象的原始内部数据数组的引用
    public synchronized Object clone() {
        try {
            @SuppressWarnings("unchecked")
            Vector<E> v = (Vector<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, elementCount);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这是不应该发生的，因为我们是可克隆的
            throw new InternalError();
        }
    }
    
    //返回一个数组，包含此向量中以恰当顺序存放的所有元素
    public synchronized Object[] toArray() {
        return Arrays.copyOf(elementData, elementCount);
    }
    
    //返回一个数组，包含此向量中以恰当顺序存放的所有元素；返回数组的运行时类型为指定数组的类型。
    //如果向量能够适应指定的数组，则返回该数组。否则使用此数组的运行时类型和此向量的大小分配一个新数组。
    //如果向量能够适应指定的数组，而且还有多余空间（即数组的元素比向量的元素多），则将紧跟向量末尾的数组元素设置为 null。
    //(仅在调用者知道向量不包含任何 null 元素的情况下，这对确定向量的长度才有用)。
    @SuppressWarnings("unchecked")
    public synchronized <T> T[] toArray(T[] a) {
        if (a.length < elementCount)
            return (T[]) Arrays.copyOf(elementData, elementCount, a.getClass());

        System.arraycopy(elementData, 0, a, 0, elementCount);

        if (a.length > elementCount)
            a[elementCount] = null;

        return a;
    }
    
    //位置访问操作
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
    
    //返回向量中指定位置的元素
    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }
    
    //用指定的元素替换此向量中指定位置处的元素
    public synchronized E set(int index, E element) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
    
    //将指定元素添加到此向量的末尾
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
    
    //移除此向量中指定元素的第一个匹配项，如果向量不包含该元素，则元素保持不变。
    //更确切地讲，移除其索引 i 满足(o==null ? get(i)==null : o.equals(get(i)))的元素(如果存在这样的元素)
    public boolean remove(Object o) {
        return removeElement(o);
    }
    
    //在此向量的指定位置插入指定的元素。将当前位于该位置的元素（如果有）及所有后续元素右移（将其索引加 1）
    public void add(int index, E element) {
        insertElementAt(element, index);
    }
    
    //移除此向量中指定位置的元素。将所有后续元素左移（将其索引减 1）。返回此向量中移除的元素。
    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--elementCount] = null; 

        return oldValue;
    }
    
    //从此向量中移除所有元素。此调用返回后，向量将为空（除非抛出了异常）
    public void clear() {
        removeAllElements();
    }
    
    //如果此向量包含指定Collection中的所有元素，则返回true
    public synchronized boolean containsAll(Collection<?> c) {
        return super.containsAll(c);
    }
    
    //将指定Collection中的所有元素添加到此向量的末尾，按照指定collection的迭代器所返回的顺序添加这些元素。
    //如果指定的Collection在操作过程中被修改，则此操作的行为是不确定的
    //（这意味着如果指定的 Collection 是此向量，而此向量不为空，则此调用的行为是不确定的）
    public synchronized boolean addAll(Collection<? extends E> c) {
        modCount++;
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);
        System.arraycopy(a, 0, elementData, elementCount, numNew);
        elementCount += numNew;
        return numNew != 0;
    }
    
    //从此向量中移除包含在指定 Collection 中的所有元素
    public synchronized boolean removeAll(Collection<?> c) {
        return super.removeAll(c);
    }
    
    //在此向量中仅保留包含在指定Collection中的元素。换句话说，从此向量中移除所有未包含在指定Collection中的元素
    public synchronized boolean retainAll(Collection<?> c) {
        return super.retainAll(c);
    }
    
    //在指定位置将指定 Collection 中的所有元素插入到此向量中。
    //将当前位于该位置的元素（如果有）及所有后续元素右移（增大其索引值）。
    //新元素在向量中按照其由指定 collection 的迭代器所返回的顺序出现。
    public synchronized boolean addAll(int index, Collection<? extends E> c) {
        modCount++;
        if (index < 0 || index > elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);

        int numMoved = elementCount - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew, numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        elementCount += numNew;
        return numNew != 0;
    }
    
    //比较指定对象与此向量的相等性。当且仅当指定的对象也是一个List、两个List大小相同，并且其中所有对应的元素对都相等时才返回true。
    //（如果 (e1==null ? e2==null : e1.equals(e2))，则两个元素 e1 和 e2 相等）。
    //换句话说，如果两个List包含相同顺序的相同元素，则这两个List就定义为相等。
    public synchronized boolean equals(Object o) {
        return super.equals(o);
    }
    
    //返回此向量的哈希码值
    public synchronized int hashCode() {
        return super.hashCode();
    }
    
    //返回此向量的字符串表示形式，其中包含每个元素的String表示形式
    public synchronized String toString() {
        return super.toString();
    }
    
    //返回此List的部分视图，元素范围为从 fromIndex（包括）到 toIndex（不包括）。
    //（如果 fromIndex 和 toIndex 相等，则返回的 List 将为空）。
    //返回的 List 由此 List 支持，因此返回 List 中的更改将反映在此 List 中，反之亦然。
    //返回的列表支持此列表支持的所有可选列表操作。
    //此方法消除了显式范围操作的需要（此操作通常针对数组存在）。
    //通过操作subList视图而非整个List，期望List的任何操作可用作范围操作。
    //例如，下面的语句从List中移除了元素的范围： list.subList(from, to).clear();
    //可以对 indexOf 和 lastIndexOf 构造类似的语句，而且 Collections 类中的所有算法都可以应用于subList。
    //如果通过任何其他方式（而不是通过返回的列表）从结构上修改内部 List（即此 List），
    //则此方法返回的List的语义将变为不确定的(从结构上修改是指更改List的大小，或者以其他方式打乱List，使正在进行的迭代产生错误的结果)
    public synchronized List<E> subList(int fromIndex, int toIndex) {
        return Collections.synchronizedList(super.subList(fromIndex, toIndex), this);
    }
    
    //从此 List 中移除其索引位于fromIndex（包括）与 toIndex（不包括）之间的所有元素。将所有后续元素左移（减小其索引值）。
    //此调用会将List缩小 (toIndex - fromIndex) 个元素（如果 toIndex==fromIndex，则此操作没有任何效果）
    protected synchronized void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = elementCount - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved);

        int newElementCount = elementCount - (toIndex-fromIndex);
        while (elementCount != newElementCount)
            elementData[--elementCount] = null;
    }
    
    private void writeObject(java.io.ObjectOutputStream s) {...}
    public synchronized ListIterator<E> listIterator(int index) {...}
    public synchronized ListIterator<E> listIterator() {...}
    public synchronized Iterator<E> iterator() {...}
    private class Itr implements Iterator<E> {...}
    final class ListItr extends Itr implements ListIterator<E> {...}
  }
```
