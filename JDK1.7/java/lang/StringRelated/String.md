* 字符串String类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  String类为什么是final?   
  主要是为了“效率”和“安全”. 若String允许被继承, 由于它的高度被使用率, 可能会降低程序的性能，所以String被定义成final.     
  String，StringBuffer，StringBuilder三者在执行速度方面的比较：StringBuilder > StringBuffer > String.    
  对于三者使用的总结：   
  &nbsp;&nbsp;&nbsp; (1) 如果要操作少量的数据用String；   
  &nbsp;&nbsp;&nbsp; (2) 单线程下操作字符串缓冲区，大量数据用StringBuilder；    
  &nbsp;&nbsp;&nbsp; (3) 多线程下操作字符串缓冲区，大量数据用StringBuffer。
* 三者的类结构图解：
  <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_2.png" /></p>
  
```java
public final class String 
      implements java.io.Serializable, Comparable<String>, CharSequence {
    //字符数组char[]形式存储字符串, 用final关键字修饰
    private final char value[];   
    
    //hash值, 默认为0
    private int hash; 
    
    private static final long serialVersionUID = -6849794470754667710L;
    private static final ObjectStreamField[] serialPersistentFields =
            new ObjectStreamField[0];
    
    //无参构造函数        
    public String() {
        this.value = new char[0];
    }
    
    //参数为字符串的构造函数        
    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
    
    //参数为字符数组char[]的构造函数      
    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
    
    //去掉字符串前导空格或尾部空格
    public String trim() {...}
    
    public String toString() {...}
    
    //将String转换为字符数组char[]
    public char[] toCharArray() {
        // 因为类的初始化顺序问题不能使用arrays.copyof
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }
    
    //将字节数组转换为字符串
    public String(byte bytes[]) {
        this(bytes, 0, bytes.length);
    }
    
    //将StringBuffer转换字符串
    public String(StringBuffer buffer) {
        synchronized(buffer) {   //因为StringBuffer是线程安全的, 所以此处用到synchronized关键字
            this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
        }
    }
    
    //将StringBuilder转换字符串, StringBuilder是线程不安全的, 不需要同步
    public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }
        
    //获取字符串的长度
    public int length() {
        return value.length;
    }
    
    //判断字符串是否为空
    public boolean isEmpty() {
        return value.length == 0;
    }
    
    //获取字符串指定位置的字符
    public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
   
    //判断字符串是否由相同的字符序列组成
    public boolean equals(Object anObject) {
        if (this == anObject) {   
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                            return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
   
    public boolean contentEquals(StringBuffer sb) {
        synchronized (sb) {
            return contentEquals((CharSequence) sb);
        }
    }
    
    //比较两个字符串大小
    public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
    
    //将str添加到原串的末尾
    public String concat(String str) {...}
    
    //将字符串中的旧字符替换为新字符
    public String replace(char oldChar, char newChar) {...}
    
    //求索引beginIndex为起始的子串
    public String substring(int beginIndex) {...}
    
    //将字符串按分隔符拆分为字符数组
    public String[] split(String regex, int limit) {
        /* fastpath if the regex is a
         (1)one-char String and this character is not one of the
            RegEx's meta characters ".$|()[{^?*+\\", or
         (2)two-char String and the first char is the backslash and
            the second is not the ascii digit or ascii letter.
         */
        char ch = 0;
        if (((regex.value.length == 1 &&
             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
             (regex.length() == 2 &&
              regex.charAt(0) == '\\' &&
              (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
              ((ch-'a')|('z'-ch)) < 0 &&
              ((ch-'A')|('Z'-ch)) < 0)) &&
            (ch < Character.MIN_HIGH_SURROGATE ||
             ch > Character.MAX_LOW_SURROGATE))
        {
            int off = 0;
            int next = 0;
            boolean limited = limit > 0;
            ArrayList<String> list = new ArrayList<>();
            while ((next = indexOf(ch, off)) != -1) {
                if (!limited || list.size() < limit - 1) {
                    list.add(substring(off, next));
                    off = next + 1;
                } else {    // 最后一个
                    //assert (list.size() == limit - 1);
                    list.add(substring(off, value.length));
                    off = value.length;
                    break;
                }
            }
            // 如果没有找到匹配，返回
            if (off == 0)
                return new String[]{this};

            // 加入剩余的部分
            if (!limited || list.size() < limit)
                list.add(substring(off, value.length));

            // 构建结果
            int resultSize = list.size();
            if (limit == 0)
                while (resultSize > 0 && list.get(resultSize - 1).length() == 0)
                    resultSize--;
            String[] result = new String[resultSize];
            return list.subList(0, resultSize).toArray(result);
        }
        return Pattern.compile(regex).split(this, limit);
    }
    
    String(char[] value, boolean share) {...}
    public String(char value[], int offset, int count) {...}
    public String(int[] codePoints, int offset, int count) {...}
    public String(byte bytes[], int offset, int length, String charsetName) 
        throws UnsupportedEncodingException {...}
    
    private static void checkBounds(byte[] bytes, int offset, int length) {...}
   
    public boolean contentEquals(CharSequence cs) {...}
    
    public boolean equalsIgnoreCase(String anotherString) {...}
    
    public int codePointAt(int index) {...}
    public int codePointBefore(int index) {...}
    public int codePointCount(int beginIndex, int endIndex) {...}
    public int offsetByCodePoints(int index, int codePointOffset) {...}
    
    void getChars(char dst[], int dstBegin) {...}
    public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {...}
    public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {...}
    
    public byte[] getBytes(String charsetName) throws UnsupportedEncodingException {...}
    public byte[] getBytes(Charset charset) {...}
    
    public byte[] getBytes() {...}
    public String(byte bytes[], int offset, int length, Charset charset) {...}
    public String(byte bytes[], String charsetName) throws UnsupportedEncodingException {...}
    public String(byte bytes[], Charset charset) {...}
    public String(byte bytes[], int offset, int length) {...}
    
    public boolean regionMatches(int toffset, String other, int ooffset, int len){...};
    public boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len) {...}
    
    public boolean startsWith(String prefix, int toffset) {...}
    public boolean startsWith(String prefix) {...}
    
    public boolean endsWith(String suffix) {
        return startsWith(suffix, value.length - suffix.value.length);
    }
    
    public int hashCode() {...}
    
    public int indexOf(int ch) {...}
    public int indexOf(int ch, int fromIndex) {...}
    private int indexOfSupplementary(int ch, int fromIndex) {...}
    public int lastIndexOf(int ch) {...}
        
    public String substring(int beginIndex, int endIndex) {...}
    public CharSequence subSequence(int beginIndex, int endIndex) {...}
        
    public boolean matches(String regex) {...}
    
    public boolean contains(CharSequence s) {...}
    
    public String replaceFirst(String regex, String replacement) {...}
    public String replaceAll(String regex, String replacement) {...}
    public String replace(CharSequence target, CharSequence replacement) {...}
    
    public String[] split(String regex) {...}
    
    //将字符串中的大写字符全部转换为小写
    public String toLowerCase(Locale locale) {
        if (locale == null) {
            throw new NullPointerException();
        }

        int firstUpper;
        final int len = value.length;

        // 现在检查是否有需要改变的字符
        scan: {
            for (firstUpper = 0 ; firstUpper < len; ) {
                char c = value[firstUpper];
                if ((c >= Character.MIN_HIGH_SURROGATE)
                        && (c <= Character.MAX_HIGH_SURROGATE)) {
                    int supplChar = codePointAt(firstUpper);
                    if (supplChar != Character.toLowerCase(supplChar)) {
                        break scan;
                    }
                    firstUpper += Character.charCount(supplChar);
                } else {
                    if (c != Character.toLowerCase(c)) {
                        break scan;
                    }
                    firstUpper++;
                }
            }
            return this;
        }

        char[] result = new char[len];
        int resultOffset = 0;  //结果可能会增长，所以i+resultoffset在结果中的处理位置

        // 只复制前几个小写字符
        System.arraycopy(value, 0, result, 0, firstUpper);

        String lang = locale.getLanguage();
        boolean localeDependent =
                (lang == "tr" || lang == "az" || lang == "lt");
        char[] lowerCharArray;
        int lowerChar;
        int srcChar;
        int srcCount;
        for (int i = firstUpper; i < len; i += srcCount) {
            srcChar = (int)value[i];
            if ((char)srcChar >= Character.MIN_HIGH_SURROGATE
                    && (char)srcChar <= Character.MAX_HIGH_SURROGATE) {
                srcChar = codePointAt(i);
                srcCount = Character.charCount(srcChar);
            } else {
                srcCount = 1;
            }
            if (localeDependent || srcChar == '\u03A3') { // 希腊大写字母
                lowerChar = ConditionalSpecialCasing.toLowerCaseEx(this, i, locale);
            } else if (srcChar == '\u0130') { // 拉丁文大写字母I点
                lowerChar = Character.ERROR;
            } else {
                lowerChar = Character.toLowerCase(srcChar);
            }
            if ((lowerChar == Character.ERROR)
                    || (lowerChar >= Character.MIN_SUPPLEMENTARY_CODE_POINT)) {
                if (lowerChar == Character.ERROR) {
                    if (!localeDependent && srcChar == '\u0130') {
                        lowerCharArray =
                                ConditionalSpecialCasing.toLowerCaseCharArray(this, i, Locale.ENGLISH);
                    } else {
                        lowerCharArray =
                                ConditionalSpecialCasing.toLowerCaseCharArray(this, i, locale);
                    }
                } else if (srcCount == 2) {
                    resultOffset += Character.toChars(lowerChar, result, i + resultOffset) - srcCount;
                    continue;
                } else {
                    lowerCharArray = Character.toChars(lowerChar);
                }

                // 如果需要的话，结果长度增长
                int mapLen = lowerCharArray.length;
                if (mapLen > srcCount) {
                    char[] result2 = new char[result.length + mapLen - srcCount];
                    System.arraycopy(result, 0, result2, 0, i + resultOffset);
                    result = result2;
                }
                for (int x = 0; x < mapLen; ++x) {
                    result[i + resultOffset + x] = lowerCharArray[x];
                }
                resultOffset += (mapLen - srcCount);
            } else {
                result[i + resultOffset] = (char)lowerChar;
            }
        }
        return new String(result, 0, len + resultOffset);
    }
    public String toLowerCase() {...}
    
    public String toUpperCase(Locale locale) {...}
    public String toUpperCase() {...}
    
    public static String format(String format, Object... args) {...}
    
    public static String valueOf(Object obj) {...}
    public static String valueOf(char data[]) {...}
    public static String valueOf(char data[], int offset, int count) {...}
    
    public static String copyValueOf(char data[], int offset, int count) {...}
    public static String copyValueOf(char data[]) {...}
    
}
