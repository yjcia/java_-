# Java List 知识梳理
>说明:
- collection 是一个interface
- Set接口继承Collection接口
- List接口继承Collection接口
- Queue接口继承Collection接口
- Collection接口继承Iterable接口
- Collection是所有集合类的根接口
- Collections是提供集合操作的工具类
  
## List
>说明:
</br>
1. 有序 ：`An ordered collection`
</br>
2. 允许重复 ：`lists typically allow duplicate elements`
</br>
3. 允许null值插入，示例代码如下：
```java
    @Test
    public void testList(){
        List list = new ArrayList();
        list.add(1);
        list.add(null);
        System.out.println(list.size()); //2
    }
```
4. ArrayList线程不安全，LinkedList线程不安全，
   </br>
   vector线程安全方法签名包含synchronized关键字
   </br>
   ***ArrayList,LinkedList如何转为线程安全？***
   </br>
   ```java
   List<Map<String,Object>> dataList 
        = Collections.synchronizedList(new ArrayList<Map<String,Object>>());
   ```
   


## 简单的速度测试：
>后方插入
```java
    @Test
    public void testList2() {
        List list = new ArrayList();
        Long startTime = System.currentTimeMillis();
        for (int i = 0; i < xxx ; i++) {
            //System.out.println(i);
            list.add(1);
        }
        Long endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime + "ms");
    }

    @Test
    public void testList2() {
        List list = new LinkedList();
        Long startTime = System.currentTimeMillis();
        for (int i = 0; i < xxx ; i++) {
            //System.out.println(i);
            list.add(1);
        }
        Long endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime + "ms");
    }

    // 三代i7 8核 4G内存 
    // ArrayList
    // 1百万 14ms
    // 5百万 68ms
    // 1千万 130ms
    // 5千万 456ms
    // 1亿 749ms
    // 2亿 12646ms
    // 3亿 41561ms
    // 4亿 java heap space
    // --------------------------
    // LinkedList
    // 1百万 17ms
    // 5百万 1682ms
    // 1千万 3789ms
    // 5千万 23308ms
    // 1亿 54245ms
    // 2亿 很慢很慢
```
>前方插入
```java
    @Test
    public void testList2() {
        List list = new LinkedList();
        Long startTime = System.currentTimeMillis();
        for (int i = 0; i < xxx; i++) {
            //System.out.println(i);
            list.add(0,i);
        }
        Long endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime + "ms");
    }

    @Test
    public void testList3() {
        List list = new ArrayList();
        Long startTime = System.currentTimeMillis();
        for (int i = 0; i < xxx; i++) {
            //System.out.println(i);
            list.add(0,i);
        }
        Long endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime + "ms");
    }
    // 三代i7 8核 4G内存 
    // ArrayList
    // 1百万 99539ms
    // 剩下不用比了
    // --------------------------
    // LinkedList
    // 1百万 126ms
    // 5百万 4114ms
    // 1千万 9033ms
```
>总结：后插ArrayList快，前插LinkedList快，因为只要断链不需要移动整体。

>读取速度
```java
    @Test
    public void testList2() {
        List list = new LinkedList();

        for (int i = 0; i < 1000000; i++) {
            //System.out.println(i);
            list.add(i);
        }
        Long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            //System.out.println(i);
            list.get(i);
        }
        Long endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime + "ms");
    }

    @Test
    public void testList3() {
        List list = new ArrayList();

        for (int i = 0; i < 1000000; i++) {
            //System.out.println(i);
            list.add(i);
        }
        Long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            //System.out.println(i);
            list.get(i);
        }

        Long endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime + "ms");
    }
    // 三代i7 8核 4G内存 
    // ArrayList
    // 1百万 5ms
    // 5百万 4ms
    // 1千万 4ms
    // --------------------------
    // LinkedList
    // 10000 170ms
    // 剩下不用比了
```
>总结：ArrayList底层基于数组，数组有下标查找快，LinkedList底层基于链表，查找较慢
