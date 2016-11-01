* UnknownError类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  UnknownError类: 当Java虚拟机中出现一个未知但严重的异常时，抛出该错误。
  
```java
  public class UnknownError extends VirtualMachineError {
    private static final long serialVersionUID = 2524784860676771849L;
    
    //构造不带详细消息的UnknownError
    public UnknownError() {
        super();
    }
    
    //构造带指定详细消息的UnknownError
    public UnknownError(String s) {
        super(s);
    }
  }
```
