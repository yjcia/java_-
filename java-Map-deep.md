# Java Map 深入

## 1. NavigableMap

>说明：
NavigableMap扩展了 SortedMap，具有了针对给定搜索目标返回最接近匹配项的导航方法。方法 lowerEntry、floorEntry、ceilingEntry 和 higherEntry 分别返回与小于、小于等于、大于等于、大于给定键的键关联的 Map.Entry 对象，如果不存在这样的键，则返回 null。类似地，方法 lowerKey、floorKey、ceilingKey 和 higherKey 只返回关联的键。所有这些方法是为查找条目而不是遍历条目而设计的。
</br>
`TreeMap implements NavigableMap<K,V>`
</br>
同理：`TreeSet implements NavigableSet<E>`
---

## 2. HashTable

>说明：
</br>
HashTable 和 HashMap的实现原理几乎一样，差别无非是：
</br>
（1）HashTable不允许key和value为null；
</br>
（2）HashTable是线程安全的，体现在`synchronized`关键字。
---

## 3. HashTable线程安全为什么性能差？

>说明：
</br>
HashTable 线程安全的策略实现代价却太大了，简单粗暴，get/put 所有相关操作都是 synchronized 的，这相当于给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。

```java
    //get
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }

    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```

<img src="https://github.com/frank-lam/2019_campus_apply/raw/master/notes/pics/hashtable-ds.png" width="600px">

---

## 4. 由Hashtable线程安全性能差所引出的ConcurrentHashMap

>说明：
</br>
ConcurrentHashMap <font color="orange">（JDK1.7）</font>采用了非常精妙的"分段锁"策略，ConcurrentHashMap 的主干是个 Segment 数组
</br>
<font color="orange">（JDK1.8）</font>使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized
<img src="https://github.com/frank-lam/2019_campus_apply/raw/master/notes/pics/hashmap-ds.png" width="600px">
</br>
Segment 继承自 ReentrantLock，所以我们可以很方便的对每一个 Segment 上锁。
对于读操作，获取 Key 所在的 Segment 时，需要保证可见性。具体实现上可以使用 volatile 关键字，也可使用锁。但使用锁开销太大，而使用 volatile 时每次写操作都会让所有 CPU 内缓存无效，也有一定开销。ConcurrentHashMap 使用如下方法保证可见性，取得最新的 Segment。

```java
    static class Segment<K,V> extends ReentrantLock implements Serializable {
        private static final long serialVersionUID = 2249069246763182397L;
        final float loadFactor;
        Segment(float lf) { this.loadFactor = lf; }
    }
```

对于写操作，并不要求同时获取所有 Segment 的锁，因为那样相当于锁住了整个 Map。它会先获取该 Key-Value 对所在的 Segment 的锁，获取成功后就可以像操作一个普通的 HashMap 一样操作该 Segment，并保证该Segment 的安全性。 <font color="orange">同时由于其它 Segment 的锁并未被获取，因此理论上可支持 concurrencyLevel（等于 Segment 的个数）个线程安全的并发读写。</font>
获取锁时，并不直接使用 lock 来获取，因为该方法获取锁失败时会挂起。事实上，它使用了<font color="orange">自旋锁</font>，如果 tryLock 获取锁失败，说明锁被其它线程占用，此时通过循环再次以 tryLock 的方式申请锁。如果在循环过程中该 Key 所对应的链表头被修改，则重置 retry 次数。如果 retry 次数超过一定值，则使用 lock 方法申请锁。
<font color="orange">这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗 CPU 资源比较多，因此在自旋次数超过阈值时切换为互斥锁。</font>