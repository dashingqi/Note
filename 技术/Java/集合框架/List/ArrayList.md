## ArrayList的源码分析

#### ArrayList的定义

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

- ArrayList是继承至AbstractList，实现了 List、RandomAccess、Cloneable、Serializable接口
- 底层实现是基于Object数组实现的
- 默认长度是10
- 每次自动扩容是之前容量的1.5倍。

#### ArrayList的构造器

- 无参构造函数

  ```java
  public ArrayList() {
      this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
   }
  ```

- 参数为int类型的构造器

  ```java
  public ArrayList(int initialCapacity) {
    			//传入的参数代表数组的长度，当大于0的时候
          if (initialCapacity > 0) {
            	//创建一个长度为 initialCapacity的对象数组，赋给 elementData
              this.elementData = new Object[initialCapacity];
          } else if (initialCapacity == 0) {
            	//等于0 将一个length = 0的 对象数组赋给 elementData
              this.elementData = EMPTY_ELEMENTDATA;
          } else {
            //小于0 就抛出异常
              throw new IllegalArgumentException("Illegal Capacity: "+
                                                 initialCapacity);
          }
      }
  ```

- 参数为一个Collection对象的构造器

  ```java
  public ArrayList(Collection<? extends E> c) {
    			//先将Collection转化成一个数组，将数组的地址赋给elementData
          elementData = c.toArray();
          if ((size = elementData.length) != 0) {
              // c.toArray might (incorrectly) not return Object[] (see 6260652)
            // 大于0的时候，就进行一步深拷贝，将Collection对象内容copy到elementData中
              if (elementData.getClass() != Object[].class)
                  elementData = Arrays.copyOf(elementData, size, Object[].class);
          } else {
              // replace with empty array.
            	// 当size 等于0的时候，就将一个length == 0的对象数组赋给 elementData
              this.elementData = EMPTY_ELEMENTDATA;
          }
      }
  ```

#### ArrayList的add方法和addAll方法

- 一个参数的add方法

  ```java
  public boolean add(E e) {
    			//确保初始容量 将size+1
          ensureCapacityInternal(size + 1);  // Increments modCount!!
    			//	赋值操作
          elementData[size++] = e;
    			//添加成功后 返回true
          return true;
      }
  
  private void ensureCapacityInternal(int minCapacity) {
    			//如果当前的elementData是一个length == 0 空对象数组
    			//也就是执行第一次添加的操作
          if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            	//会将默认的数组的容量10 赋给 minCapacity 就是当第一添加的元素的时候，数组的长度为10
              minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
          }
  				//明确容量
          ensureExplicitCapacity(minCapacity);
      }
  
  private void ensureExplicitCapacity(int minCapacity) {
    // 将修改次数进行自增
          modCount++;
          // overflow-conscious code
    			//当前数组的长度+1 已经超过数组的容量 就调用grow(方法)
          if (minCapacity - elementData.length > 0)
              grow(minCapacity);
      }
  
  private void grow(int minCapacity) {
          // overflow-conscious code
    			//获取到当前数组的容量
          int oldCapacity = elementData.length;
    			// 数组的容量值扩大到1.5倍 并 赋给 newCapacity
          int newCapacity = oldCapacity + (oldCapacity >> 1);
          if (newCapacity - minCapacity < 0)
              newCapacity = minCapacity;
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          // minCapacity is usually close to size, so this is a win:
          //将老数据copy到新的数组中
          elementData = Arrays.copyOf(elementData, newCapacity);
      }
  ```
  
  - 检查是否需要扩容
  - 然后将元素插入到尾部 返回true
  
- 两个参数的add方法

  ```java
  public void add(int index, E element) {
    			//进行插入位置的合理性判断
          if (index > size || index < 0)
              throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
  				//判断是否需要扩容
          ensureCapacityInternal(size + 1);  // Increments modCount!!
    			//扩容之后，将index后的元素的角标+1，然后在index位置插入数据
    			//这个arraycopy是一个native层的方法
          System.arraycopy(elementData, index, elementData, index + 1,
                           size - index);
          elementData[index] = element;
          size++;
      }
  ```

  - 检查数组容量是否够用
  - 将index位置及其之后的元素向后移一位
  - 将新元素插入到index处。

- addAll()方法

  ```java
   public boolean addAll(Collection<? extends E> c) {
          Object[] a = c.toArray();
          int numNew = a.length;
     			//检查是否需要扩容
          ensureCapacityInternal(size + numNew); 
     			//将数据copy到数组中
          System.arraycopy(a, 0, elementData, size, numNew);
          size += numNew;
          return numNew != 0;
      }
  ```
```
  

#### ArrayList的get方法

- get(int index)方法

  ```java
  public E get(int index) {
    			//越界检查
          rangeCheck(index);
  				//根据角标返回数据
          return elementData(index);
      }
  private void rangeCheck(int index) {
          if (index >= size)
              throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
      }
  E elementData(int index) {
          return (E) elementData[index];
      }
```

  - 检查index是否大于数组容量-1 ，就是进行数组越界检查
  - 根据index，获取到数组中的数据，并将其返回

#### ArrayList的remove方法

- 根据角标移除 remove(int index)

  ```java
   public E remove(int index) {
     
          rangeCheck(index);
  
          modCount++;
          E oldValue = elementData(index);
  
          int numMoved = size - index - 1;
          if (numMoved > 0)
              System.arraycopy(elementData, index+1, elementData, index,
                               numMoved);
          elementData[--size] = null; // clear to let GC do its work
  
          return oldValue;
      }
  ```

  - 角标越界检查
  - 获取到数组中角标对应的数据
  - 将index+1位置以及后面的元素位置进行向前移动一位，覆盖被删除的元素
  - 将数组最后一位的元素置空
  - 将被覆盖的数据给返回回来

- 根据数据来移除 remove(Object o)

  ```java
  public boolean remove(Object o) {
          if (o == null) {
              for (int index = 0; index < size; index++)
                  if (elementData[index] == null) {
                      fastRemove(index);
                      return true;
                  }
          } else {
              for (int index = 0; index < size; index++)
                  if (o.equals(elementData[index])) {
                      fastRemove(index);
                      return true;
                  }
          }
          return false;
      }
  ```

  - 如果数据为null，遍历数组，找到数据对应的最小角标，给快速删除
  - 否则的话，同样还是遍历数组，找到相同的数据的角标，进行快速删除
  - 操作成功返回true，操作失败返回false，一般就是没有对应的数据。

  ```java
   private void fastRemove(int index) {
          modCount++;
          int numMoved = size - index - 1;
          if (numMoved > 0)
              System.arraycopy(elementData, index+1, elementData, index,
                               numMoved);
          elementData[--size] = null; // clear to let GC do its work
      }
  ```

  - 和对应的角标移除相比，快速删除不进行越界检查，也不返回对应的数据

#### ArrayList的trimToSize()

- 代码如下

  ```java
   public void trimToSize() {
          modCount++;
          if (size < elementData.length) {
              elementData = (size == 0)
                ? EMPTY_ELEMENTDATA
                : Arrays.copyOf(elementData, size);
          }
      }
  ```

  - 将数组容量缩小到元素的数量。Arrays.copyOf(elementData,size);

#### ArrayList的clear()方法

- 代码如下

  ```java
  public void clear() {
          modCount++;
  
          // clear to let GC do its work
          for (int i = 0; i < size; i++)
              elementData[i] = null;
  
          size = 0;
      }
  ```

  - 遍历数组，将数组中的数据置为null
  - 将保存数组中元素个数的字段 size 

