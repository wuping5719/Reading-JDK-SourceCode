* Thread类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 线程是程序中的执行线程。Java 虚拟机允许应用程序并发地运行多个执行线程。

  &nbsp;&nbsp; 每个线程都有一个优先级，高优先级线程的执行优先于低优先级线程。每个线程都可以或不可以标记为一个守护程序。当某个线程中运行的代码创建一个新 Thread 对象时，该新线程的初始优先级被设定为创建线程的优先级，并且当且仅当创建线程是守护线程时，新线程才是守护程序。

  &nbsp;&nbsp; 当 Java 虚拟机启动时，通常都会有单个非守护线程（它通常会调用某个指定类的 main 方法）。Java 虚拟机会继续执行线程，直到下列任一情况出现时为止：  
  &nbsp;&nbsp; 1.调用了 Runtime 类的 exit 方法，并且安全管理器允许退出操作发生。         
  &nbsp;&nbsp; 2.非守护线程的所有线程都已停止运行，无论是通过从对 run 方法的调用中返回，还是通过抛出一个传播到 run 方法之外的异常。
  
  &nbsp;&nbsp; 创建新执行线程有两种方法。一种方法是将类声明为 Thread 的子类。该子类应重写 Thread 类的 run 方法。接下来可以分配并启动该子类的实例。例如，计算大于某一规定值的质数的线程可以写成：
  
```java
  class PrimeThread extends Thread {
         long minPrime;
         PrimeThread(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
             ...
         }
  }
```  

 &nbsp;&nbsp;  然后，下列代码会创建并启动一个线程：
 
```java
  PrimeThread p = new PrimeThread(143);
  p.start();
```    

  &nbsp;&nbsp; 创建线程的另一种方法是声明实现 Runnable 接口的类。该类然后实现 run 方法。然后可以分配该类的实例，在创建 Thread 时作为一个参数来传递并启动。采用这种风格的同一个例子如下所示：
  
```java
  class PrimeRun implements Runnable {
         long minPrime;
         PrimeRun(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
             ...
         }
     }
 
```       

  &nbsp;&nbsp; 然后，下列代码会创建并启动一个线程：
  
```java
  PrimeRun p = new PrimeRun(143);
  new Thread(p).start();
``` 
 
  &nbsp;&nbsp; 每个线程都有一个标识名，多个线程可以同名。如果线程创建时没有指定标识名，就会为其生成一个新名称。
  
```java
  public class Thread implements Runnable {
     //确保registerNatives是最先执行的
     //这个方法及静态代码块在Object类中也存在，可以说是重写了Object类的registerNatives代码
     private static native void registerNatives();
     static {
        registerNatives();
     }
     
     private char        name[];      //线程名
     private int         priority;    //线程优先级
     private Thread      threadQ;
     private long        eetop;       
     
     //Whether or not to single_step this thread.
     private boolean     single_step;
     
     //标志本线程是否是一个守护线程
     private boolean     daemon = false;
     
     //JVM状态
     private boolean     stillborn = false;
     
     //哪个线程会被执行，调用run方法时run方法是哪个类里面的.
     private Runnable target;
     
     //本线程所属线程组
     private ThreadGroup group;
     
     //本线程的上下文类加载器
     private ClassLoader contextClassLoader;
     
     //本线程所继承的AccessControlContext环境
     private AccessControlContext inheritedAccessControlContext;
     
     //为匿名线程命名的自动计数器。线程的编号，没有赋初值，所以从0开始。静态的。
     private static int threadInitNumber;
     //获得下一个线程的编号，使用了synchronized关键字来同步该方法，防止并发。
     private static synchronized int nextThreadNum() {
        return threadInitNumber++;
     }
     
     //与该线程相关的ThreadLocal值，该map由ThreadLocal类保持。
     ThreadLocal.ThreadLocalMap threadLocals = null;
     
     //与该线程相关的InheritableThreadLocal值，该map由InheritableThreadLocal类保持。
     ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
     
     //该线程请求的堆栈大小，如果创建者没有指定堆栈大小则默认为0
     private long stackSize;
     
     //JVM的私有状态，持续到本地线程终止
     private long nativeParkEventPointer;
     
     //线程ID
     private long tid;
     
     //用于生成线程ID
     private static long threadSeqNumber;
     
     //线程状态变量，默认为0，初始化显示线程"尚未启动”
     private volatile int threadStatus = 0;
     
     private static synchronized long nextThreadID() {
        return ++threadSeqNumber;
     }
     
     //该参数支持并发调用java.util.concurrent.locks.LockSupport.park
     volatile Object parkBlocker;
     
     //该线程在可中断的I/O操作中阻塞。在设置线程的中断状态后，该中断方法应该被调用
     private volatile Interruptible blocker;
     private final Object blockerLock = new Object();
     
     void blockedOn(Interruptible b) {
        synchronized (blockerLock) {
            blocker = b;
        }
     }
     
     //线程可以具有的最低优先级
     public final static int MIN_PRIORITY = 1;
     
     //分配给线程的默认优先级
     public final static int NORM_PRIORITY = 5;
     
     //线程可以具有的最高优先级
     public final static int MAX_PRIORITY = 10;
     
     //返回对当前正在执行的线程对象的引用
     public static native Thread currentThread();
     
     //暂停当前正在执行的线程对象，并执行其他线程
     public static native void yield();
     
     //在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），
     //此操作受到系统计时器和调度程序精度和准确性的影响。该线程不丢失任何监视器的所属权。 
     public static native void sleep(long millis) throws InterruptedException;
     
     //在指定的毫秒数加指定的纳秒数内让当前正在执行的线程休眠（暂停执行），
     //此操作受到系统计时器和调度程序精度和准确性的影响。该线程不丢失任何监视器的所属权。
     public static void sleep(long millis, int nanos) throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException("nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        sleep(millis);
    }
    
    //用当前AccessControlContext初始化一个线程
    private void init(ThreadGroup g, Runnable target, String name, long stackSize) {
        init(g, target, name, stackSize, null);
    }
    
    //Thread类中的init方法是Thread类中所有构造方法都调用的方法，用于初始化线程的各种信息
    //把线程类的引用保存到target中。这样，当调用 Thread 的 run 方法时，target 就不为空了，
    //而是继续调用了 target 的 run 方法，所以我们需要实现 Runnable 的 run 方法。
    //这样通过 Thread 的 run 方法就调用到了 Runnable 实现类中的 run 方法。
    //这也是 Runnable 接口实现的线程类需要这样启动的原因。
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        Thread parent = currentThread();   // 当先线程
        //安全管理，查看线程拥有的功能（例如：读写文件，访问网络）。  
        SecurityManager security = System.getSecurityManager();  
        if (g == null) {
            // 如果有一个安全管理器，那么询问安全管理器该做什么
            if (security != null) {
                g = security.getThreadGroup();
            }

            // 如果安全管理器没有强大的支持，那么就用调用当前线程的线程组
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }
        
        g.checkAccess();   //确定当前运行的线程是否有权修改此线程组。   

        // 我们有所需的权限吗？
        if (security != null) {
            //权限校验
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        //记录一个线程到线程组里，好像只是取得一个线程号。其实就是一个计数器自增：nUnstartedThreads++
        g.addUnstarted(); 

        this.group = g;                    //初始化确定线程组
        this.daemon = parent.isDaemon();   //初始化确定线程是否为守护线程  
        this.priority = parent.getPriority();  //初始化确定线程的优先级  
        this.name = name.toCharArray();   //初始化确定线程名字  
        if (security == null || isCCLOverridden(parent.getClass()))  //初始化确定上下文的类加载器
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;   
        //权限快照
        this.inheritedAccessControlContext = acc != null ? acc : AccessController.getContext();
        this.target = target;    //初始化确定目标Runnable对象，即具体执行哪个Runnable对象里的run方法的对象
        setPriority(priority);   //初始化设置线程优先级
        if (parent.inheritableThreadLocals != null)  //创建本地线程
            this.inheritableThreadLocals = ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        this.stackSize = stackSize;   //初始化栈大小

        tid = nextThreadID();   //初始化设置线程的ID
    }
    
    //Thread不支持clone()，构建一个新的线程。
    @Override
    protected Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
    
    //分配新的 Thread 对象。
    //这种构造方法与 Thread(null, null, gname) 具有相同的作用，其中 gname 是一个新生成的名称。
    //自动生成的名称的形式为 "Thread-"+ n，其中的 n 为整数。
    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    
    //分配新的 Thread 对象。
    //这种构造方法与 Thread(null, target, gname) 具有相同的作用，其中的 gname 是一个新生成的名称。
    //自动生成的名称的形式为 “Thread-”+ n，其中的 n 为整数。
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    
    //创建一个新线程，继承了AccessControlContext。
    //这不是一个公共构造函数。
    Thread(Runnable target, AccessControlContext acc) {
        init(null, target, "Thread-" + nextThreadNum(), 0, acc);
    }
    
    //分配新的 Thread 对象。
    //这种构造方法与 Thread(group, target, gname) 具有相同的作用，其中的 gname 是一个新生成的名称。
    //自动生成的名称的形式为 "Thread-"+ n ，其中的 n 为整数。
    public Thread(ThreadGroup group, Runnable target) {
        init(group, target, "Thread-" + nextThreadNum(), 0);
    }
    
    //分配新的 Thread 对象。这种构造方法与 Thread(null, null, name) 具有相同的作用。
    public Thread(String name) {
        init(null, null, name, 0);
    }

    //分配新的 Thread 对象。这种构造方法与 Thread(group, null, name) 具有相同的作用。
    public Thread(ThreadGroup group, String name) {
        init(group, null, name, 0);
    }
    
    //分配新的 Thread 对象。这种构造方法与 Thread(null, target, name) 具有相同的作用。
    public Thread(Runnable target, String name) {
        init(null, target, name, 0);
    }
    
    //分配新的 Thread 对象，以便将 target 作为其运行对象，将指定的 name 作为其名称，
    //并作为 group 所引用的线程组的一员。
    //如果 group 为 null，并且有安全管理器，则该组由安全管理器的 getThreadGroup 方法确定。
    //如果 group 为 null，并且没有安全管理器，或安全管理器的 getThreadGroup 方法返回 null，
    //则该组与创建新线程的线程被设定为相同的 ThreadGroup。
    //如果有安全管理器，则其 checkAccess 方法通过 ThreadGroup 作为其参数被调用。
    //此外，当被重写 getContextClassLoader 或 setContextClassLoader 
    //方法的子类构造方法直接或间接调用时，其 checkPermission 方法通过 
    // RuntimePermission("enableContextClassLoaderOverride") 权限调用。
    //其结果可能是 SecurityException。
    //如果 target 参数不是 null，则 target 的 run 方法在启动该线程时调用。
    //如果 target 参数为 null，则该线程的 run 方法在该线程启动时调用。
    //新创建线程的优先级被设定为创建该线程的线程的优先级，即当前正在运行的线程的优先级。
    //方法 setPriority 可用于将优先级更改为一个新值。
    //当且仅当创建新线程的线程当前被标记为守护线程时，新创建的线程才被标记为守护线程。
    //方法 setDaemon 可用于改变线程是否为守护线程。
    public Thread(ThreadGroup group, Runnable target, String name) {
        init(group, target, name, 0);
    }
  }
```
