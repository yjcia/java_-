# Java Set 知识梳理
>说明:
- Set 是一个接口
- AbstractSet 抽象类实现 Set 接口，继承AbstractCollection
- HashSet 无序不重复，继承AbstractSet，实现Set接口,***可以存放null*** 
- TreeSet 有序（自然排序 + Comparable接口）不重复，继承AbstractSet，实现NavigableSet接口
- LinkedHashSet 有序（保证插入顺序）不重复，继承HashSet，实现Set接口
--- 
>### Set 如何保证不重复？
>hashcode & equals 双重验证：
</br>
HashSet:
</br>
HashSet在执行add方法时，其实是将数据作为key存入，而value只保存一个PRESENT对象,又因为Map的key是不能重复的。
```java
    //HashSet add
    //static final PRESENT Object
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    //Map put
    //先比较hashcode如果hashcode一致在比较equals
    e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))
    
    //(1)当obj1.equals(obj2)为true时，obj1.hashCode() == obj2.hashCode()必须为true  
    //(2)当obj1.hashCode() == obj2.hashCode()为false时，obj1.equals(obj2)必须为false
```
## LinkedHashSet
>说明：
</br>
HashMap --> HashSet
LinkedHashMap --> LinkedHashSet
</br>
LinkedHashMap 的结构图，
主体部分跟 HashMap 完全一样，多了 header 指向双向链表的头部（是一个哑元），该双向链表的迭代顺序就是 entry 的插入顺序
</br>
</br>
<!-- ![LinkedHashMap](https://github.com/frank-lam/2019_campus_apply/raw/master/notes/pics/LinkedHashMap_base.png) -->

## TreeSet
>说明：
</br>
1.TreeSet --> TreeMap
</br>
默认排序自然排序方式：
```java
    //TreeMap put方法
    cmp = k.compareTo(t.key);
    if (cmp < 0)
        t = t.left;
    else if (cmp > 0)
        t = t.right;
    else
        return t.setValue(value);
```
>2.类型要一致
```java
    @Test
    public void testSet02(){
        Set set2 = new TreeSet();
        set2.add("b");
        set2.add("a");
        set2.add(1);
        System.out.println(set2);//java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Integer
    }
```
>3.Comparable接口
```java
    @Test
    public void testSet03() {
        Person p1 = new Person(20,"yanjun");
        Person p2 = new Person(19,"zhangsan");
        Set personSet = new TreeSet();
        personSet.add(p1);
        personSet.add(p2);
        System.out.println(personSet);
        //[Person{age=19, name='zhangsan'}, Person{age=20, name='yanjun'}]
    }

    //实现Comparable接口
    class Person implements Comparable {
        private int age;
        private String name;

        public Person(int age, String name) {
            this.age = age;
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "age=" + age +
                    ", name='" + name + '\'' +
                    '}';
        }

        //重写hashCode方法    
        @Override
        public int hashCode() {
            return name.hashCode() + age;
        }

        //重写equals方法 
        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof Person)) {
                return false;
            }
            Person p = (Person) obj;
            return this.name.equals(p.name) && this.age == p.age;

        }
        //重写compareTo方法
        //如果年龄小的会排在前面，如果年龄一致再看名称，按照名称自然排序
        @Override
        public int compareTo(Object obj) {
            Person p = (Person) obj;
            if (this.age > p.age) {
                return 1;
            }
            if (this.age < p.age) {
                return -1;
            }
            return this.name.compareTo(p.name);

        }
    }
```
