# Java ThreadLocal

## 什么是ThreadLocal？

>说明：
</br>
首先，ThreadLocal 不是用来解决共享对象的多线程访问问题的，一般情况下，通过ThreadLocal.set() 到线程中的对象是该线程自己使用的对象，其他线程是不需要访问的，也访问不到的。各个线程中访问的是不同的对象。 
另外，说ThreadLocal使得各线程能够保持各自独立的一个对象，并不是通过ThreadLocal.set()来实现的，而是通过每个线程中的new 对象的操作来创建的对象，每个线程创建一个，不是什么对象的拷贝或副本。

```java
    public class ThreadLocalTest {
    private static Logger logger = LoggerFactory.getLogger(ThreadLocalTest.class);
    private static final ThreadLocal<Object> threadLocal = new ThreadLocal<Object>() {
        @Override
        protected Object initialValue() {
            logger.debug("invoke initialValue");
            return super.initialValue();
        }
    };

    public static void main(String args[]) {
        new Thread(new MyTask("aa")).start();
        new Thread(new MyTask("bb")).start();
        new Thread(new MyTask("cc")).start();
    }

    public static class MyTask implements Runnable {

        private String name;

        public MyTask(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                if (null == ThreadLocalTest.threadLocal.get()) {
                    ThreadLocalTest.threadLocal.set(name + ":" + 0);
                    logger.debug("ThreadLocalTest.threadLocal.set : " + name + ":" + 0);
                } else {
                    String value = (String) ThreadLocalTest.threadLocal.get();
                    int currentIndex = Integer.parseInt(value.split(":")[1]);
                    currentIndex++;
                    ThreadLocalTest.threadLocal.set(name + ":" + currentIndex);
                    logger.debug("Else -- ThreadLocalTest.threadLocal.set : " + name + ":" + currentIndex);
                    if (i == 3) {
                        ThreadLocalTest.threadLocal.remove();
                    }
                }
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

>结果：
</br>

```java
DEBUG 2018-10-08 12:58:43730 invoke initialValue
DEBUG 2018-10-08 12:58:43730 invoke initialValue
DEBUG 2018-10-08 12:58:43731 ThreadLocalTest.threadLocal.set : bb:0
DEBUG 2018-10-08 12:58:43730 invoke initialValue
DEBUG 2018-10-08 12:58:43731 ThreadLocalTest.threadLocal.set : cc:0
DEBUG 2018-10-08 12:58:43731 ThreadLocalTest.threadLocal.set : aa:0
DEBUG 2018-10-08 12:58:45731 Else -- ThreadLocalTest.threadLocal.set : bb:1
DEBUG 2018-10-08 12:58:45731 Else -- ThreadLocalTest.threadLocal.set : aa:1
DEBUG 2018-10-08 12:58:45731 Else -- ThreadLocalTest.threadLocal.set : cc:1
DEBUG 2018-10-08 12:58:47731 Else -- ThreadLocalTest.threadLocal.set : bb:2
DEBUG 2018-10-08 12:58:47731 Else -- ThreadLocalTest.threadLocal.set : cc:2
DEBUG 2018-10-08 12:58:47731 Else -- ThreadLocalTest.threadLocal.set : aa:2
DEBUG 2018-10-08 12:58:49731 Else -- ThreadLocalTest.threadLocal.set : bb:3
DEBUG 2018-10-08 12:58:49731 Else -- ThreadLocalTest.threadLocal.set : cc:3
DEBUG 2018-10-08 12:58:49731 Else -- ThreadLocalTest.threadLocal.set : aa:3
DEBUG 2018-10-08 12:58:51731 invoke initialValue
DEBUG 2018-10-08 12:58:51731 ThreadLocalTest.threadLocal.set : bb:0
DEBUG 2018-10-08 12:58:51731 invoke initialValue
DEBUG 2018-10-08 12:58:51731 ThreadLocalTest.threadLocal.set : aa:0
DEBUG 2018-10-08 12:58:51731 invoke initialValue
DEBUG 2018-10-08 12:58:51731 ThreadLocalTest.threadLocal.set : cc:0
DEBUG 2018-10-08 12:58:53732 Else -- ThreadLocalTest.threadLocal.set : aa:1
DEBUG 2018-10-08 12:58:53732 Else -- ThreadLocalTest.threadLocal.set : bb:1
DEBUG 2018-10-08 12:58:53732 Else -- ThreadLocalTest.threadLocal.set : cc:1
```

## 源码分析

---

### Method Get

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

>说明：

- `Thread t = Thread.currentThread` 获得当前线程
- `ThreadLocalMap map = getMap(t)` 根据当前线程获得该线程的对应的ThreadLocalMap
    ```java
        //注意参数是Thread t
        ThreadLocalMap getMap(Thread t) {
            return t.threadLocals;
        }
    ```
- 如果map不为null：
  </br>
  `ThreadLocalMap.Entry e = map.getEntry(this)`，注意这里参数变成了this,
  根据之前的示例代码this指的是当前的ThreadLocal（ThreadLocalTest$1@810）
- `T result = (T)e.value;` 获得实际的数据
- 如果map为null：
  </br>
  执行`setInitialValue()`

  ```java
  private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);//重点关注
        return value;
    }
  ```

  ```java

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
        //每个线程new一个ThreadLocalMap，其中key就是当前ThreadLocal,t则是当前线程，所以每个线程都会保存该只能由线程能访问到的数据副本。
    }
  ```

---

### Method Set

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

>说明：

- `Thread t = Thread.currentThread()` 获得当前线程
- `ThreadLocalMap map = getMap(t);` 根据当前线程获得当前线程对应的ThreadLocalMap
- `map.set(this, value);` 存入数据，key为this

---

### ThreadLocalMap

```java
    static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        //为什么要用弱引用？
        /**
        * key为null，由于是weak reference 所以依旧可以被GC
        我们假设如果ThreadLocalMap中是强引用会出现什么情况？定义ThreadLocal时定义的强引用被置为null的时候，如果还有其它使用了该ThreadLocal的线程没有完成，还需要很久会执行完成，那么这个线程将一直持有该ThreadLocal实例的引用，直到线程完成，期间ThreadLocal实例都不能被回收，最重要的是如果不了解ThreadLocal内部实现，你可能都不知道还有其他线程引用了threadLocal实例。
        把ThreadLocalMap中的key设计为weakReference，也使set方法可以通过key==null && entry != null判断entry是否失效。
        */
```

## 总结

ThreadLocal类的使用虽然是用来解决多线程的问题的，但是还是有很明显的针对性。最明显的，ThreadLoacl变量的活动范围为某线程，并且我的理解是该线程“专有的，独自霸占”，对该变量的所有操作均有该线程完成！也就是说，ThreadLocal不是用来解决共享，竞争问题的。