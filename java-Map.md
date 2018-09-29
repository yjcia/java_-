# Java Map 知识梳理
>说明：
</br>
- Map 是一个接口
- Map 无序，key唯一
- HashMap 无序，key唯一，key、value都可存放null
- LinkedHashMap 有序，key唯一
- TreeMap 有序 ，key的自然排序或者根据compareTo结果
- ConcurrentHashMap 重点
  
>是否以上这些Map数据结构中的`key,value`都可以为`null`？
```java
    @Test
    public void testMap01(){
        Map demoMap = new HashMap();
        demoMap.put(null,null);//junit pass
    }

    @Test
    public void testMap02(){
        Map demoMap = new LinkedHashMap();
        demoMap.put(null,null);//junit pass
    }

    @Test
    public void testMap03(){
        Map demoMap = new TreeMap();
        demoMap.put(null,null);//junit fail,NullPointerException
    }

    @Test
    public void testMap04(){
        Map demoMap = new ConcurrentHashMap();
        demoMap.put(null,null);//junit fail,NullPointerException
    }
```
## HashMap底层存储结构以及Hash冲突

>**JDK 1.7** - 链地址法
</br>
<img src="https://github.com/frank-lam/2019_campus_apply/raw/master/notes/pics/hashmap-link.jpg" width="500px"/>
```java
    //每个Entry实现Map.Entry
    //每个Entry包含下一个节点的引用，Entry<K,V> next
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
```
>**JDK 1.7** 如何解决Hash冲突？
</br>
当我们往hashmap中put元素的时候，先根据key的hash值得到这个元素在数组中的位置（即下标），然后就可以把这个元素放到对应的位置中了。 如果这个元素所在的位子上已经存放有其他元素了，那么在同一个位子上的元素将以链表的形式存放，<font color="orange">新加入的放在链头，最先加入的放在链尾。从hashmap 中get元素时，首先计算key的hashcode，找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。从这里我们可以想象得到</font>，当然 Hash 算法计算结果越分散均匀，Hash 碰撞的概率就越小，map 的存取效率就会越高。

>**JDK 1.8** - 链地址法 + 红黑树
</br>
<img src="https://github.com/frank-lam/2019_campus_apply/raw/master/notes/pics/hashmap-rb-link.jpg" width="500px"/>
```java
    //每个Node实现Map.Entry
    //每个Node包含下一个节点的引用，Node<K,V> next
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;//用来定位索引数组的位置
        final K key;
        V value;
        Node<K,V> next;
```
>**JDK 1.8** 如何解决Hash冲突？
</br>
在 JDK1.8 版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高 HashMap 的性能，其中会用到红黑树的插入、删除、查找等算法。
## HashMap Put (JDK 8)
<img src="https://github.com/frank-lam/2019_campus_apply/raw/master/notes/pics/hashmap-put.png" width="600px">

## HashMap 扩容机制（JDK 8）
>说明：
- 在 HashMap 中，<font color="orange">哈希桶数组 table 的长度 length 大小必须为 2的n次方（一定是合数）</font>，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。<font color="orange">相对来说素数导致冲突的概率要小于合数.</font>
- Node[] table的初始化长度 length (默认值是16)，Load factor 为负载因子(默认值是0.75)
- 重要参数
  
参数|说明|
|:--|:--|
|buckets|在 HashMap 的注释里使用哈希桶来形象的表示数组中每个地址位置。注意这里并不是数组本身，数组是装哈希桶的，他可以被称为哈希表。|
|capacity|table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。|
|size|table 的实际使用量。|
|threshold|size 的临界值，size 必须小于 threshold，如果大于等于，就必须进行扩容操作。|
|loadFactor|装载因子，table 能够使用的比例，threshold = capacity * loadFactor。|
|TREEIFY_THRESHOLD|树化阀值，哈希桶中的节点个数大于该值（默认为8）的时候将会被转为红黑树行存储结构。
|UNTREEIFY_THRESHOLD|非树化阀值，小于该值（默认为 6）的时候将再次改为单链表的格式存储

```java
    void resize(int newCapacity) {   //传入新的容量
        Entry[] oldTable = table;    //引用扩容前的Entry数组
        int oldCapacity = oldTable.length;         
        if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
            threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
            return;
        }
    
        Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
        transfer(newTable);                         //！！将数据转移到新的Entry数组里
        table = newTable;                           //HashMap的table属性引用新的Entry数组
        threshold = (int)(newCapacity * loadFactor);//修改阈值
    }
```
```java
    void transfer(Entry[] newTable) {
        Entry[] src = table;                   //src引用了旧的Entry数组
        int newCapacity = newTable.length;
        for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
            Entry<K,V> e = src[j];             //取得旧Entry数组的每个元素
            if (e != null) {
                src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
                do {
                    Entry<K,V> next = e.next;
                    int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
                    e.next = newTable[i]; //标记[1]
                    newTable[i] = e;      //将元素放在数组上
                    e = next;             //访问下一个Entry链上的元素
                } while (e != null);
            }
        }
    }
```