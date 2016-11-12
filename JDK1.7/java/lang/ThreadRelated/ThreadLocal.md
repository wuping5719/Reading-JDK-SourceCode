* ThreadLocal类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 该类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其 get 或 set 方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。ThreadLocal 实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如：用户 ID 或事务 ID）相关联。

  &nbsp;&nbsp; 例如：以下类生成对每个线程唯一的局部标识符。 线程 ID 是在第一次调用 UniqueThreadIdGenerator.getCurrentThreadId() 时分配的，在后续调用中不会更改。
```java
  import java.util.concurrent.atomic.AtomicInteger;

  public class UniqueThreadIdGenerator {
     private static final AtomicInteger uniqueId = new AtomicInteger(0);

     private static final ThreadLocal < Integer > uniqueNum = 
         new ThreadLocal < Integer > () {
             @Override protected Integer initialValue() {
                 return uniqueId.getAndIncrement();
         }
     };
 
     public static int getCurrentThreadId() {
         return uniqueId.get();
     }
 } 
```  
   &nbsp;&nbsp; 每个线程都保持对其线程局部变量副本的隐式引用，只要线程是活动的并且 ThreadLocal 实例是可访问的；在线程消失之后，其线程局部实例的所有副本都会被垃圾回收（除非存在对这些副本的其他引用）。
   
```java
  public class ThreadLocal<T> {
    //ThreadLocal依靠每个线程的哈希映射线性探针连接到每个线程(Thread.threadLocals和inheritableThreadLocals)
    //ThreadLocal对象作为密钥，通过threadLocalHashCode搜索。
    private final int threadLocalHashCode = nextHashCode();
    
    //下一个要给出的哈希码。自动更新，从0开始。
    private static AtomicInteger nextHashCode = new AtomicInteger();
    
    //连续生成的哈希码之间的区别
    private static final int HASH_INCREMENT = 0x61c88647;
    
    //返回下一个哈希码
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
    
    //返回此线程局部变量的当前线程的“初始值”。
    //线程第一次使用 get() 方法访问变量时将调用此方法，但如果线程之前调用了 set(T) 方法，
    //则不会对该线程再调用 initialValue 方法。通常，此方法对每个线程最多调用一次，
    //但如果在调用 get() 后又调用了 remove()，则可能再次调用此方法。
    //该实现返回 null；如果程序员希望线程局部变量具有 null 以外的值，
    //则必须为 ThreadLocal 创建子类，并重写此方法。通常将使用匿名内部类完成此操作。
    protected T initialValue() {
        return null;
    }
    
    //创建一个线程本地变量。
    public ThreadLocal() {
    }
    
    //返回此线程局部变量的当前线程副本中的值。
    //如果变量没有用于当前线程的值，则先将其初始化为调用 initialValue() 方法返回的值。
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }
    
    //set()建立初值的变体，被用于代替set()，以防用户已经重写了set()方法。
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
    
    //将此线程局部变量的当前线程副本中的值设置为指定值。
    //大部分子类不需要重写此方法，它们只依靠 initialValue() 方法来设置线程局部变量的值。
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    //移除此线程局部变量当前线程的值。
    //如果此线程局部变量随后被当前线程读取，且这期间当前线程没有设置其值，
    //则将调用其 initialValue() 方法重新初始化其值。这将导致在当前线程多次调用 initialValue 方法。
    public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
    }
    
    //得到一个ThreadLocal相关的Map，重写InheritableThreadLocal。
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
    
    //创建一个ThreadLocal相关的Map，重写InheritableThreadLocal。
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
    
    //创建继承本地线程Map的工厂方法，被设计为只能被Thread构造器调用。
    static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }
    
    //该方法在子类InheritableThreadLocal中定义
    T childValue(T parentValue) {
        throw new UnsupportedOperationException();
    }
    
    //ThreadLocalMap是一个定制的散列映射只适合维护线程本地值。
    //在ThreadLocal类外没有操作出口。帮助处理非常大的和长生存期的用法，哈希表条目使用WeakReference键。
    static class ThreadLocalMap {...}
    
  }
```  
