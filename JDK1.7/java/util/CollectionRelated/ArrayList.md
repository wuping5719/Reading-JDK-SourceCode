* ArrayList类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; List 接口的大小可变数组的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。除了实现 List 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。（此类大致上等同于 Vector 类，除了此类是不同步的。）

  &nbsp;&nbsp; size、isEmpty、get、set、iterator 和 listIterator 操作都以固定时间运行。add 操作以分摊的固定时间运行，也就是说，添加 n 个元素需要 O(n) 时间。其他所有操作都以线性时间运行（大体上讲）。与用于 LinkedList 实现的常数因子相比，此实现的常数因子较低。

  &nbsp;&nbsp; 每个 ArrayList 实例都有一个容量。该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向 ArrayList 中不断添加元素，其容量也自动增长。并未指定增长策略的细节，因为这不只是添加元素会带来分摊固定时间开销那样简单。

  &nbsp;&nbsp; 在添加大量元素前，应用程序可以使用 ensureCapacity 操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。

  &nbsp;&nbsp; 注意：此实现不是同步的。如果多个线程同时访问一个 ArrayList 实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。（结构上的修改是指任何添加或删除一个或多个元素的操作，或者显式调整底层数组的大小；仅仅设置元素的值不是结构上的修改。）这一般通过对自然封装该列表的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedList 方法将该列表“包装”起来。这最好在创建时完成，以防止意外对列表进行不同步的访问：List list = Collections.synchronizedList(new ArrayList(...));    
  
  &nbsp;&nbsp; 此类的 iterator 和 listIterator 方法返回的迭代器是快速失败的：在创建迭代器之后，除非通过迭代器自身的 remove 或 add 方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

  &nbsp;&nbsp; 注意：迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测 bug。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
      private static final long serialVersionUID = 8683452581122892189L;
      
      //默认初始容量为10
      private static final int DEFAULT_CAPACITY = 10;
      
      //用于空实例的共享空数组实例
      private static final Object[] EMPTY_ELEMENTDATA = {};
      
      //用于存储ArrayList元素的数组
      private transient Object[] elementData;
      
      //ArrayList的大小（它包含的元素数）
      private int size;
      
      //构造一个具有指定初始容量的空列表
      public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        this.elementData = new Object[initialCapacity];
      }
      
      //构造一个初始容量为 10 的空列表
      public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
      }
      
      //构造一个包含指定collection的元素的列表，这些元素是按照该collection的迭代器返回它们的顺序排列的.
      public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
      }
      
      //将此ArrayList实例的容量调整为列表的当前大小。应用程序可以使用此操作来最小化ArrayList实例的存储量
      public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = Arrays.copyOf(elementData, size);
        }
      }
      
      //如有必要，增加此ArrayList实例的容量，以确保它至少能够容纳最小容量参数所指定的元素数
      public void ensureCapacity(int minCapacity) {
         int minExpand = (elementData != EMPTY_ELEMENTDATA) ? 0 : DEFAULT_CAPACITY;

         if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
         }
      }
      
      private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
      }
      
      private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
      }
      //可以分配的最大数组大小
      private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
      private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);   
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // mincapacity通常接近Size
        elementData = Arrays.copyOf(elementData, newCapacity);
      }
      private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) 
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
      }
      
      //返回此列表中的元素数
      public int size() {
        return size;
      }
      
      //如果此列表中没有元素，则返回true
      public boolean isEmpty() {
        return size == 0;
      }
      
      //如果此列表中包含指定的元素，则返回true。
      //更确切地讲，当且仅当此列表包含至少一个满足(o==null ? e==null : o.equals(e))的元素 e 时，则返回true
      public boolean contains(Object o) {
        return indexOf(o) >= 0;
      }
      
      //返回此列表中首次出现的指定元素的索引，或如果此列表不包含元素，则返回-1。
      //更确切地讲，返回满足(o==null ? get(i)==null : o.equals(get(i)))的最低索引i，如果不存在此类索引，则返回-1
      public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
      }
      
      //返回此列表中最后一次出现的指定元素的索引，或如果此列表不包含索引，则返回-1。
      //更确切地讲，返回满足(o==null ? get(i)==null : o.equals(get(i)))的最高索引i，如果不存在此类索引，则返回-1
      public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
      }
      
      //返回此ArrayList实例的浅副本（不复制这些元素本身）
      public Object clone() {
        try {
            @SuppressWarnings("unchecked")
            ArrayList<E> v = (ArrayList<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
      }
      
      //按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组。
      //由于此列表不维护对返回数组的任何引用，因而它将是“安全的”。
      //（换句话说，此方法必须分配一个新的数组）。因此，调用者可以自由地修改返回的数组。
      //此方法担当基于数组的 API 和基于 collection 的 API 之间的桥梁。
      public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
      }
      
      //按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组；
      //返回数组的运行时类型是指定数组的运行时类型。如果指定的数组能容纳列表，则将该列表返回此处。
      //否则，将分配一个具有指定数组的运行时类型和此列表大小的新数组。
      //如果指定的数组能容纳队列，并有剩余的空间（即数组的元素比队列多），
      //那么会将数组中紧接 collection 尾部的元素设置为 null。
      //（仅在调用者知道列表中不包含任何 null 元素时才能用此方法确定列表长度）。
      @SuppressWarnings("unchecked")
      public <T> T[] toArray(T[] a) {
        if (a.length < size)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
     }
     
     //位置访问操作
     @SuppressWarnings("unchecked")
     E elementData(int index) {
        return (E) elementData[index];
     }
     
     //返回此列表中指定位置上的元素
     public E get(int index) {
        rangeCheck(index);

        return elementData(index);
     }
     
     //用指定的元素替代此列表中指定位置上的元素
     public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
     }
     
     //将指定的元素添加到此列表的尾部
     public boolean add(E e) {
        ensureCapacityInternal(size + 1); 
        elementData[size++] = e;
        return true;
     }
     
     //将指定的元素插入此列表中的指定位置。向右移动当前位于该位置的元素（如果有）以及所有后续元素（将其索引加1）
     public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1); 
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
     }
     
     //移除此列表中指定位置上的元素。向左移动所有后续元素（将其索引减 1）
     public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; 

        return oldValue;
     }
     
     //移除此列表中首次出现的指定元素（如果存在）。如果列表不包含此元素，则列表不做改动。
     //更确切地讲，移除满足(o==null ? get(i)==null : o.equals(get(i)))的最低索引的元素（如果存在此类元素）
     //如果列表中包含指定的元素，则返回true（或者等同于这种情况：如果列表由于调用而发生更改，则返回true）
     public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
     }
     
     //跳过边界检查和不返回删除的值
     private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; 
     }
     
     //移除此列表中的所有元素。此调用返回后，列表将为空
     public void clear() {
        modCount++;

        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
     }
     
     //按照指定 collection 的迭代器所返回的元素顺序，将该 collection 中的所有元素添加到此列表的尾部。
     //如果正在进行此操作时修改指定的 collection ，那么此操作的行为是不确定的。
     //（这意味着如果指定的 collection 是此列表且此列表是非空的，那么此调用的行为是不确定的）。
     public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew); 
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
     }
     
     //从指定的位置开始，将指定 collection 中的所有元素插入到此列表中。
     //向右移动当前位于该位置的元素（如果有）以及所有后续元素（增加其索引）。
     //新元素将按照指定 collection 的迭代器所返回的元素顺序出现在列表中。
     public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  
        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew, numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
    
    //移除列表中索引在 fromIndex（包括）和 toIndex（不包括）之间的所有元素。
    //向左移动所有后续元素（减小其索引）。此调用将列表缩短了(toIndex - fromIndex)个元素。
    //（如果 toIndex==fromIndex，则此操作无效）
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved);

        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }
    
    //检查给定的索引是否在范围内
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    //构造IndexOutOfBoundsException的详细信息
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }
    
    //从这个列表中移除包含在指定集合中的所有元素
    public boolean removeAll(Collection<?> c) {
        return batchRemove(c, false);
    }
    
    //仅保留包含在指定集合中的列表元素
    public boolean retainAll(Collection<?> c) {
        return batchRemove(c, true);
    }
    
    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            if (r != size) {
                System.arraycopy(elementData, r, elementData, w, size - r);
                w += size - r;
            }
            if (w != size) {
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
    
    private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException {...}
    private void readObject(java.io.ObjectInputStream s) 
          throws java.io.IOException, ClassNotFoundException {...}
    public ListIterator<E> listIterator(int index) {...}
    public ListIterator<E> listIterator() {...}
    public Iterator<E> iterator() {...}
    
    private class Itr implements Iterator<E> {...}
    private class ListItr extends Itr implements ListIterator<E> {...}
    
    public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
    static void subListRangeCheck(int fromIndex, int toIndex, int size) {...}
    private class SubList extends AbstractList<E> implements RandomAccess {...}
  }
```
