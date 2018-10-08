# Java String 知识梳理

## 为什么String是不可变对象？

1. 使用Final修饰String类：

```java

public final class String
{
    ...
}
```

---

1. 成员变量都是private修饰

```java
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0

/** use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = -6849794470754667710L;
```

---

## 不可变的优点

>- 字符串常量池的需要.字符串常量池可以将一些字符常量放在常量池中重复使用，避免每次都重新创建相同的对象、节省存储空间。但如果字符串是可变的，此时相同内容的String还指向常量池的同一个内存空间，当某个变量改变了该内存的值时，其他遍历的值也会发生改变。所以不符合常量池设计的初衷。
>- 线程安全考虑。同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。
>- 类加载器要用到字符串，不可变性提供了安全性，以便正确的类被加载。譬如你想加载java.sql.Connection类，而这个值被改成了myhacked.Connection，那么会对你的数据库造成不可知的破坏。
>- 支持hash映射和缓存。因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

---

## 真的是不可变类吗？

```java
@Test
    public void testChangeStr(){
        String s = "hello";
        System.out.println("s = " + s);
        try {
            Field field = String.class.getDeclaredField("value");
            field.setAccessible(true);
            char[] value = (char[])field.get(s);
            value[0] = 'a';
            System.out.println("s = " + s);
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
```

>输出：
</br>
s = hello
</br>
s = aello
</br>
总结：发现String的值已经发生了改变。也就是说，通过反射是可以修改所谓的“不可变”对象的

---

## 为什么StringBuffer是线程安全的？

```java
public synchronized XXX //同步关键字修饰方法
```

---

## 创建了几个对象

```java
String str1 = "abc";
String str2 = new String("abc");

```

>说明：
</br>
str1 = "abc" 如果字符串常量池中没有，则创建一个字符串在池中
</br>
str2 = new String("abc") 堆内存开辟一个字符串对象
，又因为"abc"字符串此时已经存在于常量池中，所以无需额外创建
---

## 字符串的比较

```java
    @Test
    public void testStringEquals() {
        String s1 = "hello";
        String s2 = "hello";
        System.out.println(s1 == s2); // true

        String s3 = "hello";
        String s4 = new String("hello");
        System.out.println(s3 == s4); // false

        String s5 = "hello";
        String s6 = "hell" + "o"; //编译器就能确定所以导致结果为true
        System.out.println(s5 == s6); // true

        String s7 = "hello1";
        int temp = 1;
        String s8 = "hello" + temp; //而s8含有变量导致不确定
        System.out.println(s7 == s8);// false

        String s9 = "hello1";
        final int temp2 = 1;
        String s10 = "hello" + temp2; //而s10含有final变量可以确定确定
        System.out.println(s9 == s10);// true
    }
```

---

## String的intern()方法

### JDK1.6 JDK1.7 JDK1.8 区别很大

### [intern详细介绍-1](超链接地址 "https://blog.csdn.net/baidu_31657889/article/details/52315902")

### [intern详细介绍-2](超链接地址 "https://blog.csdn.net/qq_36357995/article/details/79985538")

```java
String s1 = new String("1") + new String("1");
s1.intern();
String s2 = "11";
System.out.println(s1 == s2);
```

>说明：
</br>
jdk1.6运行结果是false，jdk1.7运行结果是true，jdk1.8运行结果false
</br>
</br>
***jdk6字符串常量池在方法区中和堆是独立的***
</br>
***jdk7字符串常量池在堆中***
</br>
***jdk8字符串常量池在metaspace中和堆是独立的***
</br>
</br>
第一行代码做的事情（只说对我们有影响的）：在栈中开辟空间存储s1（指向堆中引用地址）、在堆中开辟空间存储对象引用（"11"），接下来是jdk1.6和jdk1.7之间的区别：
jdk1.6中，s1.intern()运行时，首先去常量池查找，发现没有该常量，则在常量池中开辟空间存储"11"，返回堆空间地址（注意这里也没有使用该返回值），第三行中，s2则使栈中空间直接指向常量池，所以s1和s2不想等。
jdk1.7中，由于常量池在堆空间中，所以在s1.intern()运行时，发现常量池没有常量，则添加常量并使其指向堆空间地址，返回堆空间地址（注意这里也没有使用该返回值），这时s2通过查找常量池中的常量，同样也指向了堆空间地址，所以s1和s2相等。
>小结：
</br>
jdk7与jdk6相比，对String常量池的位置、String#intern()的语义都做了修改：
将String常量池从Perm区移到了Heap区。
调用String#intern()方法时，<font color="red">**堆中有该字符串而常量池中没有，则直接在常量池中保存堆中对象的引用，而不会在常量池中重新创建对象**</font>。
---

## String “+” 使用

```java
    @Test
    public void testStringIntern(){
        String b1 = "hello";
        String b2 = "world";
        String b3 = b1 + b2;
        System.out.println("helloworld" == b3); // false
    }
```

>说明：
</br>
因为 `b3` 指向堆中的`helloworld`对象，而`helloworld`是字符串池中的对象，所以结果为false。
</br>
JVM对 `String b1="hello"` 和 `String b2="world"` 对象放在常量池中是在编译时做的，而 `String b3 = b1+ b2` 是在运行时刻才能知道的。
`new` 对象也是在运行时才做的。而这段代码总共创建了5个对象，字符串池中两个、堆中三个。
</br>
`+` 运算符会在堆中建立来两个 `String` 对象，这两个对象的值分别是`hello`和`world`，也就是说从字符串池中复制这两个值，然后在堆中创建两个对象，然后再建立对象`b3`,然后将 `helloworld` 的堆地址赋给 `b3`
