* StringBuilder类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  StringBuilder是非线程安全的，可变的字符序列，继承AbstractStringBuilder抽象类。
```java
  public final class StringBuilder extends AbstractStringBuilder
      implements java.io.Serializable, CharSequence {
    //序列化版本号
    static final long serialVersionUID = 4383685877147921099L;
      
    //构造一个初始为空，容量大小为16的字符序列
    public StringBuilder() {
      super(16);
    }
      
    //构造一个初始为空，容量为capacity大小的字符序列
    public StringBuilder(int capacity) {
      super(capacity);
    }
      
    //构造一个初始为str，容量为str.length() + 16大小的字符序列
    public StringBuilder(String str) {
      super(str.length() + 16);
      append(str);
    }
    public StringBuilder(CharSequence seq) {...}
      
    //将字符串str添加到原序列末尾
    public StringBuilder append(String str) {
       super.append(str);
       return this;
    }
    public StringBuilder append(Object obj) {...}
      
    //将新字符序列sb添加到原字符序列末尾
    private StringBuilder append(StringBuilder sb) {
        if (sb == null)
            return append("null");
        int len = sb.length();
        int newcount = count + len;
        if (newcount > value.length)
            expandCapacity(newcount);
        sb.getChars(0, len, value, count);
        count = newcount;
        return this;
    }
    public StringBuilder append(StringBuffer sb) {...}
    
    public StringBuilder append(CharSequence s) {...}
    public StringBuilder append(CharSequence s, int start, int end) {...}
    
    public StringBuilder append(char[] str) {...}
    public StringBuilder append(char[] str, int offset, int len) {...}
    
    public StringBuilder append(boolean b) {...}
    public StringBuilder append(char c) {...}
    public StringBuilder append(int i) {...}
    public StringBuilder append(long lng) {...}
    public StringBuilder append(float f) {...}
    public StringBuilder append(double d) {...}
    
    public StringBuilder appendCodePoint(int codePoint) {...}
    
    //删除原序列中从start开始到end结束的序列
    public StringBuilder delete(int start, int end) {
        super.delete(start, end);
        return this;
    }
    
    //删除原序列中指定索引index位置的字符
    public StringBuilder deleteCharAt(int index) {
        super.deleteCharAt(index);
        return this;
    }
    
    //将原序列中从start开始到end结束的子序列替换为字符串str
    public StringBuilder replace(int start, int end, String str) {
        super.replace(start, end, str);
        return this;
    }
    
    //在原字符序列中插入字符数组str
    public StringBuilder insert(int index, char[] str, int offset, int len) {
        super.insert(index, str, offset, len);
        return this;
    }
    public StringBuilder insert(int offset, Object obj) {...}
    public StringBuilder insert(int offset, String str) {...}
    public StringBuilder insert(int offset, char[] str) {...}
    
    public StringBuilder insert(int dstOffset, CharSequence s) {...}
    public StringBuilder insert(int dstOffset, CharSequence s, int start, int end) {...}
    
    public StringBuilder insert(int offset, boolean b) {...}
    public StringBuilder insert(int offset, char c) {...}
    public StringBuilder insert(int offset, int i) {...}
    public StringBuilder insert(int offset, long l) {...}
    public StringBuilder insert(int offset, float f) {...}
    public StringBuilder insert(int offset, double d) {...}
    
    public int indexOf(String str) {...}
    //返回字符串在序列中的起始位置
    public int indexOf(String str, int fromIndex) {
        return String.indexOf(value, 0, count,
                              str.toCharArray(), 0, str.length(), fromIndex);
    }
    
    public int lastIndexOf(String str) {...}
    public int lastIndexOf(String str, int fromIndex) {...}
    
    //翻转字符序列
    public StringBuilder reverse() {
        super.reverse();
        return this;
    }
    
    public String toString() {
        // 创建一个副本，不要共享数组
        return new String(value, 0, count);
    }
    
    //序列化StringBuilder实例
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        s.defaultWriteObject();
        s.writeInt(count);
        s.writeObject(value);
    }
    //反序列化StringBuilder实例
    private void readObject(java.io.ObjectInputStream s){...}
    
  }
```
