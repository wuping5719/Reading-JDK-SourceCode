* AbstractStringBuilder抽象类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  AbstractStringBuilder抽象类是一个可变的字符序列, 非线程安全。
```java
  abstract class AbstractStringBuilder implements Appendable, CharSequence {
     //以字符数组形式存储
     char[] value;
     
     //使用的字符数
     int count;
     
     //子类序列化必须的无参构造函数
     AbstractStringBuilder() {
     }
     
     //创建指定的容量AbstractStringBuilder 
     AbstractStringBuilder(int capacity) {
        value = new char[capacity];
     }
     
     //返回长度（字符数）
     public int length() {
        return count;
     }
     
     //返回当前容量。容量是可用于新插入的字符的存储量，超出容量重新分配空间
     public int capacity() {
        return value.length;
     }
     
     //确保容量至少等于指定的最小值。如果当前容量小于参数，那么新的内部数组将被分配更大的容量。
     public void ensureCapacity(int minimumCapacity) {
        if (minimumCapacity > 0)
            ensureCapacityInternal(minimumCapacity);
     }
     //该方法与ensureCapacity类似，但不是同步的。
     private void ensureCapacityInternal(int minimumCapacity) {
        if (minimumCapacity - value.length > 0)
            expandCapacity(minimumCapacity);
     }
     //扩展容量，新的容量是：MinimumCapacity和oldCapacity *2 + 2中的较大值。
     void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // 溢出
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
    
    //试图减少已使用字符序列的存储。
    public void trimToSize() {
        if (count < value.length) {
            value = Arrays.copyOf(value, count);
        }
    }
    
    //设置字符序列的长度。
    public void setLength(int newLength) {
        if (newLength < 0)
            throw new StringIndexOutOfBoundsException(newLength);
        ensureCapacityInternal(newLength);

        if (count < newLength) {
            for (; count < newLength; count++)
                value[count] = '\0';
        } else {
            count = newLength;
        }
    }
    
    //获取指定索引位置的字符
    public char charAt(int index) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        return value[index];
    }
    
    public int codePointAt(int index) {...}
    public int codePointBefore(int index) {...}
    public int codePointCount(int beginIndex, int endIndex) {...}
    public int offsetByCodePoints(int index, int codePointOffset) {...}
    
    //把源序列的字符复制到目标字符数组DST
    public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin) {...}
    
    //将索引处的字符设置为参数字符ch
    public void setCharAt(int index, char ch) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        value[index] = ch;
    }
    
    //将指定字符串添加到原字符序列末尾
    public AbstractStringBuilder append(String str) {
        if (str == null) str = "null";
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
    public AbstractStringBuilder append(Object obj) {
        return append(String.valueOf(obj));
    }
    public AbstractStringBuilder append(StringBuffer sb) {...}
    public AbstractStringBuilder append(CharSequence s) {...}
    public AbstractStringBuilder append(CharSequence s, int start, int end) {...}
    public AbstractStringBuilder append(char[] str) {...}
    public AbstractStringBuilder append(char str[], int offset, int len) {...}
    
    //将布尔值添加到原字符序列末尾
    public AbstractStringBuilder append(boolean b) {
        if (b) {
            ensureCapacityInternal(count + 4);
            value[count++] = 't';
            value[count++] = 'r';
            value[count++] = 'u';
            value[count++] = 'e';
        } else {
            ensureCapacityInternal(count + 5);
            value[count++] = 'f';
            value[count++] = 'a';
            value[count++] = 'l';
            value[count++] = 's';
            value[count++] = 'e';
        }
        return this;
    }
    
    //将字符添加到原字符序列末尾
    public AbstractStringBuilder append(char c) {
        ensureCapacityInternal(count + 1);
        value[count++] = c;
        return this;
    }
    
    //将整数添加到原字符序列末尾
    public AbstractStringBuilder append(int i) {
        if (i == Integer.MIN_VALUE) {
            append("-2147483648");
            return this;
        }
        int appendedLength = (i < 0) ? Integer.stringSize(-i) + 1
                                     : Integer.stringSize(i);
        int spaceNeeded = count + appendedLength;
        ensureCapacityInternal(spaceNeeded);
        Integer.getChars(i, spaceNeeded, value);
        count = spaceNeeded;
        return this;
    }
    
    //将长整数添加到原字符序列末尾
    public AbstractStringBuilder append(long l) {
        if (l == Long.MIN_VALUE) {
            append("-9223372036854775808");
            return this;
        }
        int appendedLength = (l < 0) ? Long.stringSize(-l) + 1
                                     : Long.stringSize(l);
        int spaceNeeded = count + appendedLength;
        ensureCapacityInternal(spaceNeeded);
        Long.getChars(l, spaceNeeded, value);
        count = spaceNeeded;
        return this;
    }
    
    //将浮点数添加到原字符序列末尾
    public AbstractStringBuilder append(float f) {
        new FloatingDecimal(f).appendTo(this);
        return this;
    }
    public AbstractStringBuilder append(double d) {
        new FloatingDecimal(d).appendTo(this);
        return this;
    }
    
    //删除序列中从起始位置start开始到结束位置end的字符
    public AbstractStringBuilder delete(int start, int end) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (end > count)
            end = count;
        if (start > end)
            throw new StringIndexOutOfBoundsException();
        int len = end - start;
        if (len > 0) {
            System.arraycopy(value, start+len, value, start, count-end);
            count -= len;
        }
        return this;
    }
    
    public AbstractStringBuilder appendCodePoint(int codePoint) {...}
    
  }
```
