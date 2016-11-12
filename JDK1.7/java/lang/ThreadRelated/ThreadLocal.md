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
    //
    private final int threadLocalHashCode = nextHashCode();
  }
```  
