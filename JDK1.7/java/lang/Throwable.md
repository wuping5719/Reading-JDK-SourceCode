* Throwable类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Throwable 类是Java语言中所有错误或异常的超类。只有当对象是此类（或其子类之一）的实例时，才能通过Java虚拟机或者Java throw语句抛出。类似地，只有此类或其子类之一才可以是catch子句中的参数类型。    
  &nbsp;&nbsp; 两个子类的实例: Error和Exception，通常用于指示发生了异常情况。通常，这些实例是在异常情况的上下文中新近创建的，因此包含了相关的信息（比如堆栈跟踪数据）。   
  &nbsp;&nbsp; Exception类及其子类是Throwable的一种形式，它指出了合理的应用程序想要捕获的条件。   
  &nbsp;&nbsp; 注意它有四个构造函数:    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable()     
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(String message)    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(Throwable cause)    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(String message, Throwable cause)     
* 深入理解java异常处理机制：&nbsp; <http://blog.csdn.net/hguisu/article/details/6155636>
<p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_Throwable.png" /></p>

```java
  public class Throwable implements Serializable {
    //版本号
    private static final long serialVersionUID = -3042686055658047285L;

    //保存断点处的栈回溯信息  
    private transient Object backtrace;

    //描述此异常的信息  
    private String detailMessage;
    
    //仅用于序列化对象，推迟初始化的哨兵容器类。
    private static class SentinelHolder {...}
    
    //空栈的共享值
    private static final StackTraceElement[] UNASSIGNED_STACK = new StackTraceElement[0];
    
    //表示当前异常由那个Throwable引起, 如果为null表示此异常不是由其他Throwable引起的,
    //如果此对象与自己相同, 表明此异常的起因对象还没有被初始化.
    private Throwable cause = this;
    
    //描述异常轨迹的数组
    private StackTraceElement[] stackTrace = UNASSIGNED_STACK;
    
    private static final List<Throwable> SUPPRESSED_SENTINEL =
        Collections.unmodifiableList(new ArrayList<Throwable>(0));
    
    //被抑制的异常列表
    private List<Throwable> suppressedExceptions = SUPPRESSED_SENTINEL;
    
    //消息字符串：试图抑制空异常
    private static final String NULL_CAUSE_MESSAGE = "Cannot suppress a null exception.";
    
    //消息字符串：试图自我抑制
    private static final String SELF_SUPPRESSION_MESSAGE = "Self-suppression not permitted";
    
    //消息字符串：标记异常堆栈跟踪的标题
    private static final String CAUSE_CAPTION = "Caused by: ";
    
    //消息字符串：用于标记抑制异常堆栈跟踪的标题
    private static final String SUPPRESSED_CAPTION = "Suppressed: ";
    
    //构造函数, 起因对象没有被初始化时可以在以后使用initCause进行初始化, fillInStackTrace可以用来初始化它的异常轨迹数组.
    public Throwable() {
        fillInStackTrace();
    }
    
    //带有细节信息参数的构造函数
    public Throwable(String message) {
        fillInStackTrace();          //填充异常轨迹数组
        detailMessage = message;     //初始化异常描述信息
    }
    
    //cause表示起因对象, 带有细节信息和造成异常原因参数的构造函数, 
    //与异常原因相关的细节信息不会自动写入Throwable类的详细信息中.
    public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }
    
    //参数为起因对象cause的构造函数
    public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
    }
    
    //带有启用禁用异常抑制，启用禁用堆栈跟踪等参数的保护构造函数
    protected Throwable(String message, Throwable cause, boolean enableSuppression,
                        boolean writableStackTrace) {
        if (writableStackTrace) {
            fillInStackTrace();
        } else {
            stackTrace = null;
        }
        detailMessage = message;
        this.cause = cause;
        if (!enableSuppression)
            suppressedExceptions = null;
   }
   
   //获取详细信息
   public String getMessage() {
        return detailMessage;
   }
   
   //创建该异常的局部描述
   public String getLocalizedMessage() {
        return getMessage();
   }
   
   //同步获取造成异常的起因对象，如果起因不存在或未知，则返回null.
   public synchronized Throwable getCause() {
        return (cause==this ? null : cause);
   }
   
   //初始化起因对象, 这个方法只能在未被初始化的情况下调用一次 
   public synchronized Throwable initCause(Throwable cause) {
        if (this.cause != this)  //如果不是未初始化状态则抛出异常
            throw new IllegalStateException("Can't overwrite cause with " +
                                            Objects.toString(cause, "a null"), this);
        if (cause == this)       //要设置的起因对象与自身相等则抛出异常  
            throw new IllegalArgumentException("Self-causation not permitted", this);
        this.cause = cause;     //设置起因对象 
        return this;            //返回设置的起因的对象
   }
   
   //字符串表示形式
   public String toString() {...}
   
   //打印错误轨迹  
   public void printStackTrace() {
        printStackTrace(System.err);
   }
   
   //打印错误轨迹
   public void printStackTrace(PrintStream s) {
        printStackTrace(new WrappedPrintStream(s));
   }
   private void printStackTrace(PrintStreamOrWriter s) {
        // 通过提供一个身份平等的语义的堆来防范恶意覆盖Throwable.equals
        Set<Throwable> dejaVu =
            Collections.newSetFromMap(new IdentityHashMap<Throwable, Boolean>());
        dejaVu.add(this);

        synchronized (s.lock()) {
            //打印堆栈信息
            s.println(this);
            StackTraceElement[] trace = getOurStackTrace();
            for (StackTraceElement traceElement : trace)
                s.println("\tat " + traceElement);

            //如果有被抑制的异常，则打印出来
            for (Throwable se : getSuppressed())
                se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);

            //如果有异常起因，则打印异常起因
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
        }
    }
    
    //将堆栈跟踪作为指定堆栈跟踪的一个封闭的异常打印
    private void printEnclosedStackTrace(PrintStreamOrWriter s, StackTraceElement[] enclosingTrace,
                                         String caption, String prefix, Set<Throwable> dejaVu) {
        assert Thread.holdsLock(s.lock());
        if (dejaVu.contains(this)) {
            s.println("\t[CIRCULAR REFERENCE:" + this + "]");
        } else {
            dejaVu.add(this);
            // 计算和外围跟踪之间普通帧的数量
            StackTraceElement[] trace = getOurStackTrace();
            int m = trace.length - 1;            //m为当前异常轨迹数组的最后一个元素位置,   
            int n = enclosingTrace.length - 1;   //n为封闭异常的异常轨迹数组的最后一个元素  
            // 分别从两个数组的后面做循环, 如果相等则一直循环, 直到不等或数组到头
            while (m >= 0 && n >=0 && trace[m].equals(enclosingTrace[n])) {
                m--; n--;
            }
            int framesInCommon = trace.length - 1 - m;    //相同的个数

            // 打印堆栈信息
            s.println(prefix + caption + this);
            for (int i = 0; i <= m; i++)
                s.println(prefix + "\tat " + trace[i]);
            if (framesInCommon != 0)             //如果有相同的则打印出相同的个数 
                s.println(prefix + "\t... " + framesInCommon + " more");

            // 如果有被抑制的异常，则打印出来
            for (Throwable se : getSuppressed())
                se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION,
                                           prefix +"\t", dejaVu);

            // 如果有异常起因，则打印异常起因
            Throwable ourCause = getCause();
            if (ourCause != null)
                ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, prefix, dejaVu);
        }
    }
    
    //打印错误轨迹
    public void printStackTrace(PrintWriter s) {
        printStackTrace(new WrappedPrintWriter(s));
    }
    
    private abstract static class PrintStreamOrWriter {
        //当使用StreamOrWriter时，返回被锁定的对象
        abstract Object lock();

        ///打印StreamOrWriter一行
        abstract void println(Object o);
    }
    
    private static class WrappedPrintStream extends PrintStreamOrWriter {...}
    private static class WrappedPrintWriter extends PrintStreamOrWriter {...}
    
    //填充异常轨迹, 记录当前线程执行的堆栈帧状态
    public synchronized Throwable fillInStackTrace() {
        if (stackTrace != null ||
            backtrace != null /* 脱离协议状态 */ ) {
            fillInStackTrace(0);
            stackTrace = UNASSIGNED_STACK;
        }
        return this;
    }
    // 本地方法
    private native Throwable fillInStackTrace(int dummy);
    
    //返回当前的异常轨迹的拷贝
    public StackTraceElement[] getStackTrace() {
        return getOurStackTrace().clone();
    }
    
    //获取当前的异常轨迹
    private synchronized StackTraceElement[] getOurStackTrace() {
        // 如果第一次调用此方法则初始化异常轨迹数组
        if (stackTrace == UNASSIGNED_STACK ||
            (stackTrace == null && backtrace != null)) {
            int depth = getStackTraceDepth();                    //获得异常轨迹深度
            stackTrace = new StackTraceElement[depth];           //创建新的异常轨迹数组, 并填充它
            for (int i=0; i < depth; i++)
                stackTrace[i] = getStackTraceElement(i);         //获取指定位标的异常轨迹
        } else if (stackTrace == null) {
            return UNASSIGNED_STACK;
        }
        return stackTrace;
    }
    
    // 设置异常轨迹
    public void setStackTrace(StackTraceElement[] stackTrace) {
        // 验证参数
        StackTraceElement[] defensiveCopy = stackTrace.clone();
        for (int i = 0; i < defensiveCopy.length; i++) {
            if (defensiveCopy[i] == null)         // 如果设置参数有空元素则抛出异常  
                throw new NullPointerException("stackTrace[" + i + "]");
        }

        synchronized (this) {
            if (this.stackTrace == null && // 不可变的栈
                backtrace == null) // 测试协议状态
                return;
            this.stackTrace = defensiveCopy;
        }
    }
    
    //异常轨迹的深度, 0表示无法获得
    native int getStackTraceDepth();
    
    // 获取指定位标的异常轨迹
    native StackTraceElement getStackTraceElement(int index);
    
    //反序列化
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {...}
    //序列化
    private synchronized void writeObject(ObjectOutputStream s) throws IOException {...}
    
    //添加被抑制的异常
    public final synchronized void addSuppressed(Throwable exception) {
        if (exception == this)
            throw new IllegalArgumentException(SELF_SUPPRESSION_MESSAGE, exception);

        if (exception == null)
            throw new NullPointerException(NULL_CAUSE_MESSAGE);

        if (suppressedExceptions == null) // 不记录被抑制的异常
            return;

        if (suppressedExceptions == SUPPRESSED_SENTINEL)
            suppressedExceptions = new ArrayList<>(1);

        suppressedExceptions.add(exception);
    }

    private static final Throwable[] EMPTY_THROWABLE_ARRAY = new Throwable[0];
    
    public final synchronized Throwable[] getSuppressed() {
        if (suppressedExceptions == SUPPRESSED_SENTINEL ||
            suppressedExceptions == null)
            return EMPTY_THROWABLE_ARRAY;
        else
            return suppressedExceptions.toArray(EMPTY_THROWABLE_ARRAY);
    }
  } 
```
