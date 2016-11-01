* InternalError类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  InternalError类: 该异常指示Java虚拟机中出现一些意外的内部错误。
  
```java
  public class InternalError extends VirtualMachineError {
    private static final long serialVersionUID = -9062593416125562365L;
    
    //构造不带详细消息的InternalError
    public InternalError() {
        super();
    }
    
    //构造带指定详细消息的InternalError
    public InternalError(String s) {
        super(s);
    }
  }
```
