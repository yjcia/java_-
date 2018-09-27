# Java List 深入
### RandomAccess 接口
```java
/**
* 
Marker interface used by <tt>List</tt>    implementations to indicate that
they support fast (generally constant time) random access.
 ...
*/
public interface RandomAccess {
}

int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```
```java
//ArrayList 实现RandomAccess接口
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```  
```java
//LinkedList 没有实现RandomAccess接口
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```
### RandomAccess接口作用
>说明
</br>
标志接口,只要List集合实现这个接口，就能支持快速随机访问
使用</br>`instanceof`来判断
`list instanceof RandomAccess`

### 算法练习题：二分法查找
```java
    @Test
    public void testBinarySearch() {
        List<Integer> dataList = new ArrayList<>();
        for (int i = 0; i < 11; i++) {
            dataList.add(i * 2 + 1);
        }
        System.out.println(dataList);
        int key = 21;
        int index = doBinarySearch(dataList,key);
        System.out.println("value : " + key + "  index : " + index);
    }

    //dataList 升序
    private int doBinarySearch(List<Integer> dataList,int value){
        int low = 0;
        int high = dataList.size() - 1;
        int mid = (low + high) / 2;
        while(low <= high){
            int midValue = dataList.get(mid);
            if(value == midValue){
                return mid;
            }else if(value < midValue){
                high = mid - 1;
            }else if(value > midValue){
                low = mid + 1;
            }
            mid = (low + high) / 2;
        }
        return -1;
    }
```
### List 扩容
`DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}`
</br>
`DEFAULT_CAPACITY = 10`
</br>
添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1)，也就是旧容量的 1.5 倍。

扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。
```java
    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); // 1.5倍扩容
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
```java
    //最大到Integer.MAX_VALUE 21亿
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
### CopyOnWriteArrayList
`List<String> list = new CopyOnWriteArrayList<>();`
>说明
</br>
CopyOnWrite 容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对 CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以 CopyOnWrite 容器也是一种读写分离的思想，读和写不同的容器。

>缺点
</br>
**内存占用问题**
</br>
因为 CopyOnWrite 的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意：在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说 200M 左右，那么再写入 100M 数据进去，内存就会占用 300M，那么这个时候很有可能造成频繁的 Yong GC 和 Full GC。之前我们系统中使用了一个服务由于每晚使用 CopyOnWrite 机制更新大对象，造成了每晚 15 秒的 Full GC，应用响应时间也随之变长。
</br>
针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是 10 进制的数字，可以考虑把它压缩成 36 进制或 64 进制。或者不使用 CopyOnWrite 容器，而使用其他的并发容器，如 ConcurrentHashMap。
</br>
</br>
**数据一致性问题**
</br>
CopyOnWrite 容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用 CopyOnWrite 容器。
