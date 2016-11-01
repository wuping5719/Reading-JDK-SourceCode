* StringBuffer类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  StringBuffer是线程安全的，可变的字符序列，继承AbstractStringBuilder抽象类。
```java
  public final class StringBuffer extends AbstractStringBuilder
       implements java.io.Serializable, CharSequence {
       //序列化全局版本号
       static final long serialVersionUID = 3388685877147921107L;
       
       //默认构造函数：构造一个初始容量为16的，没有字符的字符串缓冲区。
       public StringBuffer() {
         super(16);
       }
       
       //构造一个指定容量为capacity的，没有字符的字符串缓冲区。
       public StringBuffer(int capacity) {
          super(capacity);
       }
       
       //构造一个初始化为指定字符串内容的字符串缓冲区。字符串缓冲区的初始容量为16，加上字符串参数的长度。
       public StringBuffer(String str) {
         super(str.length() + 16);
         append(str);
       }
       
       public StringBuffer(CharSequence seq) {...}
       
       //序列长度
       public synchronized int length() {
          return count;
       }

       //初始容量
       public synchronized int capacity() {
         return value.length;
       }
       
       //小于最小容量时扩大容量
       public synchronized void ensureCapacity(int minimumCapacity) {
          if (minimumCapacity > value.length) {
            expandCapacity(minimumCapacity);
          }
       }
       
       public synchronized void trimToSize() {...}
       public synchronized void setLength(int newLength) {...}
       
       //获取索引位置的字符
       public synchronized char charAt(int index) {
          if ((index < 0) || (index >= count))
             throw new StringIndexOutOfBoundsException(index);
          return value[index];
       }
       
       public synchronized int codePointAt(int index) {...}
       public synchronized int codePointBefore(int index) {...}
       public synchronized int codePointCount(int beginIndex, int endIndex) {...}
       public synchronized int offsetByCodePoints(int index, int codePointOffset) {...}
       
       public synchronized void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin) {...}
       
       //修改指定索引位置的字符
       public synchronized void setCharAt(int index, char ch) {
          if ((index < 0) || (index >= count))
             throw new StringIndexOutOfBoundsException(index);
          value[index] = ch;
       }
       
       //在原字符序列后添加新字符串
       public synchronized StringBuffer append(String str) {
          super.append(str);
          return this;
       }
       public synchronized StringBuffer append(Object obj) {...}
       public synchronized StringBuffer append(StringBuffer sb) {...}
       
       public StringBuffer append(CharSequence s) {
         // 注意: 通过调用实现同步
          if (s == null)
             s = "null";
          if (s instanceof String)
             return this.append((String)s);
          if (s instanceof StringBuffer)
             return this.append((StringBuffer)s);
          return this.append(s, 0, s.length());
       }
       public synchronized StringBuffer append(CharSequence s, int start, int end) {...}
       
       public synchronized StringBuffer append(char[] str) {...}
       public synchronized StringBuffer append(char[] str, int offset, int len) {...}
       
       //在原字符序列后添加基本类型：boolean，char，int，long，float，double
       public synchronized StringBuffer append(boolean b) {...}
       public synchronized StringBuffer append(char c) {...}
       public synchronized StringBuffer append(int i) {...}
       public synchronized StringBuffer append(long lng) {...}
       public synchronized StringBuffer append(float f) {...}
       public synchronized StringBuffer append(double d) {...}
       
       public synchronized StringBuffer appendCodePoint(int codePoint) {...}
       
       //删除原字符序列中从start到end的字符
       public synchronized StringBuffer delete(int start, int end) {
         super.delete(start, end);
         return this;
       }
       
       //删除原字符序列中指定索引位置的字符
       public synchronized StringBuffer deleteCharAt(int index) {
          super.deleteCharAt(index);
          return this;
       }
       
       //把原字符序列中从start到end的子序列替换为参数串str
       public synchronized StringBuffer replace(int start, int end, String str) {
          super.replace(start, end, str);
          return this;
       }
       
       //获取原字符序列中从start开始的子串
       public synchronized String substring(int start) {
          return substring(start, count);
       }
       public synchronized CharSequence subSequence(int start, int end) {...}
       
       //获取原字符序列中从start开始到end结束的子串
       public synchronized String substring(int start, int end) {
         return super.substring(start, end);
       }
       
       //往原序列中插入字符数组
       public synchronized StringBuffer insert(int index, char[] str, int offset, int len)
       {
          super.insert(index, str, offset, len);
          return this;
       }
       public synchronized StringBuffer insert(int offset, Object obj) {...}
       public synchronized StringBuffer insert(int offset, String str) {...}
       public synchronized StringBuffer insert(int offset, char[] str) {...}
       public StringBuffer insert(int dstOffset, CharSequence s) {...}
       public synchronized StringBuffer insert(int dstOffset, CharSequence s, int start, int end) {...}
       
       public StringBuffer insert(int offset, boolean b) {...}
       public synchronized StringBuffer insert(int offset, char c) {...}
       public StringBuffer insert(int offset, int i) {...}
       public StringBuffer insert(int offset, long l) {...}
       public StringBuffer insert(int offset, float f) {...}
       public StringBuffer insert(int offset, double d) {...}
       
       public int indexOf(String str) {
            return indexOf(str, 0);
       }
       public synchronized int indexOf(String str, int fromIndex) {...}
       
       public int lastIndexOf(String str) {
          return lastIndexOf(str, count);
       }
       public synchronized int lastIndexOf(String str, int fromIndex) {...}
       
       //翻转原序列
       public synchronized StringBuffer reverse() {
          super.reverse();
          return this;
       }
       
       public synchronized String toString() {...}
       
       //StringBuffer的可序列化字段
       private static final java.io.ObjectStreamField[] serialPersistentFields =
       {
          new java.io.ObjectStreamField("value", char[].class),
          new java.io.ObjectStreamField("count", Integer.TYPE),
          new java.io.ObjectStreamField("shared", Boolean.TYPE),
       };
       private synchronized void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException {...}
       private void readObject(java.io.ObjectInputStream s) throws java.io.IOException, ClassNotFoundException {...}
       
  }
```
