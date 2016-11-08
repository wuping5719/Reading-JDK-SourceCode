* Map接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)       
  &nbsp;&nbsp; 将键映射到值的对象。一个映射不能包含重复的键；每个键最多只能映射到一个值。

  &nbsp;&nbsp; 此接口取代 Dictionary 类，后者完全是一个抽象类，而不是一个接口。

  &nbsp;&nbsp; Map 接口提供三种collection 视图，允许以键集、值集或键-值映射关系集的形式查看某个映射的内容。映射顺序 定义为迭代器在映射的 collection 视图上返回其元素的顺序。某些映射实现可明确保证其顺序，如 TreeMap 类；另一些映射实现则不保证顺序，如 HashMap 类。

  &nbsp;&nbsp; 注：将可变对象用作映射键时必须格外小心。当对象是映射中某个键时，如果以影响 equals 比较的方式更改了对象的值，则映射的行为将是不确定的。此项禁止的一种特殊情况是不允许某个映射将自身作为一个键包含。虽然允许某个映射将自身作为值包含，但请格外小心：在这样的映射上 equals 和 hashCode 方法的定义将不再是明确的。

  &nbsp;&nbsp; 所有通用的映射实现类应该提供两个“标准的”构造方法：一个 void（无参数）构造方法，用于创建空映射；一个是带有单个 Map 类型参数的构造方法，用于创建一个与其参数具有相同键-值映射关系的新映射。实际上，后一个构造方法允许用户复制任意映射，生成所需类的一个等价映射。尽管无法强制执行此建议（因为接口不能包含构造方法），但是 JDK 中所有通用的映射实现都遵从它。

  &nbsp;&nbsp; 此接口中包含的“破坏”方法可修改其操作的映射，如果此映射不支持该操作，这些方法将抛出UnsupportedOperationException。如果是这样，那么在调用对映射无效时，这些方法可以（但不要求）抛出UnsupportedOperationException。例如，如果某个不可修改的映射（其映射关系是“重叠”的）为空，则对该映射调用 putAll(Map) 方法时，可以（但不要求）抛出异常。

  &nbsp;&nbsp; 某些映射实现对可能包含的键和值有所限制。例如，某些实现禁止 null 键和值，另一些则对其键的类型有限制。尝试插入不合格的键或值将抛出一个未经检查的异常，通常是 NullPointerException 或 ClassCastException。试图查询是否存在不合格的键或值可能抛出异常，或者返回 false；某些实现将表现出前一种行为，而另一些则表现后一种。一般来说，试图对不合格的键或值执行操作且该操作的完成不会导致不合格的元素被插入映射中时，将可能抛出一个异常，也可能操作成功，这取决于实现本身。这样的异常在此接口的规范中标记为“可选”。

  &nbsp;&nbsp; 此接口是 Java Collections Framework 的成员。

  &nbsp;&nbsp; Collections Framework 接口中的很多方法是根据 equals 方法定义的。例如，containsKey(Object key) 方法的规范中写道：“当且仅当此映射包含针对满足 (key==null ? k==null : key.equals(k)) 的键 k 的映射关系时，返回 true”。不应将此规范解释为：调用具有非空参数 key 的 Map.containsKey 将导致对任意的键 k 调用 key.equals(k)。实现可随意进行优化，以避免调用 equals，例如，可首先比较两个键的哈希码（Object.hashCode() 规范保证哈希码不相等的两个对象不会相等）。一般来说，只要实现者认为合适，各种 Collections Framework 接口的实现可随意利用底层 Object 方法的指定行为。
 
```java
  public interface Map<K,V> {
    //返回此映射中的键-值映射关系数。如果映射关系数大于Integer.MAX_VALUE，则返回 Integer.MAX_VALUE。
    int size();
    
    //如果此映射未包含键-值映射关系，则返回 true。
    boolean isEmpty();
    
    //如果此映射包含指定键的映射关系，则返回 true。
    //更确切地讲，当且仅当此映射包含针对满足(key==null ? k==null : key.equals(k))的键k的映射关系时，返回true。
    //（最多只能有一个这样的映射关系）。
    boolean containsKey(Object key);
    
    //如果此映射将一个或多个键映射到指定值，则返回 true。
    //更确切地讲，当且仅当此映射至少包含一个对满足(value==null ? v==null : value.equals(v))的值v的映射关系时，
    //返回 true。对于大多数 Map 接口的实现而言，此操作需要的时间可能与映射大小呈线性关系。
    boolean containsValue(Object value);
    
    //返回指定键所映射的值；如果此映射不包含该键的映射关系，则返回 null。
    //更确切地讲，如果此映射包含满足(key==null ? k==null : key.equals(k))的键k到值v的映射关系，则此方法返回v；
    //否则返回 null。（最多只能有一个这样的映射关系）。
    //如果此映射允许 null 值，则返回 null 值并不一定表示该映射不包含该键的映射关系；
    //也可能该映射将该键显示地映射到 null。使用 containsKey 操作可区分这两种情况。
    V get(Object key);
    
    //将指定的值与此映射中的指定键关联（可选操作）。
    //如果此映射以前包含一个该键的映射关系，则用指定值替换旧值
    //（当且仅当 m.containsKey(k) 返回 true 时，才能说映射 m 包含键 k 的映射关系）。
    V put(K key, V value);
    
    //如果存在一个键的映射关系，则将其从此映射中移除（可选操作）。
    //更确切地讲，如果此映射包含从满足(key==null ? k==null :key.equals(k))的键k到值v的映射关系，则移除该映射关系。
    //（该映射最多只能包含一个这样的映射关系。）
    //返回此映射中以前关联该键的值，如果此映射不包含该键的映射关系，则返回 null。
    //如果此映射允许null值，则返回null值并不一定表示该映射不包含该键的映射关系；也可能该映射将该键显示地映射到null。
    //调用返回后，此映射将不再包含指定键的映射关系。
    V remove(Object key);
    
    //从指定映射中将所有映射关系复制到此映射中（可选操作）。
    //对于指定映射中的每个键 k 到值 v 的映射关系，此调用等效于对此映射调用一次 put(k, v)。
    //如果正在进行此操作的同时修改了指定的映射，则此操作的行为是不确定的。
    void putAll(Map<? extends K, ? extends V> m);
    
    //从此映射中移除所有映射关系（可选操作）。此调用返回后，该映射将为空。
    void clear();
    
    //返回此映射中包含的键的 Set 视图。该 set 受映射支持，所以对映射的更改可在此 set 中反映出来，反之亦然。
    //如果对该 set 进行迭代的同时修改了映射（通过迭代器自己的 remove 操作除外），则迭代结果是不确定的。
    //set 支持元素移除，通过Iterator.remove、 Set.remove、 removeAll、 
    //retainAll 和 clear 操作可从映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
    Set<K> keySet();
    
    //返回此映射中包含的值的 Collection 视图。
    //该 collection 受映射支持，所以对映射的更改可在此 collection 中反映出来，反之亦然。
    //如果对该 collection 进行迭代的同时修改了映射（通过迭代器自己的 remove 操作除外），则迭代结果是不确定的。
    //collection 支持元素移除，通过Iterator.remove、 Collection.remove、
    //removeAll、 retainAll 和 clear 操作可从映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
    Collection<V> values();
    
    //返回此映射中包含的映射关系的 Set 视图。
    //该 set 受映射支持，所以对映射的更改可在此 set 中反映出来，反之亦然。
    //如果对该 set 进行迭代的同时修改了映射（通过迭代器自己的 remove 操作，
    //或者通过对迭代器返回的映射项执行 setValue 操作除外），则迭代结果是不确定的。
    //set 支持元素移除，通过 Iterator.remove、 Set.remove、 removeAll、 retainAll 
    //和 clear 操作可从映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
    Set<Map.Entry<K, V>> entrySet();
    
    //键-值对
    interface Entry<K,V> {
       K getKey();
       V getValue();
       V setValue(V value);
       boolean equals(Object o);
       int hashCode();
    }
    
    //比较指定的对象与此映射是否相等。
    //如果给定的对象也是一个映射，并且这两个映射表示相同的映射关系，则返回 true。
    //更确切地讲，如果 m1.entrySet().equals(m2.entrySet())，则两个映射 m1 和 m2 表示相同的映射关系。
    //这可以确保 equals 方法在不同的 Map 接口实现间运行正常。
    boolean equals(Object o);
    
    //返回此映射的哈希码值。映射的哈希码定义为此映射 entrySet() 视图中每个项的哈希码之和。
    //这确保 m1.equals(m2) 对于任意两个映射 m1 和 m2 而言，都意味着 m1.hashCode()==m2.hashCode()，
    //正如 Object.hashCode() 常规协定的要求。
    int hashCode();
  }
```
