* AbstractSet抽象类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 此类提供 Set 接口的骨干实现，从而最大限度地减少了实现此接口所需的工作。

  &nbsp;&nbsp; 通过扩展此类来实现一个 set 的过程与通过扩展 AbstractCollection 来实现 Collection 的过程是相同的，除了此类的子类中的所有方法和构造方法都必须服从 Set 接口所强加的额外限制（例如，add 方法必须不允许将一个对象的多个实例添加到一个 set 中）。

  &nbsp;&nbsp; 注意：此类并没有重写 AbstractCollection 类中的任何实现。它仅仅添加了 equals 和 hashCode 的实现。

  &nbsp;&nbsp; 此类是 Java Collections Framework 的成员。
 
```java
  public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
    //单独的构造方法（由子类构造方法调用，通常是隐式的）
    protected AbstractSet() {
    }
    
    //比较指定对象与此 set 的相等性。
    //如果给定对象也是一个 set，两个 set 的大小相等，并且给定 set 的每个成员都包含在此 set 中，则返回 true。
    //这确保 equals 方法在 Set 接口的不同实现间正常工作。
    //此实现首先检查指定的对象是否是此 set；如果是，则返回 true。
    //然后，它将检查指定的对象是否是一个大小与此 set 的大小相等的 set；如果不是，则返回 false。
    //如果是，则返回 containsAll((Collection) o)。
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Set))
            return false;
        Collection c = (Collection) o;
        if (c.size() != size())
            return false;
        try {
            return containsAll(c);
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }
    }
    
    //返回此 set 的哈希码值。set 的哈希码被定义为该 set 中元素的哈希码的总和，其中 null 元素的哈希码被定义为 0。
    //这确保了 s1.equals(s2) 意味着对于任何两个 set s1 和 s2，都有 s1.hashCode()==s2.hashCode()，
    //正如 Object.hashCode() 的常规协定所要求的。
    //此实现对 set 进行迭代，在 set 的每个元素上调用 hashCode 方法，并合计结果。
    public int hashCode() {
        int h = 0;
        Iterator<E> i = iterator();
        while (i.hasNext()) {
            E obj = i.next();
            if (obj != null)
                h += obj.hashCode();
        }
        return h;
    }
    
    //从此 set 中移除包含在指定 collection 中的所有元素（可选操作）。
    //如果指定 collection 也是一个 set，则此操作有效地修改此 set，从而其值成为两个 set 的不对称差集。
    //通过在此 set 和指定 collection 上调用 size 方法，此实现可以确定哪一个更小。
    //如果此 set 中的元素更少，则该实现将在此 set 上进行迭代，依次检查迭代器返回的每个元素，
    //查看它是否包含在指定的 collection 中。如果包含它，则使用迭代器的 remove 方法从此 set 中将其移除。
    //如果指定 collection 中的元素更少，则该实现将在指定的 collection 上进行迭代，并使用此 set 的 remove 方法，
    //从此 set 中移除迭代器返回的每个元素。
    //注意：如果 iterator 方法返回的迭代器没有实现 remove 方法，则此实现抛出 UnsupportedOperationException。
    public boolean removeAll(Collection<?> c) {
        boolean modified = false;

        if (size() > c.size()) {
            for (Iterator<?> i = c.iterator(); i.hasNext(); )
                modified |= remove(i.next());
        } else {
            for (Iterator<?> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
        }
        return modified;
    }
    
  }
```
