## 该篇文章主要从如下几个方面来学习下HashMap

- HashMap是啥
- HashMap的代码实操
- HashMap的get和put方法的源码分析
- 在1.7 和 1.8的区别
- 两个小问题

## HashMap是啥

- HashMap是继承至AbstractMap，实现了Map接口的 线程不安全的无序的散列表。
- 内部根据Key的hashCode存储key-value键值对。

## HashMap的代码实操

```java
 Map<Integer, String> map = new HashMap<>();
        map.put(2, "index2");
        map.put(4, "index4");
        map.put(3, "index3");
        map.put(1, "index1");

        System.out.println("key ==3 ,value == " + map.get(3));
        map.forEach((k, v) -> {
            System.out.println("k == " + k + ",v == " + v);
        });
//运行结果
key ==3 ,value == index3
k == 1,v == index1
k == 2,v == index2
k == 3,v == index3
k == 4,v == index4
```



## HashMap的原理分析

#### HashMap的构造方法

```java
/**
*创建一个HashMap对象，其中加载因子默认为0.75 ，
*这个加载因子是当用来扩容的时候进行使用的，当前HashMap的容量超过了总容量的75%就
*会进行扩容的操作！
*
*/
public HashMap() {
  		
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

/**
 * @param  initialCapacity the initial capacity. 
 * @throws IllegalArgumentException 如果当前传进的initialCapacity为负数，就  
 * 报这个错误
 * 创建一个初始容量为 initialCapacity 加载因子为0.75的一个HashMap对象
 */
public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
 /**
     *         
     * 1. 构造一个自定义的 容量和加载因子的 HashMap对象
     */
public HashMap(int initialCapacity, float loadFactor) {
  // 如果自定义的容量为负数就抛出异常
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
  			//如果自定义的容量大于 1 << 30
        if (initialCapacity > MAXIMUM_CAPACITY)
          	//就将 1 << 30 赋值给 initialCapacity
            initialCapacity = MAXIMUM_CAPACITY;
  			//如果加载因子小于等于0 或者是非数字 就抛出异常
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
  			// 转换initialCapacity
        this.threshold = tableSizeFor(initialCapacity);
    }
/**
* 构造一个和 传入Map有相同元素的HashMap，
* 初始容量能容下传入的Map，加载因子为0.75
*
**/
 public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }

final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

###### tableSizeFor(int cap)

```java
 /**
     * Returns a power of two size for the given target capacity.
     返回给定目标容量两个大小的幂
     */
    static final int tableSizeFor(int cap) {
      	// cap-1 是为了防止当前入参已经是2的幂，结果返回的却是入参的2倍
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
      // 该方法最总返回的是，离入参最近的2的n次幂，比如入参是3 返回4，入参 5 返回是8 
    }
```

**移位运算**

- 将二进制数据移动位置并补0，比如 11000>>>1 为 01100  ；01100 >>> 2 为00010

**或运算**

- 有一个1则为1，全是0位0

#### HashMap的put()方法

```kotlin
// 将指定的value 与映射中的 key进行映射
// 如果该映射中先前包含该键的映射，那么替换旧的值 并且返回旧值。
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }


final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
  			//如果table为null 或者 table的长度为0
  			//就调用resize()方法 将返回的 Node[K,V]数组赋给tab 并将该数组的长度赋给 n
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
  			// 如果当前tab的最后一个Node为null 说明tab为空
  			// 那么 就调用newNode()方法创建一个新的Node对象，存在在tab数组中
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
          	// 如果当前key对应的位置是有数据的，说明发生了hash冲突
            Node<K,V> e; K k;
          	// 当前key和原有的key相同，就将数组中数据赋值给变量 e
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
          //如果当前 p 是否是 红黑树结构 就调用 putTreeVal
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
              	// 当前不是红黑树结构，就是单链表结构
                for (int binCount = 0; ; ++binCount) {
                  		//如果链表中Node中的next为null ，说明是单链表中尾部的数据
                     // e= p.next;
                  	//  if(e == null)
                    if ((e = p.next) == null) {
                      // 调用newNode方法 创刊一个新的 Node，同时 将创建新的Node赋值给 p.next
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  //当 新的key与原有的key相同 直接跳出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
          
          // e!=null 说明存在有相同的key 就把值进行覆盖然后 将原有的值 返回
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
              //这个方法在HashMap中没有做任何事情，只会在存在相同的key才调用 在子类LinkedHashMap中有具体的实现
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
  		//	该方法会在每次put的时候（没有存在相同key的时候）都会调用，在子类LinkedHashMap中有具体的实现
        afterNodeInsertion(evict);
        return null;
    }
// putTreeVal：循环遍历红黑树的节点，如果节点的key与新的key相同，如果相同返回原有的Node对象，不同就返回null
```

###### HashMap的put方法总结

- 将全局的变量table赋值给局部的变量table，如果tab为null就调用reSize()进行初始化 将新建的Node[] 赋值给tab，将初始化好的数组长度赋值给 n。
- 拿到key对应位置的数据，如果为null说明该位置之前没有存放过数据，则调用newNode方法创建一个新的数据存放到tab中
- 拿到的key计算出来的hashCode对应的位置之前存在过，说明发生了hash碰撞，那么判断当前的key和存在的key是否真正的相等，相同则替换对象。
- 如果key不相等，判断当前碰撞的位置的数据结构是否是黑红树，如果是就调用putTreeVal()方法，如果发现书的节点key与新的key存在相同，那么把旧的node返回赋值给e，否则返回null
- 以上两者都不是，说明当前位置对应的数据结构就是一个单项链表，那么循环判断当前位置的node对象持有的下一个节点对象是否为null（e = p.next），为null就通过newNode方法创建一个新的节点，指向了p.next，并且判断当前容量是否超出了8（默认值），超出了就将当前的单链表结构转换成红黑树结构，调用方法treeifyBin(),然后跳出循环；如果当前位置的key和新的key相等也跳出循环；否则的话将p.next赋值给p，然后重复循环，知道满足当中的一个条件跳出循环。
- 跳出循环后，判断当前e时候为null，不为null说明当前e持有的是原有的node对象，就重e中获取之前的旧值替换成新值，然后返回旧值。
- 判断数组中的数据个数是否大于了阀值，大于了就进行扩容，调用reSize方法；然后调用了afterNodeInsertion(evict);并且返回null
- get方法要么返回 null 要么返回相同的key对应的旧值。

###### resize方法

```java
final Node<K,V>[] resize() {
        //全局的table赋值给oldTab
        Node<K,V>[] oldTab = table;
        // 保存之前的 Node数组的长度
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //保存之前扩容的阀值
        int oldThr = threshold;
        //
        int newCap, newThr = 0;
        //如果oldCap大于0，说明扩容之前的 table不为null
        if (oldCap > 0) {
            // 如果之前的老table的长度大于了MAXIMUM_CAPACITY
            if (oldCap >= MAXIMUM_CAPACITY) {
                //就将扩容的阀值设置为Integer.MAX_VALUE;
                threshold = Integer.MAX_VALUE;
                //并且返回老的table，说明不在进行扩容操作了
                return oldTab;
            }
            // 将 老table的容量进行翻倍赋值给newCap，newCap小于MAXIMUM_CAPACITY
            //并且扩容之前的oldCap大于16，那么将原有阀值翻倍赋值给新的阀值存储
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        //说明 老的table并没有赋值，oldTab为null
        //那么oldCap为0，这时oldThr大于0 就是阀值大于0 说明 initialCapacity大于0
        //HashMap是通过如下构造方法构建的
        //public HashMap(int initialCapacity)
        //public HashMap(int initialCapacity)
        //public HashMap(Map<? extends K, ? extends V> m)
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {// zero initial threshold signifies using defaults
            //能走到这个分支里面，说明 oldThr =0，oldCap<=0 ，也就是HashMap是通过 new HashMap()创建的
            // 新创建的HashMap的数组容量默认为16
            newCap = DEFAULT_INITIAL_CAPACITY;
            //新的扩容阀值默认是(int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); 12
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //如果老的oldTab不为null 就将旧数据迁移到新的table中。
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        //如果是一个节点就将数据放到newTab的指定位置上。
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果当前节点的数据结构是红黑树的
                    else if (e instanceof TreeNode)
                        //就进行红黑树的split操作
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        //否则的话就是一个单链表
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                          // JDK1.8 优化的地方，没有重新计算每个key的hash值
                          //而是通过高位元算来决定 扩容后的位置是否有变动
                          //如果结果高一位为0 表示扩容后的位置没有变动
                          //如果为高一位为1 表示有变动，位置为原位置+原数组的长度
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
  //返回新的扩容后的table
        return newTab;
    }
```

**resize方法的总结**

- resize主要是返回一个新的table和并进行扩容。
- 如果oldTab不为null，就判断oldTable.length与MAXIMUM_CAPACITY做比较，如果大于等于 就将阀值设为值整形最大值，并且停止扩容；如果小于并且oldTable.length的2倍也小于MAXIMUM_CAPACITY并且oldTable.length大于等于默认的阀值就将阀值扩大为当前的2倍，数组容量扩大为2倍。
- 在JDK1.7时 是头插法，到了1.8是尾差法。

#### HashMap的get()方法

```java
public V get(Object key) {
        Node<K,V> e;
        //根据key的hashCode去查找位置，返回对应的 node 从而拿到了value的值。
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

 final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
   			//table不为null，并且根据key的hashCode找到存放的位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
          // 如果该位置对应的第一个就是就返回链表头。
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
          //第一个不是
            if ((e = first.next) != null) {
              下一个是红黑树，就通过getTreeNode获取
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
              //不是红黑树 是链表结构 就不但循环取 直到找到key相同的node，返回该node
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

###### get方法的总结

- 根据key的hashCode值找到存放的位置对应的数据
- 当链表的第一个数据就是就返回这个链表头
- 如果不是的话，就判断当前节点的数据结构是否是红黑树，是的话就调用getTreeNode方法获取，不是的话就循环调用，比对key直到找到相同的key，返回对应的node；
- 如果对应的key找不到相应的位置，就返回null

## HashMap 1.7和1.8的差别

|         差异类别         |                             1.7                              |                             1.8                              |
| :----------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|         数据结构         |                     数组+链表（Entry[]）                     | 数组+链表（Node[]）+红黑树：<br />当链表的长度>8的时候转换成红黑树 |
|      数据存放的方式      | key的hashCoded没有发生碰撞的时候，就采用数组存储，<br />发生碰撞了就采用单链表的数据结构 | key的hashCoded没有发生碰撞的时候，就采用数组存储，<br />发生碰撞了就采用单链表的数据结构，当链表的长度>8的时候采用红黑树 |
|      数据插入的方式      |                   头插法（原有数据向后移）                   |                            尾插法                            |
| 扩容后，老数据的移动放肆 |       头插法（原数据会出现逆序，出现环形链表，死循环）       |        尾插法（不会出现逆序，环形列表，死循环等问题）        |
|         扩容时机         |                  先判断大于阀值的时候在插入                  |              数据先插入然后判断大于阀值，在扩容              |

## 几个问题

###### hash碰撞是怎回事以及怎么解决

> 当调用put方法的时候，会先进行key的hash计算，通过hash计算找到要存放的位置，如果此时这个存在数据，说明不同的key有相同的hashCode值，就发生了hash碰撞。
>
> 
>
> 1.8之前采用数组+单链表的数据结构解决的hash碰撞的问题
>
> 
>
> 1.8的时候采用数组+单链表+红黑树结构解决了这个问题。
>
> 我们知道1.8之前的单链表是采用头插法，在扩容的时候，容易出现逆序，死循环的问题，所以在1.8的时候加入红黑树 优化了1.7出现的问题。



###### 为什么HashMap中的String，Interger适合作为key

> 因为包装类 String，Integer是fianl修饰的，保证了计算key时 hash的不可更改和准确性
>
> 有效减少了hash碰撞的几率。内部重写了equals()和hashCode()方法，不容易出现hash值计算的错误。

