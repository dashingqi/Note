## 本文主要从如下节点讲解

- LRU算法简介
- LruCache的简介
- LruCache的代码实操
- LruCache的原理
- LruCache的总结

## LRU算法简介

- LRU（Least Recently Used）：是最近最少使用的算法
- 这个算法的核心就是，当缓存要满的时候，会优先淘汰最近最少使用的缓存对象。
- 采用LRU算法的有LruCache和DisLruCahce，分别用于内存缓存和磁盘缓存。

### LruCache的简介

- LruCache是一个泛型类。
- 主要原理就是把最近使用的对象的强引用存储在LinkedHashMap中。当缓存满是，会把最近最少使用的对象从内存中移除。
- 底层就是在维护一个LinkedHashMap。

### LruCache的简单代码实操

- 代码如下

  ```java
  public class MainActivity extends AppCompatActivity {
      private static final String TAG = "MainActivity";
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          //初始化一个容量为5的LruCache
          LruCache<Integer, Integer> lruCache = new LruCache<>(5);
          //调用put函数添加数据
          lruCache.put(1, 1);
          lruCache.put(2, 2);
          lruCache.put(3, 3);
          lruCache.put(4, 4);
          lruCache.put(5, 5);
          // 通过snapshot()方法获取Map，拿到Entry
          for (Map.Entry<Integer, Integer> entry : lruCache.snapshot().entrySet()) {
              Log.d(TAG, "key = " + entry.getKey() + ", value = " + entry.getValue());
          }
          Integer integer = lruCache.get(2);
          Log.e(TAG, "超出容量");
          //在容量满的情况下，进行put操作
          //通过Log打印，新添加的数据放在队列的尾部，而队列的头部的数据会被移除。
          lruCache.put(6, 6);
          for (Map.Entry<Integer, Integer> entry : lruCache.snapshot().entrySet()) {
              Log.d(TAG, "key = " + entry.getKey() + ",value = " + entry.getValue());
          }
  
      }
  }
  ```

- 运行结果如下

  ```java
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 1, value = 1
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 2, value = 2
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 3, value = 3
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 4, value = 4
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 5, value = 5
  2020-03-06 10:19:20.264 10184-10184/? E/MainActivity: 超出容量
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 3,value = 3
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 4,value = 4
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 5,value = 5
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 2,value = 2
  2020-03-06 10:19:20.264 10184-10184/? D/MainActivity: key = 6,value = 6
  ```

  

### LruCache的原理

###### LruCache的构造方法

- 代码如下

  ```java
  public LruCache(int maxSize) {
          if (maxSize <= 0) {
              throw new IllegalArgumentException("maxSize <= 0");
          }
          //lrucahche的容量值
          this.maxSize = maxSize;
          //声明一个容量为0，加载因子为0.75 并且开启访问顺序排序的LinkedHashMap
          this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
      }
  ```

  

###### LruCache的put()方法

- 代码如下

  ```java
  public final V put(K key, V value) {
          if (key == null || value == null) {
              throw new NullPointerException("key == null || value == null");
          }
  
          V previous;
          synchronized (this) {
              //添加次数自加
              putCount++;
              // safeSizeOf() 默认返回1 先给 size+1；
              size += safeSizeOf(key, value);
              //如果当前添加的key在底层的LinkedHashMap有存在相同，那就把原有的包含键值对的Entry给返回回来
              // 不存在，就返回 null
              previous = map.put(key, value);
              //说明存在要替换的数据
              if (previous != null) {
                  // 将size -1
                  size -= safeSizeOf(key, previous);
              }
          }
  
          //有存在要替换的数据
          if (previous != null) {
              // 该方法默认为空实现
              entryRemoved(false, key, previous, value);
          }
          //进行容量的检测，如果添加数据的时候，超出了容量值，就将头部的数据移除
          trimToSize(maxSize);
          return previous;
      }
  
  //trimToSize的源码解析
  private void trimToSize(int maxSize) {
          while (true) {
              K key;
              V value;
              synchronized (this) {
                  if (size < 0 || (map.isEmpty() && size != 0)) {
                      throw new IllegalStateException(getClass().getName()
                              + ".sizeOf() is reporting inconsistent results!");
                  }
                  // 实际存储数据个数 小于等于 最大容量，结束函数的执行
                  if (size <= maxSize) {
                      break;
                  }
  
                  // BEGIN LAYOUTLIB CHANGE
                  // get the last item in the linked list.
                  // This is not efficient, the goal here is to minimize the changes
                  // compared to the platform version.
                  Map.Entry<K, V> toEvict = null;
                  for (Map.Entry<K, V> entry : map.entrySet()) {
                    //找到最近最少使用的Entry
                      toEvict = entry;
                  }
                  // END LAYOUTLIB CHANGE
  
                  if (toEvict == null) {
                      break;
                  }
  
                  key = toEvict.getKey();
                  value = toEvict.getValue();
                  //移除元素
                  map.remove(key);
                  //size -1
                  size -= safeSizeOf(key, value);
                  //缓存移除的次数+1；
                  evictionCount++;
              }
  
              entryRemoved(true, key, value, null);
          }
      }
  ```

###### LruCache的get()方法

- 代码如下

  ```java
  public final V get(K key) {
          //对key的非空判断
          if (key == null) {
              throw new NullPointerException("key == null");
          }
  
          V mapValue;
          synchronized (this) {
              // 通过key获取到value
              mapValue = map.get(key);
              //如果value不为null
              if (mapValue != null) {
                  //缓存命中次数自增
                  hitCount++;
                  //返回当前的value
                  return mapValue;
              }
              //缓存没有命中次数自增
              missCount++;
          }
  
          /*尝试创建一个值，这将会花费一段时间，create方法创建的值有可能导致哈希表发生变化，
           *当哈希表中添加了一个冲突的值的时候，将会保留原有的值。
           * Attempt to create a value. This may take a long time, and the map
           * may be different when create() returns. If a conflicting value was
           * added to the map while create() was working, we leave that value in
           * the map and release the created value.
           *
           * 注： 其实create方法是未加锁的，当多个线程调用同一个key，生成多个value，就发生了hash碰撞了，
           * 从而保留第一个被添加进去的值。
           * 
           */
  
          //调用create方法创建一个 createValue  create方法默认返回一个null
          V createdValue = create(key);
          if (createdValue == null) {
              return null;
          }
  
          //同样，我们可以重写create方法，返回一个不为null的value
          //这样就执行如下的代码了
          synchronized (this) {
              //创建值自增
              createCount++;
              //将key和对应的创建值存到LinkedHashMap中。
              //如果该key之前存在，那么返回原有的值，否则返回null
              mapValue = map.put(key, createdValue);
  
              if (mapValue != null) {
                  // There was a conflict so undo that last put
                  //如果返回的value不为null，就将之前源于的值重新存储到LinkedHashMap中
                  map.put(key, mapValue);
              } else {
                  // size +1
                  size += safeSizeOf(key, createdValue);
              }
          }
          //如果存在原有的值
          if (mapValue != null) {
              entryRemoved(false, key, createdValue, mapValue);
              //返回原有的值
              return mapValue;
          } else {
              //进行容量的检测，如果超出了容量的值，就将头部的数据移除
              trimToSize(maxSize);
              //返回创造的值
              return createdValue;
          }
      }
  ```

###### LruCache的remove方法

- 代码如下

  ```java
  public final V remove(K key) {
          // 对key做非空判断
          if (key == null) {
              throw new NullPointerException("key == null");
          }
  
          V previous;
          synchronized (this) {
              //根据key移除链表中的Node的value，并返回Node的value
              previous = map.remove(key);
              if (previous != null) {
                  //数据删除后，需要经size-1
                  size -= safeSizeOf(key, previous);
              }
          }
          //当获取到的值不为ull。
          // 调用了 entryRemoved方法
          if (previous != null) {
              entryRemoved(false, key, previous, null);
          }
          //返回当前要移除的值
          ret
  ```

  

###### LruCache的entryRemoved()

> 该方法在我们分析的方法都有调用
>
> put：        entryRemoved(false, key, previous, value);
>
> trimSize：entryRemoved(true, key, value, null);
>
> get：         entryRemoved(false, key, createdValue, mapValue);
> remove： entryRemoved(false, key, previous, null);

- 其实我们可以通过重写 entryRemoved()方法获得如下几个信息
  - evicted == true 代表着这个方法是在trimToSize方法中调用的。
  - evicted == false
    - newValue == null 代表是在remove方法中调用的
    - newValue !=  null 代表是在get或者put方法中执行的

## LruCache的总结

- LruCache底层是在维护一个LinkedHashMap
- 针对 k/v 都有非空检查，key和value都不能为null
- LruCache使用并不是线程安全的，需要重写entryRemoved方法解决异常情况。