#### 单列容器

##### ArrayList

- 默认容量

  ```java
  private static final int DEFAULT_CAPACITY = 10;
  ```

- 最大容量

  ```java
  private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
  ```

- 容器

  ```java
  transient Object[] elementData; // non-private to simplify nested class access
  ```

- 压缩存储，将数组大小压缩为现有的容量。

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

- 扩容

  			```java
    private Object[] grow(int minCapacity) {
        //copyOf方法通过反射创建新数组
        return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
    }
    ```
    
- 浅拷贝

  			```java
    public Object clone() {
         try {
             ArrayList<?> v = (ArrayList<?>) super.clone();
             //下面这行只是将所有的引用拷贝到v.elementData中。
             v.elementData = Arrays.copyOf(elementData, size);
             v.modCount = 0;
             return v;
         } catch (CloneNotSupportedException e) {
             // this shouldn't happen, since we are Cloneable
             throw new InternalError(e);
         }
    }
  ```

- equals方法既检查类型、又检查数组是否相同，还检查对数组做的修改次数是否相同。

			```java
	public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
    
    
        if (!(o instanceof List)) {
            return false;
        }
        
        final int expectedModCount = modCount;
       	// ArrayList can be subclassed and given arbitrary behavior, but we can
        // still deal with the common case where o is ArrayList precisely
        boolean equal = (o.getClass() == ArrayList.class)
            ? equalsArrayList((ArrayList<?>) o)
            : equalsRange((List<?>) o, 0, size);
        
        checkForComodification(expectedModCount);
        return equal;
    }
    ```
    
- 计算hashcode的方法，用数组中每一个元素乘以31并累加。

    ```java
    	int hashCodeRange(int from, int to) {
            final Object[] es = elementData;
            if (to > es.length) {
                throw new ConcurrentModificationException();
            }
            int hashCode = 1;
            for (int i = from; i < to; i++) {
                Object e = es[i];
                hashCode = 31 * hashCode + (e == null ? 0 : e.hashCode());
            }
            return hashCode;
        }
    ```

- 序列化

    ```java
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();
        
        // Write out size as capacity for behavioral compatibility with clone()
        s.writeInt(size);
    
        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
    
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    ```

##### Vector

- 大部分方法与ArrayList一致，但是都是同步方法。
- 迭代器Enumeration是快速失败的。当然后来新加了iterator迭代器。

##### LinkedList

- 定义好所有双向链表的操作，如linkFirst、linkLast、unlinkFirst、unlinkLast、unlink、linkBefore等等。然后队列、栈等操作定义上层的逻辑API。

- 通过遍历来找指定的位置

  ```java
  Node<E> node(int index) {
      // assert isElementIndex(index);
  
      if (index < (size >> 1)) {
          Node<E> x = first;
          for (int i = 0; i < index; i++)
              x = x.next;
          return x;
      } else {
          Node<E> x = last;
          for (int i = size - 1; i > index; i--)
              x = x.prev;
          return x;
      }
  }
  ```

- 浅拷贝

  ```java
  public Object clone() {
      LinkedList<E> clone = superClone();
  
      // Put clone into "virgin" state
      clone.first = clone.last = null;
      clone.size = 0;
      clone.modCount = 0;
  
      // Initialize clone with our elements
      for (Node<E> x = first; x != null; x = x.next)
          clone.add(x.item);
  
      return clone;
  }
  ```

#### 双列容器

##### HashMap

- 默认容量和最大容量

  ```
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
  static final int MAXIMUM_CAPACITY = 1 << 30;
  ```

- 负载因子

  ```
  static final float DEFAULT_LOAD_FACTOR = 0.75f;
  ```

- 转化为树的节点个数阈值和转化为链表的节点个数阈值

  ```
  static final int TREEIFY_THRESHOLD = 8;
  static final int UNTREEIFY_THRESHOLD = 6;
  ```

- 转化为树的数组容量阈值

  ```
  static final int MIN_TREEIFY_CAPACITY = 64;
  ```

- Node类的hashcode使用key和value的hashcode异或。

  ```
  public final int hashCode() {
      return Objects.hashCode(key) ^ Objects.hashCode(value);
  }
  ```

- 计算key的hashcode，使用高16位和低16位异或来尽可能降低hash冲突。

  ```
  static final int hash(Object key) {
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
  ```

- 扩容时不重新计算hash，某些元素直接使用原索引+旧数组容量作为新的位置。

  ```
  因为扩容时直接乘以2，数组长度始终保持为2的幂，它的二进制码只有一个1。
  通过if ((e.hash & oldCap) == 0)选出保持在原来位置以及要放入新位置的元素。
  ```

- 插入时在链表尾插入，并且先插入再扩容。

  ```java
  	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else {
              Node<K,V> e; K k;
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;
              else if (p instanceof TreeNode)
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              else {
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {
                          p.next = newNode(hash, key, value, null);
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);
                          break;
                      }
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
              if (e != null) { // existing mapping for key
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
          ++modCount;
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);
          return null;
      }
  ```

- 浅拷贝

##### HashTable

- 默认容量：11

- 直接用key的hashcode对数组长度取余获取索引。

  ```java
  public synchronized boolean containsKey(Object key) {
      Entry<?,?> tab[] = table;
      int hash = key.hashCode();
      int index = (hash & 0x7FFFFFFF) % tab.length;
      for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
          if ((e.hash == hash) && e.key.equals(key)) {
              return true;
          }
      }
      return false;
  }
  ```

- 先扩容再插入，头插法

  ```java
  private void addEntry(int hash, K key, V value, int index) {
          Entry<?,?> tab[] = table;
          if (count >= threshold) {
              // Rehash the table if the threshold is exceeded
              rehash();
  
              tab = table;
              hash = key.hashCode();
              index = (hash & 0x7FFFFFFF) % tab.length;
          }
  
          // Creates the new entry.
          @SuppressWarnings("unchecked")
          Entry<K,V> e = (Entry<K,V>) tab[index];
          tab[index] = new Entry<>(hash, key, value, e);
          count++;
          modCount++;
      }
  ```

  

  



















| 修饰词     | 本类 | 同一个包的类 | 继承类 | 其他类 |
| ---------- | ---- | ------------ | ------ | ------ |
| private    | √    | ×            | ×      | ×      |
| 无（默认） | √    | √            | ×      | ×      |
| protected  | √    | √            | √      | ×      |
| public     | √    | √            | √      | √      |

```
static一般用来修饰成员变量或函数。但有一种特殊用法是用static修饰内部类，普通类是不允许声明为静态的，只有内部类才可以。

被static修饰的内部类可以直接作为一个普通类来使用，而不需实例化一个外部类。

当一个内部类没有使用static修饰的时候，不能直接使用内部类创建对象，必须先使用外部类对象点new内部类对象（外部类对象.new 内部类())。
```

```java
Integer的hashcode方法
    public static int hashCode(int value) {
            return value;
    }
String的hashcode方法
    public static int hashCode(byte[] value) {//for StringLatin1
        int h = 0;
        for (byte v : value) {
            h = 31 * h + (v & 0xff);//8位的byte，直接以31为基数求余数的逆运算
        }
        return h;
    }
	public static int hashCode(byte[] value) {//for StringUTF16
        int h = 0;
        int length = value.length >> 1;
        for (int i = 0; i < length; i++) {
            h = 31 * h + getChar(value, i);//8位的byte，首先左移8位再加上原来的byte，变为16位数再以31为基数求余数的逆运算
        }
        return h;
    }
    static char getChar(byte[] val, int index) {
        assert index >= 0 && index < length(val) : "Trusted caller missed bounds check";
        index <<= 1;
        return (char)(((val[index++] & 0xff) << HI_BYTE_SHIFT) |
                      ((val[index]   & 0xff) << LO_BYTE_SHIFT));
    }
    private static native boolean isBigEndian();

    static final int HI_BYTE_SHIFT;
    static final int LO_BYTE_SHIFT;
    static {
        if (isBigEndian()) {//Java为大端模式，即高位字节存储在低位地址上
            HI_BYTE_SHIFT = 8;
            LO_BYTE_SHIFT = 0;
        } else {
            HI_BYTE_SHIFT = 0;
            LO_BYTE_SHIFT = 8;
        }
    }
```



