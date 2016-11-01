* Exception类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  Java中的异常分为两大类：
  &nbsp;&nbsp; (1) 运行时异常(Runtime Exception)(Unchecked Exception).    
  &nbsp;&nbsp; (2) 非运行时异常(Checked Exception).    

```java
  public class Exception extends Throwable {
    static final long serialVersionUID = -3387516993124229948L;
    
    //构造详细消息为null的新异常
    public Exception() {
        super();
    }
    
    //构造带指定详细消息的新异常
    public Exception(String message) {
        super(message);
    }
    
    //构造带指定详细消息和原因的新异常
    public Exception(String message, Throwable cause) {
        super(message, cause);
    }
    
    //根据指定的原因和(cause==null ? null : cause.toString())的详细消息构造新异常(它通常包含cause的类和详细消息)
    public Exception(Throwable cause) {
        super(cause);
    }
    
    protected Exception(String message, Throwable cause, boolean enableSuppression,
                        boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
  }
```
