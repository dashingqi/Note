## 本文主要从如下几点学习LinkedHashMap

- LinkedHashMap是啥
- 代码实操
- 原理分析
- 图的形式展示双向列表

## LinkedHashMap是啥

- 继承 HashMap实现了Map接口的散列表，HashMap本身是数组加单向链表

- 数据结构：HashMap+双向链表；HashMap的数据结构是（数组+单向链表+（红黑树））

- 是根据插入或者访问顺序实现有序输出的HashMap，线程不安全的，允许key为null，value为null

  

## 代码实操

- put 、get、forEach

  ```java
   LinkedHashMap<Integer, String> linkedMap = new LinkedHashMap<>();
          linkedMap.put(1, "index1");
          linkedMap.put(3, "index3");
          linkedMap.put(5, "index5");
          linkedMap.put(4, "index4");
  
          System.out.println("key == 3" + linkedMap.get(3));
  
          linkedMap.forEach((k, v) -> {
              System.out.println("k == " + k + ", v == " + v);
          });
  // 运行结果
  key == 3 index3
  k == 1, v == index1
  k == 3, v == index3
  k == 5, v == index5
  k == 4, v == index4
    
     // 构造函数 打开按访问顺序排列
     LinkedHashMap<Integer, String> linkedMap = new LinkedHashMap<>(5,0.75f,true);
     linkedMap.put(1, "index1");
     linkedMap.put(3, "index3");
     linkedMap.put(5, "index5");
     linkedMap.put(4, "index4");
     System.out.println("key == 3" + linkedMap.get(3));
  
      linkedMap.forEach((k, v) -> {
         System.out.println("k == " + k + ", v == " + v);
      });
  
  // 运行结果为
  key == 3 index3
  k == 1, v == index1
  k == 5, v == index5
  k == 4, v == index4
  k == 3, v == index3
  ```

  





## 原理分析

#### 几个全局变量的解释

######  transient LinkedHashMapEntry<K,V> head;

- 是双向列表的头节点，指向双向列表的头节点。

###### transient LinkedHashMapEntry<K,V> tail;

- 是双向列表的尾节点，指向双向列表的尾节点。

###### final boolean accessOrder;

- accessOrder默认在LinkedHashMap的构造函数中赋值为false
- 如果为false，那么在迭代输出节点的时候，会按照插入的顺序进行输出
- 如果为true（使用了这个 new LinkedHashMap<>(5,0.75f,true) 构造函数创建LinkedHashMap），在迭代的时候，会按照节点的访问顺序输出节点，最近使用的放在双向列表的尾部。

#### LinkedHashMapEntry的介绍

- 继承至HashMap.Node，具有了单向链表的功能。

- 比HashMap的Node多了 befor和after这两个变量，这两个变量是用来维护LinkedHashMap的双向列表。

  ```java
  static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
          LinkedHashMapEntry<K,V> before, after;
          LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
              super(hash, key, value, next);
          }
      }
  ```

  

#### LinkedHashMap的构造函数介绍

- 构造函数如下

  ```java
  //常规用法
  public LinkedHashMap() {
          super();
          accessOrder = false;
      }
      
   //指定了初始化时数组的容量   
  public LinkedHashMap(int initialCapacity) {
          super(initialCapacity);
          accessOrder = false;
      }
   //指定了初始化时容量和加载因子   
  public LinkedHashMap(int initialCapacity, float loadFactor) {
          super(initialCapacity, loadFactor);
          accessOrder = false;
      }
      
  //指定了容量、加载因子和迭代输出时的顺序
  public LinkedHashMap(int initialCapacity,
                           float loadFactor,
                           boolean accessOrder) {
          super(initialCapacity, loadFactor);
          this.accessOrder = accessOrder;
      }
  // 采用一个Map来构造
   public LinkedHashMap(Map<? extends K, ? extends V> m) {
          super();
          accessOrder = false;
     			//批量插入
          putMapEntries(m, false);
      }
  ```

  

#### putI()方法的源码的分析

 **LinkedHashMap中没有重写put方法，但是在HashMap进行put操作的时候，有如下两点判断**

- 当key计算出来的位置，没有数据存在，会调用***newNode()***方法。
  - 该方法，在LinkedHashMap被重写了
  - 当LinkedHashMap调用put时，此时用到了多态，会调用LinkedHashMap中的***newNode()***方法
- 当key计算出来的位置，有数据存在；
  - 当key完全相同，进行数据的赋值，然后进行数据的替换。
    - 在数据进行替换的时候，调用了***afterNodeAccess(e)***方法
    - 该方法在LinkedHashMap有进行重写。
  - 当key所在的位置的数据结构时红黑树，调用了putTreeVal()方法，插入节点
  - 当key所在位置是一个单向链表；
    - 会在内部循环，如果key当前所在位置时存在数据的，那么就会判断p.next是否为null
    - 为null，就会调用***newNode()***创建一个新的节点插入进去。

**还有一个方法，afterNodeInsertion(evict)**

- 该方法在LinkedHashMap也进行了重写
- evict默认时true

###### newNode()方法内部调用了linkNodeLast(p);方法。

- 代码分析如下

  ```java
  Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
          //构造带双向链表属性的LinkedHashMapEntry的对象
          LinkedHashMapEntry<K,V> p =
              new LinkedHashMapEntry<K,V>(hash, key, value, e);
          //双向列表的维护
          linkNodeLast(p);
          return p;
      }
  
  private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
          //首先获取当前链表的最后一个元素
          LinkedHashMapEntry<K,V> last = tail;
          //当前插入的元素定义为最后一个元素
          tail = p;
          //如果之前的最后一个元素是null，说明之前的链表就是空的，所以当前的元素是第一个元素
          if (last == null)
              head = p;
          else {
              //如果之前的链表不是null
              //put前的最后一个元素，设置为当前put元素的前一个
              p.before = last;
              //当前put元素设置为put前最后一个元素的下一个。
              last.after = p;
            //这样就形成了一个双向列表
          }
      }
  ```

  

###### afterNodeAccess(e)方法分析

- 代码分析如下

  ```java
  // 会将当前访问dao的字点e，移动到双向链表的尾部
  void afterNodeAccess(Node<K,V> e) { // move node to last
      LinkedHashMapEntry<K,V> last;
      //当accessOrder为true，并且链表的尾部节点和 要替换的数据（旧数据）不相等
      if (accessOrder && (last = tail) != e) {
          LinkedHashMapEntry<K,V> p =
              (LinkedHashMapEntry<K,V>)e, 
        	//将旧数据中前一个节点的before变量赋给b
          b = p.before, 
        	//将旧数据中后一个节点的after变量赋给a
          a = p.after;
        	//
          p.after = null;
        	// 当b为null，说明p的前置节点为null，p之前时头节点
          if (b == null)
            //p的后置节点设置为链表头部
              head = a;
          else
            	//不为null ，将p的后置节点a更新为 p的前置节点的后置节点
              b.after = a;
        
          if (a != null)
            	//当 a不为null，将p的前置节点更新为p的后置节点
              a.before = b;
          else
            //如果原本p的后置节点为null，说明p就是链表的尾部节点，那么将p的前置节点数据b设置为链表的尾部数据
              last = b;
        // 当链表尾部为null，就将当前的p设置链表的头部数据
        // 可以这么理解，当现在的key与原有的key发生了hash碰撞，但是原有的key对应的value时null
          if (last == null)
              head = p;
          else {
            //更新当前节点p的前置节点为原尾节点last，last的后置节点为p
              p.before = last;
              last.after = p;
          }
        //尾节点的引用赋值为p
          tail = p;
        	//改变modCount
          ++modCount;
      }
  }
  ```

  

   

###### afterNodeInsertion方法分析

- 代码分析如下

  ```java
  void afterNodeInsertion(boolean evict) { // possibly remove eldest
          LinkedHashMapEntry<K,V> first;
          // removeEldestEntry 默认是返回false的 所以if内的代码不会去执行
          if (evict && (first = head) != null && removeEldestEntry(first)) {
              K key = first.key;
              //移除链表头部的元素
              removeNode(hash(key), key, null, false, true);
          }
      }
  ```

  

#### get()方法的源码分析

**LinkedHashMap重写了HashMap的get方法**

- 代码如下

  ```java
  public V get(Object key) {
          Node<K,V> e;
    			//首先根据key的hashCode值查找当前为对应的节点，如果不存就返回null
          if ((e = getNode(hash(key), key)) == null)
              return null;
    			//当accessOrder为true的时候，将当前查到的节点e，移动到链表的尾部
          if (accessOrder)
              afterNodeAccess(e);
    			//返回查询到节点的中的value
          return e.value;
      }
  ```

  

#### LinkedHashMap的删除操作

> LinkedHashMap没有重写remove方法，
>
> 我们知道remove方法内部调用了removeNode()方法，removeNode方法内部调用了afterNodeRemoval()
>
> LinkedHashMap内对afterNodeRemoval()方法进行重写

- 代码如下

  ```java
  //删除节点e的时候，将节点e在双向列表上的前置和后置的节点引用都置空，然后更新前后节点（这里的前后节点对应e的前后节点）的指向。
  void afterNodeRemoval(Node<K,V> e) { // unlink
          LinkedHashMapEntry<K,V> p =
              (LinkedHashMapEntry<K,V>)e;
          b = p.before;
          a = p.after;
    			//将待删除节点P的前置和后置节点都置空
          p.before = p.after = null;
    			//如果前置节点null，那么将后置节点设置为头部节点
          if (b == null)
              head = a;
          else
            // 前置节点的后置节点 更新为 当前p的后置节点所指向的节点
              b.after = a;
    			//当后置节点为null，将前置节点设置为尾部节点
          if (a == null)
              tail = b;
          else
            //将后置节点 与 前置节点的后置节点进行关联
              a.before = b;
      }
  ```

  

#### LinkedHashMap的containsValue()

- 代码如下

  ```java
  public boolean containsValue(Object value) {
    			//从头部节点开始，利用双向列表的特点，每次拿到节点的后置节点 进行循环判断
    			//匹配到就返回true，匹配不到就返回false
          for (LinkedHashMapEntry<K,V> e = head; e != null; e = e.after) {
              V v = e.value;
              if (v == value || (value != null && value.equals(v)))
                  return true;
          }
          return false;
      }
  ```

  

**相比较于HashMap**

- 代码如下

  ```java
  //HashMap采用了嵌套for循环，效率不太行呀
  public boolean containsValue(Object value) {
          Node<K,V>[] tab; V v;
          if ((tab = table) != null && size > 0) {
              for (int i = 0; i < tab.length; ++i) {
                  for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                      if ((v = e.value) == value ||
                          (value != null && value.equals(v)))
                          return true;
                  }
              }
          }
          return false;
      }
  ```

  

## 图的形式展示双向列表

> 上面讲的又是节点，又是前置节点，又是后置节点
>
> 然后什么accessOrder == true的时候，又要移动节点，又要更新前置和后置的引用
>
> 下面就以一张图表示一下。

![LinkedHashMap_put_get_操作示意图.png](https://upload-images.jianshu.io/upload_images/4997216-44ed5f122ef5d7b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)