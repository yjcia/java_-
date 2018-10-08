# Java Integer 知识梳理

## Integer '==' 比较

```java
    @Test
    public void testInteger(){
        Integer a = 100;
        Integer b = 100;
        System.out.println(a == b); // true
    }

```

>说明：
在上限和下限范围之内声明`Integer`的时候直接总`cache`中获取,
反之`new Integer`获取
</br>
Integer 缓存下限：`static final int low = -128;`
</br>
Integer 缓存上限：
</br>
默认：`h=127`
</br>
参数设置参考下方代码

```java
String integerCacheHighPropValue =
    sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
if (integerCacheHighPropValue != null) {
    try {
        int i = parseInt(integerCacheHighPropValue);
        i = Math.max(i, 127);
        // Maximum array size is Integer.MAX_VALUE
        h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
    } catch( NumberFormatException nfe) {
        // If the property cannot be parsed into an int, ignore it.
    }
}
```

```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

## Integer 和 int 比较

```java
    @Test
    public void testIntegerInt(){
        Integer a = 100;
        int b = 100;
        System.out.println(a == b); // true
    }
    //一个Integer 与 int比较，先将Integer转换成int类型，再做值比较
```

```java
    @Test
    public void testIntegerInt(){
        Integer a = new Integer(100);
        int b = 100;
        System.out.println(a == b); // true

    }
    //原因同上
```

```java
    @Test
    public void testIntegerInt(){
        Integer a = new Integer(100);
        Integer b = Integer.valueOf(100);
        System.out.println(a == b); // false

    }
    //a -> 堆中的地址
    //b -> cache缓存地址
```