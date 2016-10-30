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
    
    //删除指定索引位置的字符
    public AbstractStringBuilder deleteCharAt(int index) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        System.arraycopy(value, index+1, value, index, count-index-1);
        count--;
        return this;
    }
    
    //用参数字符串str替换从start开始到end结束的字符序列
    public AbstractStringBuilder replace(int start, int end, String str) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (start > count)
            throw new StringIndexOutOfBoundsException("start > length()");
        if (start > end)
            throw new StringIndexOutOfBoundsException("start > end");

        if (end > count)
            end = count;
        int len = str.length();
        int newCount = count + len - (end - start);
        ensureCapacityInternal(newCount);

        System.arraycopy(value, end, value, start + len, count - end);
        str.getChars(value, start);
        count = newCount;
        return this;
    }
    
    //从start位置开始返回子串
    public String substring(int start) {
        return substring(start, count);
    }
    public CharSequence subSequence(int start, int end) {...}
    public String substring(int start, int end) {...}
    
    //把字符数组插入到序列中的对应位置
    public AbstractStringBuilder insert(int index, char[] str, int offset, int len) {
        if ((index < 0) || (index > length()))
            throw new StringIndexOutOfBoundsException(index);
        if ((offset < 0) || (len < 0) || (offset > str.length - len))
            throw new StringIndexOutOfBoundsException(
                "offset " + offset + ", len " + len + ", str.length "
                + str.length);
        ensureCapacityInternal(count + len);
        System.arraycopy(value, index, value, index + len, count - index);
        System.arraycopy(str, offset, value, index, len);
        count += len;
        return this;
    }
    public AbstractStringBuilder insert(int offset, Object obj) {...}
    public AbstractStringBuilder insert(int offset, String str) {...}
    public AbstractStringBuilder insert(int offset, char[] str) {...}
    public AbstractStringBuilder insert(int dstOffset, CharSequence s) {...}
    public AbstractStringBuilder insert(int dstOffset, CharSequence s, int start, int end) {...}
    
    public AbstractStringBuilder insert(int offset, boolean b) {
        return insert(offset, String.valueOf(b));
    }
    public AbstractStringBuilder insert(int offset, char c) {...}
    public AbstractStringBuilder insert(int offset, int i) {...}
    public AbstractStringBuilder insert(int offset, long l) {...}
    public AbstractStringBuilder insert(int offset, float f) {...}
    public AbstractStringBuilder insert(int offset, double d) {...}
    
    //返回字符串在序列中第一次出现的位置
    public int indexOf(String str) {...}
    public int indexOf(String str, int fromIndex) {...}
    
    //返回字符串在序列中最右边出现的位置
    public int lastIndexOf(String str) {...}
    public int lastIndexOf(String str, int fromIndex) {...}
    
    //翻转字符序列
    public AbstractStringBuilder reverse() {
        boolean hasSurrogate = false;
        int n = count - 1;
        for (int j = (n-1) >> 1; j >= 0; --j) {
            char temp = value[j];
            char temp2 = value[n - j];
            if (!hasSurrogate) {
                hasSurrogate = (temp >= Character.MIN_SURROGATE && temp <= Character.MAX_SURROGATE)
                    || (temp2 >= Character.MIN_SURROGATE && temp2 <= Character.MAX_SURROGATE);
            }
            value[j] = temp2;
            value[n - j] = temp;
        }
        if (hasSurrogate) {
            // 反向所有有效的代理对
            for (int i = 0; i < count - 1; i++) {
                char c2 = value[i];
                if (Character.isLowSurrogate(c2)) {
                    char c1 = value[i + 1];
                    if (Character.isHighSurrogate(c1)) {
                        value[i++] = c1;
                        value[i] = c2;
                    }
                }
            }
        }
        return this;
    }
    
    public abstract String toString();
    
    //contentEquals方法需要该方法
    final char[] getValue() {
        return value;
    }
  }
```
