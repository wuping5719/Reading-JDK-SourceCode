* Error类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  Error类:    
  &nbsp;&nbsp; (1) 总是不可控制的(unchecked)。     
  &nbsp;&nbsp; (2) 经常用来用于表示系统错误或低层资源的错误。    
  &nbsp;&nbsp; (3) 如果可能的话，应该在系统级被捕捉。

```java
  public class Error extends Throwable {
     // 版本号
     static final long serialVersionUID = 4980196508277280342L;
     
     //构造详细消息为null的新错误。原因尚未进行初始化，可在以后通过调用Throwable.initCause(Throwable)对其进行初始化。
     public Error() {
        super();
     }
     
     //构造带指定详细消息的新错误。原因尚未进行初始化，可在以后通过调用Throwable.initCause(Throwable)对其进行初始化。
     public Error(String message) {
        super(message);
     }
     
     //构造带指定详细消息和原因的新错误。注意: 与cause相关的详细消息不是自动合并到这个错误的详细消息中的。
     public Error(String message, Throwable cause) {
        super(message, cause);
     }
     
     //根据指定的原因和(cause==null ? null :cause.toString())的详细消息来构造新的错误(通常包含cause的类和详细消息)
     //对于错误而言，此构造方法与其他Throwable的包装器一样有用。
     public Error(Throwable cause) {
        super(cause);
     }
     
     protected Error(String message, Throwable cause, boolean enableSuppression,
                    boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
     }
  }
```
