* StackOverflowError类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  StackOverflowError类: 堆栈溢出错误。当一个应用递归调用的层次太深而导致堆栈溢出时抛出该错误。
  
```java
  public class StackOverflowError extends VirtualMachineError {
    //版本号
    private static final long serialVersionUID = 8609175038441759607L;
    
    //构造不带详细消息的StackOverflowError
    public StackOverflowError() {
        super();
    }
    
    //构造带指定详细消息的StackOverflowError
    public StackOverflowError(String s) {
        super(s);
    }
  }
```
