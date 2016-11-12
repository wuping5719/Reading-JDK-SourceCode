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
    
    //分配新的 Thread 对象，以便将 target 作为其运行对象，将指定的 name 作为其名称，
    //作为 group 所引用的线程组的一员，并具有指定的堆栈大小。
    //除了允许指定线程堆栈大小以外，这种构造方法与 Thread(ThreadGroup,Runnable,String) 完全一样。
    //堆栈大小是虚拟机要为该线程堆栈分配的地址空间的近似字节数。 
    //stackSize 参数（如果有）的作用具有高度的平台依赖性。
    //在某些平台上，指定一个较高的 stackSize 参数值可能使线程在抛出 StackOverflowError 之前达到较大的递归深度。
    //同样，指定一个较低的值将允许较多的线程并发地存在，且不会抛出 OutOfMemoryError（或其他内部错误）。
    //stackSize 参数的值与最大递归深度和并发程度之间的关系细节与平台有关。
    //在某些平台上，stackSize 参数的值无论如何不会起任何作用。
    //作为建议，可以让虚拟机自由处理 stackSize 参数。
    //如果指定值对于平台来说过低，则虚拟机可能使用某些特定于平台的最小值；
    //如果指定值过高，则虚拟机可能使用某些特定于平台的最大值。 
    //同样，虚拟机还会视情况自由地舍入指定值（或完全忽略它）。
    //将 stackSize 参数值指定为零将使这种构造方法与 
    // Thread(ThreadGroup, Runnable, String) 构造方法具有完全相同的作用。
    //由于这种构造方法的行为具有平台依赖性，因此在使用它时要非常小心。
    //执行特定计算所必需的线程堆栈大小可能会因 JRE 实现的不同而不同。
    //鉴于这种不同，仔细调整堆栈大小参数可能是必需的，而且可能要在支持应用程序运行的 JRE 实现上反复调整。
    //实现注意事项：鼓励 Java 平台实现者文档化其 stackSize parameter 的实现行为。
    public Thread(ThreadGroup group, Runnable target, String name, long stackSize) {
        init(group, target, name, stackSize);
    }
    
    //使该线程开始执行；Java 虚拟机调用该线程的 run 方法。
    //结果是两个线程并发地运行；当前线程（从调用返回给 start 方法）和另一个线程（执行其 run 方法）。
    //多次启动一个线程是非法的。特别是当线程已经结束执行后，不能再重新启动。
    public synchronized void start() {
        //如果线程状态不为0，则抛出线程状态非法的异常。
        //这也意味着重复调用start()方法会抛出异常。所以一个线程要启动时只能调用一次start()方法。
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        //否则，就把这个线程对象加到线程组中
        group.add(this);

        boolean started = false;
        try {
            start0();   //调用native的start0()方法来启动新线程
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                //什么都不做
            }
        }
    }
    private native void start0();
    
    //如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；
    //否则，该方法不执行任何操作并返回。
    //Thread 的子类应该重写该方法。
    @Override
    public void run() {
        //如果有目标Runnable对象，那么执行目标Runnable对象的run方法，否则什么都不做。
        if (target != null) { 
            target.run();
        }
    }
    
    //该方法被系统调用在线程退出之前清理该线程拥有的资源
    private void exit() {
        if (group != null) {
            group.threadTerminated(this);
            group = null;
        }
        target = null;
        /* Speed the release of some of these resources */
        threadLocals = null;
        inheritableThreadLocals = null;
        inheritedAccessControlContext = null;
        blocker = null;
        uncaughtExceptionHandler = null;
    }
    
    //已过时。 该方法具有固有的不安全性。
    //用 Thread.stop 来终止线程将释放它已经锁定的所有监视器
    //（作为沿堆栈向上传播的未检查 ThreadDeath 异常的一个自然后果）。
    //如果以前受这些监视器保护的任何对象都处于一种不一致的状态，则损坏的对象将对其他线程可见，这有可能导致任意的行为。
    //stop 的许多使用都应由只修改某些变量以指示目标线程应该停止运行的代码来取代。
    //目标线程应定期检查该变量，并且如果该变量指示它要停止运行，则从其运行方法依次返回。
    //如果目标线程等待很长时间（例如基于一个条件变量），则应使用 interrupt 方法来中断该等待。
    //有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。
    //强迫线程停止执行。
    //如果安装了安全管理器，则以 this 作为其参数调用 checkAccess 方法。
    //这可能引发 SecurityException（在当前线程中）。
    //如果该线程不同于当前线程（即当前线程试图终止除它本身以外的某一线程），
    //则安全管理器的 checkPermission 方法（带有 RuntimePermission("stopThread") 参数）也会被调用。
    //这会再次抛出 SecurityException（在当前线程中）。
    //无论该线程在做些什么，它所代表的线程都被迫异常停止，并抛出一个新创建的 ThreadDeath 对象，作为异常。
    //停止一个尚未启动的线程是允许的。 如果最后启动了该线程，它会立即终止。
    //应用程序通常不应试图捕获 ThreadDeath，除非它必须执行某些异常的清除操作
    //（注意，抛出 ThreadDeath 将导致 try 语句的 finally 子句在线程正式终止前执行）。
    //如果 catch 子句捕获了一个 ThreadDeath 对象，则重新抛出该对象很重要，因为这样该线程才会真正终止。
    //对其他未捕获的异常作出反应的顶级错误处理程序不会打印输出消息，
    //或者另外通知应用程序未捕获到的异常是否为 ThreadDeath 的一个实例。
    @Deprecated
    public final void stop() {
        stop(new ThreadDeath());
    }
    
    //已过时。  该方法具有固有的不安全性。
    //强迫线程停止执行。
    @Deprecated
    public final synchronized void stop(Throwable obj) {
        if (obj == null)
            throw new NullPointerException();

        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            checkAccess();
            if ((this != Thread.currentThread()) ||
                (!(obj instanceof ThreadDeath))) {
                security.checkPermission(SecurityConstants.STOP_THREAD_PERMISSION);
            }
        }

        if (threadStatus != 0) {
            resume(); // Wake up thread if it was suspended; no-op otherwise
        }

        // The VM can handle all thread states
        stop0(obj);
    }
    
    //中断线程。
    //如果当前线程没有中断它自己（这在任何情况下都是允许的），
    //则该线程的 checkAccess 方法就会被调用，这可能抛出 SecurityException。
    //如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，
    //或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，
    //则其中断状态将被清除，它还将收到一个 InterruptedException。
    //如果该线程在可中断的通道上的 I/O 操作中受阻，则该通道将被关闭，
    //该线程的中断状态将被设置并且该线程将收到一个 ClosedByInterruptException。
    //如果该线程在一个 Selector 中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，
    //并可能带有一个非零值，就好像调用了选择器的 wakeup 方法一样。
    //如果以前的条件都没有保存，则该线程的中断状态将被设置。
    //中断一个不处于活动状态的线程不需要任何作用。
    public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // 只是为了设置中断标志
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }
    
    //测试当前线程是否已经中断。线程的中断状态由该方法清除。
    //换句话说，如果连续两次调用该方法，则第二次调用将返回 false
    //（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。
    //线程中断被忽略，因为在中断时不处于活动状态的线程将由此返回 false 的方法反映出来。
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }
    
    //测试线程是否已经中断。线程的中断状态不受该方法的影响。
    //线程中断被忽略，因为在中断时不处于活动状态的线程将由此返回 false 的方法反映出来。
    public boolean isInterrupted() {
        return isInterrupted(false);
    }
    private native boolean isInterrupted(boolean ClearInterrupted);
    
    //已过时。该方法最初用于破坏该线程，但不作任何清除。它所保持的任何监视器都会保持锁定状态。
    //不过，该方法决不会被实现。即使要实现，它也极有可能以 suspend() 方式被死锁。
    //如果目标线程被破坏时保持一个保护关键系统资源的锁，则任何线程在任何时候都无法再次访问该资源。
    //如果另一个线程曾试图锁定该资源，则会出现死锁。这类死锁通常会证明它们自己是“冻结”的进程。
    //有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。
    @Deprecated
    public void destroy() {
        throw new NoSuchMethodError();
    }
    
    //测试线程是否处于活动状态。如果线程已经启动且尚未终止，则为活动状态。
    public final native boolean isAlive();
    
    //已过时。该方法已经遭到反对，因为它具有固有的死锁倾向。
    //如果目标线程挂起时在保护关键系统资源的监视器上保持有锁，则在目标线程重新开始以前任何线程都不能访问该资源。
    //如果重新开始目标线程的线程想在调用 resume 之前锁定该监视器，则会发生死锁。
    //这类死锁通常会证明自己是“冻结”的进程。
    //有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。
    //挂起线程。
    //首先，调用线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException（在当前线程中）。
    //如果线程处于活动状态则被挂起，且不再有进一步的活动，除非重新开始。
    @Deprecated
    public final void suspend() {
        checkAccess();
        suspend0();
    }
    
    //已过时。该方法只与 suspend() 一起使用，但 suspend() 已经遭到反对，因为它具有死锁倾向。
    //有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。
    //重新开始挂起的进程。
    //首先，调用线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException（在当前线程中）。
    //如果线程处于活动状态但被挂起，则它会在执行过程中重新开始并允许继续活动。
    @Deprecated
    public final void resume() {
        checkAccess();
        resume0();
    }
    
    //更改线程的优先级。
    //首先调用线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException。
    //在其他情况下，线程优先级被设定为指定的 newPriority 和该线程的线程组的最大允许优先级相比较小的一个。
    public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        //传入的优先级变量必须在1-10之间，否则抛异常。
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }public final void setName(String name) {
        checkAccess();
        this.name = name.toCharArray();
    }
    
        //当前线程的权限大于线程组的权限，则赋予线程组的最大权限。  
        //使用线程组中最大的优先级变量，其实传入的优先级变量不一定是真正的优先级。
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
    
    //返回线程的优先级。
    public final int getPriority() {
        return priority;
    }
    
    //改变线程名称，使之与参数 name 相同。
    //首先调用线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException。
    public final void setName(String name) {
        checkAccess();  //判定当前运行的线程是否有权修改该线程
        this.name = name.toCharArray();
    }
    
    //返回该线程的名称。
    public final String getName() {
        return String.valueOf(name);
    }
    
    //返回该线程所属的线程组。如果该线程已经终止（停止运行），该方法则返回 null。
    public final ThreadGroup getThreadGroup() {
        return group;
    }
    
    //返回当前线程的线程组中活动线程的数目。
    public static int activeCount() {
        return currentThread().getThreadGroup().activeCount();
    }
    
    //将当前线程的线程组及其子组中的每一个活动线程复制到指定的数组中。
    //该方法只调用当前线程的线程组的 enumerate 方法，且带有数组参数。
    //首先，如果有安全管理器，则 enumerate 方法调用安全管理器的 checkAccess 方法，并将线程组作为其参数。
    //这可能导致抛出 SecurityException。
    public static int enumerate(Thread tarray[]) {
        return currentThread().getThreadGroup().enumerate(tarray);
    }
    
    //已过时。该调用的定义依赖于 suspend()，但它遭到了反对。此外，该调用的结果从来都不是意义明确的。
    //计算该线程中的堆栈帧数。线程必须挂起。
    @Deprecated
    public native int countStackFrames();
    
    //等待该线程终止的时间最长为 millis 毫秒。超时为 0 意味着要一直等下去。
    public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        //如果传入的等待时间为0，那判断线程是否活着，如果活着，那么一直等待。
        if (millis == 0) { 
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
    
    //等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。
    public final synchronized void join(long millis, int nanos) throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException("nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        join(millis);
    }
    
    //等待该线程终止。
    public final void join() throws InterruptedException {
        join(0);
    }
    
    //将当前线程的堆栈跟踪打印至标准错误流。该方法仅用于调试。
    public static void dumpStack() {
        new Exception("Stack trace").printStackTrace();
    }
    
    //将该线程标记为守护线程或用户线程。当正在运行的线程都是守护线程时，Java 虚拟机退出。
    //该方法必须在启动线程前调用。
    //该方法首先调用该线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException（在当前线程中）。
    public final void setDaemon(boolean on) {
        checkAccess();
        if (isAlive()) {
            throw new IllegalThreadStateException();
        }
        daemon = on;
    }
    
    //测试该线程是否为守护线程。
    public final boolean isDaemon() {
        return daemon;
    }
    
    //判定当前运行的线程是否有权修改该线程。
    //如果有安全管理器，则调用其 checkAccess 方法，并将该线程作为其参数。这可能导致抛出 SecurityException。
    public final void checkAccess() {
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkAccess(this);
        }
    }
    
    //返回该线程的字符串表示形式，包括线程名称、优先级和线程组。
    public String toString() {
        ThreadGroup group = getThreadGroup();
        if (group != null) {
            return "Thread[" + getName() + "," + getPriority() + "," +
                           group.getName() + "]";
        } else {
            return "Thread[" + getName() + "," + getPriority() + "," +
                            "" + "]";
        }
    }
    
    //返回该线程的上下文 ClassLoader。上下文 ClassLoader 由线程创建者提供，
    //供运行于该线程中的代码在加载类和资源时使用。如果未设定，则默认为父线程的 ClassLoader 上下文。
    //原始线程的上下文 ClassLoader 通常设定为用于加载应用程序的类加载器。
    //首先，如果有安全管理器，并且调用者的类加载器不是 null，
    //也不同于其上下文类加载器正在被请求的线程上下文类加载器的祖先，
    //则通过 RuntimePermission("getClassLoader") 权限调用该安全管理器的 checkPermission 方法，
    //查看是否可以获取上下文 ClassLoader。
    public ClassLoader getContextClassLoader() {
        if (contextClassLoader == null)
            return null;

        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            ClassLoader.checkClassLoaderPermission(contextClassLoader, Reflection.getCallerClass());
        }
        return contextClassLoader;
    }
    
    //设置该线程的上下文 ClassLoader。
    //上下文 ClassLoader 可以在创建线程设置，并允许创建者在加载类和资源时向该线程中运行的代码提供适当的类加载器。
    //首先，如果有安全管理器，则通过 RuntimePermission("setContextClassLoader") 
    //权限调用其 checkPermission 方法，查看是否可以设置上下文 ClassLoader。
    public void setContextClassLoader(ClassLoader cl) {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            sm.checkPermission(new RuntimePermission("setContextClassLoader"));
        }
        contextClassLoader = cl;
    }
    
    //当且仅当当前线程在指定的对象上保持监视器锁时，才返回 true。
    //该方法旨在使程序能够断言当前线程已经保持一个指定的锁：assert Thread.holdsLock(obj);
    public static native boolean holdsLock(Object obj);

    private static final StackTraceElement[] EMPTY_STACK_TRACE = new StackTraceElement[0];
    
    //返回一个表示该线程堆栈转储的堆栈跟踪元素数组。如果该线程尚未启动或已经终止，则该方法将返回一个零长度数组。
    //如果返回的数组不是零长度的，则其第一个元素代表堆栈顶，它是该序列中最新的方法调用。
    //最后一个元素代表堆栈底，是该序列中最旧的方法调用。
    //如果有安全管理器，并且该线程不是当前线程，则通过 RuntimePermission("getStackTrace") 
    //权限调用安全管理器的 checkPermission 方法，查看是否可以获取堆栈跟踪。
    //某些虚拟机在某些情况下可能会从堆栈跟踪中省略一个或多个堆栈帧。
    //在极端情况下，没有该线程堆栈跟踪信息的虚拟机可以从该方法返回一个零长度数组。
    public StackTraceElement[] getStackTrace() {
        if (this != Thread.currentThread()) {
            SecurityManager security = System.getSecurityManager();
            if (security != null) {
                security.checkPermission(SecurityConstants.GET_STACK_TRACE_PERMISSION);
            }
           
            if (!isAlive()) {
                return EMPTY_STACK_TRACE;
            }
            StackTraceElement[][] stackTraceArray = dumpThreads(new Thread[] {this});
            StackTraceElement[] stackTrace = stackTraceArray[0];
            
            if (stackTrace == null) {
                stackTrace = EMPTY_STACK_TRACE;
            }
            return stackTrace;
        } else {
            return (new Exception()).getStackTrace();
        }
    }
    
    //返回所有活动线程的堆栈跟踪的一个映射。映射键是线程，而每个映射值都是一个 StackTraceElement 数组，
    //该数组表示相应 Thread 的堆栈转储。 返回的堆栈跟踪的格式都是针对 getStackTrace 方法指定的。
    //在调用该方法的同时，线程可能也在执行。每个线程的堆栈跟踪仅代表一个快照，并且每个堆栈跟踪都可以在不同时间获得。
    //如果虚拟机没有线程的堆栈跟踪信息，则映射值中将返回一个零长度数组。
    //如果有安全管理器，则通过 RuntimePermission("getStackTrace") 权限和
    //RuntimePermission("modifyThreadGroup") 权限调用其 checkPermission 方法，
    //查看是否可以获取所有线程的堆栈跟踪。
    public static Map<Thread, StackTraceElement[]> getAllStackTraces() {
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkPermission(SecurityConstants.GET_STACK_TRACE_PERMISSION);
            security.checkPermission(SecurityConstants.MODIFY_THREADGROUP_PERMISSION);
        }

        // Get a snapshot of the list of all threads
        Thread[] threads = getThreads();
        StackTraceElement[][] traces = dumpThreads(threads);
        Map<Thread, StackTraceElement[]> m = new HashMap<>(threads.length);
        for (int i = 0; i < threads.length; i++) {
            StackTraceElement[] stackTrace = traces[i];
            if (stackTrace != null) {
                m.put(threads[i], stackTrace);
            }
        }
        return m;
    }
    
    private static final RuntimePermission SUBCLASS_IMPLEMENTATION_PERMISSION =
                    new RuntimePermission("enableContextClassLoaderOverride");
    
    //子类安全审计结果的缓存
    private static class Caches {
        static final ConcurrentMap<WeakClassKey,Boolean> subclassAudits = new ConcurrentHashMap<>();

        //子队列弱引用审计
        static final ReferenceQueue<Class<?>> subclassAuditsQueue = new ReferenceQueue<>();
    }
    
    //证明这个（可能是子类）实例可以在不违反安全约束的条件下被构造：子类不能重写安全敏感的非final方法，
    //并且“enableContextClassLoaderOverride”，检查RuntimePermission。    
    private static boolean isCCLOverridden(Class cl) {
        if (cl == Thread.class)
            return false;

        processQueue(Caches.subclassAuditsQueue, Caches.subclassAudits);
        WeakClassKey key = new WeakClassKey(cl, Caches.subclassAuditsQueue);
        Boolean result = Caches.subclassAudits.get(key);
        if (result == null) {
            result = Boolean.valueOf(auditSubclass(cl));
            Caches.subclassAudits.putIfAbsent(key, result);
        }

        return result.booleanValue();
    }
    
    //对给定子类进行反射检查，以验证它不覆盖安全敏感的非最终方法。如果子类重写任何方法返回true，否则为false。
    private static boolean auditSubclass(final Class subcl) {
        Boolean result = AccessController.doPrivileged(
            new PrivilegedAction<Boolean>() {
                public Boolean run() {
                    for (Class cl = subcl; cl != Thread.class; cl = cl.getSuperclass()) {
                        try {
                            cl.getDeclaredMethod("getContextClassLoader", new Class[0]);
                            return Boolean.TRUE;
                        } catch (NoSuchMethodException ex) {
                        }
                        try {
                            Class[] params = {ClassLoader.class};
                            cl.getDeclaredMethod("setContextClassLoader", params);
                            return Boolean.TRUE;
                        } catch (NoSuchMethodException ex) {
                        }
                    }
                    return Boolean.FALSE;
                }
            }
        );
        return result.booleanValue();
    }

    private native static StackTraceElement[][] dumpThreads(Thread[] threads);
    private native static Thread[] getThreads();
    
    //返回该线程的标识符。线程 ID 是一个正的 long 数，在创建该线程时生成。
    //线程 ID 是唯一的，并终生不变。线程终止时，该线程 ID 可以被重新使用。
    public long getId() {
        return tid;
    }
    
    //线程状态
    //在给定时间点上，一个线程只能处于一种状态。这些状态是虚拟机状态，它们并没有反映所有操作系统线程状态。
    public enum State {
        //至今尚未启动的线程处于这种状态。 
        NEW,

        //可运行线程的线程状态。
        //处于可运行状态的某一线程正在 Java 虚拟机中运行，但它可能正在等待操作系统中的其他资源，比如处理器。 
        RUNNABLE,

        //受阻塞并且正在等待监视器锁的某一线程的线程状态。
        //处于受阻塞状态的某一线程正在等待监视器锁，以便进入一个同步的块/方法，
        //或者在调用 Object.wait 之后再次进入同步的块/方法。 
        BLOCKED,

        //无限期地等待另一个线程来执行某一特定操作的线程处于这种状态。 
        //某一等待线程的线程状态。某一线程因为调用下列方法之一而处于等待状态： 
        //   不带超时值的 Object.wait
        //   不带超时值的 Thread.join
        //   LockSupport.park 
        //处于等待状态的线程正等待另一个线程，以执行特定操作。 
        //例如，已经在某一对象上调用了 Object.wait() 的线程正等待另一个线程，
        //以便在该对象上调用 Object.notify() 或 Object.notifyAll()。
        //已经调用了 Thread.join() 的线程正在等待指定线程终止。 
        WAITING,

        //具有指定等待时间的某一等待线程的线程状态。
        //某一线程因为调用以下带有指定正等待时间的方法之一而处于定时等待状态： 
        //   Thread.sleep 
        //   带有超时值的 Object.wait 
        //   带有超时值的 Thread.join 
        //   LockSupport.parkNanos 
        //   LockSupport.parkUntil 
        TIMED_WAITING,

        //已退出的线程处于这种状态。 
        TERMINATED;
    }
    
    //返回该线程的状态。该方法用于监视系统状态，不用于同步控制。
    public State getState() {
        return sun.misc.VM.toThreadState(threadStatus);
    }
    
    //当 Thread 因未捕获的异常而突然终止时，调用处理程序的接口。 
    //当某一线程因未捕获的异常而即将终止时，Java 虚拟机将使用 Thread.getUncaughtExceptionHandler() 
    //查询该线程以获得其 UncaughtExceptionHandler 的线程，并调用处理程序的 uncaughtException 方法，
    //将线程和异常作为参数传递。如果某一线程没有明确设置其 UncaughtExceptionHandler，
    //则将它的 ThreadGroup 对象作为其 UncaughtExceptionHandler。
    //如果 ThreadGroup 对象对处理异常没有什么特殊要求，那么它可以将调用转发给默认的未捕获异常处理程序。 
    public interface UncaughtExceptionHandler {
        void uncaughtException(Thread t, Throwable e);
    }
    
    private volatile UncaughtExceptionHandler uncaughtExceptionHandler;

    private static volatile UncaughtExceptionHandler defaultUncaughtExceptionHandler;
    
    //设置当线程由于未捕获到异常而突然终止，并且没有为该线程定义其他处理程序时所调用的默认处理程序。
    //未捕获到的异常处理首先由线程控制，然后由线程的 ThreadGroup 对象控制，
    //最后由未捕获到的默认异常处理程序控制。如果线程不设置明确的未捕获到的异常处理程序，
    //并且该线程的线程组（包括父线程组）未特别指定其 uncaughtException 方法，
    //则将调用默认处理程序的 uncaughtException 方法。
    //通过设置未捕获到的默认异常处理程序，
    //应用程序可以为那些已经接受系统提供的任何“默认”行为的线程改变未捕获到的异常处理方式
    //（如记录到某一特定设备或文件）。
    //请注意，未捕获到的默认异常处理程序通常不应顺从该线程的 ThreadGroup 对象，因为这可能导致无限递归。
    public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler eh) {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            sm.checkPermission(new RuntimePermission("setDefaultUncaughtExceptionHandler"));
        }

        defaultUncaughtExceptionHandler = eh;
    }
    
    //返回线程由于未捕获到异常而突然终止时调用的默认处理程序。如果返回值为 null，则没有默认处理程序。
    public static UncaughtExceptionHandler getDefaultUncaughtExceptionHandler(){
        return defaultUncaughtExceptionHandler;
    }
    
    //返回该线程由于未捕获到异常而突然终止时调用的处理程序。如果该线程尚未明确设置未捕获到的异常处理程序，
    //则返回该线程的 ThreadGroup 对象，除非该线程已经终止，在这种情况下，将返回 null。
    public UncaughtExceptionHandler getUncaughtExceptionHandler() {
        return uncaughtExceptionHandler != null ? uncaughtExceptionHandler : group;
    }

    //设置该线程由于未捕获到异常而突然终止时调用的处理程序。
    //通过明确设置未捕获到的异常处理程序，线程可以完全控制它对未捕获到的异常作出响应的方式。 
    //如果没有设置这样的处理程序，则该线程的 ThreadGroup 对象将充当其处理程序。
    public void setUncaughtExceptionHandler(UncaughtExceptionHandler eh) {
        checkAccess();
        uncaughtExceptionHandler = eh;
    }
    
    
  }
```
