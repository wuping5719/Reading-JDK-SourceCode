* Runnable接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 线程状态转换图：(借的)：<http://www.devba.com/index.php/archives/3892.html>
  <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_thread.png" /></p>
  
  &nbsp;&nbsp; Runnable 接口应该由那些打算通过某一线程执行其实例的类来实现。类必须定义一个称为 run 的无参数方法。

  &nbsp;&nbsp; 接口的目的是为希望在活动时执行代码的对象提供一个公共协议。例如，Thread 类实现了 Runnable。激活的意思是说某个线程已启动并且尚未停止。

  &nbsp;&nbsp; 此外，Runnable 为非 Thread 子类的类提供了一种激活方式。通过实例化某个 Thread 实例并将自身作为运行目标，就可以运行实现 Runnable 的类而无需创建 Thread 的子类。大多数情况下，如果只想重写 run() 方法，而不重写其他 Thread 方法，那么应使用 Runnable 接口。这很重要，因为除非程序员打算修改或增强类的基本行为，否则不应为该类创建子类。
  
```java
  public interface Runnable {
     //使用实现接口 Runnable 的对象创建一个线程时，启动该线程将导致在独立执行的线程中调用对象的 run 方法。
     //方法 run 的常规协定是，它可能执行任何所需的动作。
     public abstract void run();
  }
```
