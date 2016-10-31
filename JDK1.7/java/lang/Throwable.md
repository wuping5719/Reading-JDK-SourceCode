* Throwable类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Throwable 类是Java语言中所有错误或异常的超类。只有当对象是此类（或其子类之一）的实例时，才能通过Java虚拟机或者Java throw语句抛出。类似地，只有此类或其子类之一才可以是catch子句中的参数类型。    
  &nbsp;&nbsp; 两个子类的实例: Error和Exception，通常用于指示发生了异常情况。通常，这些实例是在异常情况的上下文中新近创建的，因此包含了相关的信息（比如堆栈跟踪数据）。   
  &nbsp;&nbsp; Exception类及其子类是Throwable的一种形式，它指出了合理的应用程序想要捕获的条件。   
  &nbsp;&nbsp; 注意它有四个构造函数:    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable()     
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(String message)    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(Throwable cause)    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(String message, Throwable cause)
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
    
    //javaThrowable源码解析:http://wenku.baidu.com/link?url=RpVDFsboE00Ccpte1NrKl-FdA_Z1B_LAXQWSb7Dp8MefRW6PAHJX7MDKT1RlDLRypSJ40S1k5iyRRERPcZj6A3J3puXo-gNY35HwgCNp26a
  } 
```
