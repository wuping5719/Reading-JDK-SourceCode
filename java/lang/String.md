* 字符串String类的源码解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)
```java
public final class String 
      implements java.io.Serializable, Comparable<String>, CharSequence {
    //字符数组char[]形式存储字符串
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
    
    public String(char value[], int offset, int count) {...}
    
    public String(int[] codePoints, int offset, int count) {...}
    
    private static void checkBounds(byte[] bytes, int offset, int length) {...}
    
    public String(byte bytes[], int offset, int length, String charsetName) 
        throws UnsupportedEncodingException {...}
    
    public String(byte bytes[], int offset, int length, Charset charset) {...}
    
    public String(byte bytes[], String charsetName) throws UnsupportedEncodingException {...}
    
    public String(byte bytes[], Charset charset) {...}
    
    public String(byte bytes[], int offset, int length) {...}
    
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
    
    String(char[] value, boolean share) {...}
    
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
    
    public int codePointAt(int index) {...}
    
    public int codePointBefore(int index) {...}
    
    public int codePointCount(int beginIndex, int endIndex) {...}
    
    public int offsetByCodePoints(int index, int codePointOffset) {...}
    
    void getChars(char dst[], int dstBegin) {...}
    
    public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {...}
    
}
