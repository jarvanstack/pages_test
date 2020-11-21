

### 1.什么是HashMap？

首先要理解Hash和Map

#### Hash

举个例子 
问题：从下面的数组中找到值为C的元素
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119170309410.png#pic_center)

1.传统数组的使用
遍历每个下标，直到找到值为C的元素（时间复杂度O(n））
2.**使用Hash的做法**
* 首先每个元素都有可以计算出一个hash值，
* 然后我们再通过这个hash值计算它的下标位置，我们直接访问该位置元素（比如说 array[2]),时间复杂度是O(1)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119171153642.png#pic_center)
1. 首先计算要寻找的元素C的hash值，举例C的hash值为16018
2. 然后通过计算（源码中使用位与就算效果等同于取模运算 `(16018 & (16-1))` =`16018%16` = 2）得到index下标的值，**时间复杂度O(1)!!!**
#### Map
Map就是K，V键值对，比如我们张三为键，张三的信息（年龄，性别，爱好）为值，通过张三的名字就能得到张三的信息
```json
{
	'张三': {
					name: '张三',
					gender: '男',
					age: 13
			}
}
```
### 2.数据结构
Hash和Map是hashMap的基础

hashMap的数据结构是数组+链表（红黑树）
（我理解为数组上面挂了很多”串串“）

* 不同hash值的元素的键key值储存到数组（hash表）中
* 相同hash值的元素的key值储存到挂在数组上的链表（链表长度大于7之后就会转化为红黑树）中
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111917371645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phcnZhbjU=,size_16,color_FFFFFF,t_70#pic_center)

### 3.常用核心方法
#### put()
put() 是最核心的方法之一，例如put(“C3",zhangsan）zhangsan对象`{name:"张三",age:13,gender:"男"}`
1. 首先计算出键"C3"的hash值，例如 16018
2. 然后通过hash值计算出下标为2
3. 如果tab[2]为null就赋值（使用实体类Node节点封装key,hash,value,next)
4. tab[2] 不为null
   * 判断C位置的key是否等于C3,相等就更新值
   * 遍历链表（红黑树）插入或者更新值，（这里是红黑树，遍历红黑树，更新C2处的值）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119184527776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phcnZhbjU=,size_16,color_FFFFFF,t_70#pic_center)
 #### get()
 get()方法十分类似put()方法，这里列举`get（"C3")`的例子
 1. 计算“C2”的hash值为 16018
 2. 计算下标index = 2
 3. 找到index = 2 的位置，遍历红黑树（这里是红黑树，其他情况可能是链表）找到node.key = “C3" 的元素取出
#### resize()扩容
数组的长度是不能变的，hashMap的长度是可以变的，如何实现的hashMap长度可变呢？

答案就是**resize()**方法

每次put()值就会对现有的长度进行判断，如果现有的表的长度大于 thresholdI(阈值）就会进行一次扩容,每次扩容容量**Capacity**就会增加一倍

阈值**threshold**等于多少呢？

 阈值**threshold**的取值靠**loadFactory**负载因子（默认为0.75),`threshold = factory * capacity(容量） `

所以 **每次扩容阈值threshold也会增加一倍**


* capacity 的最大值 maxCapacity 为 2^30(2的30次方）
* 初始的Capacity 为 2 ^ 4 (2的四次方 = 16）
* 因为扩容会导致数组长度的变化，扩容后要rehash()重新设置元素的hash值
> 拓展：hash码是如何计算的？
> * String 就是String内容，String相同Hash码相同
> * Integer就是Integer的值int
> * Object 是通过地址转化实现的

总之：**在数组容量到达阈值threshold 的时候resize（）就会进行一次扩容，容量*2翻倍**


### 4.优化

HashMap 提供2个优化的参数
* initCapacity: 初始容量默认2^4= 16
* loadFactory: 负载因子默认 0.75

我们可以预测业务的大小自定义初始化容量initCapacity的大小，传入的initCapacity的大小应该为 2^n^（2的n次方）的大小，如果不是2^n^ ，HashMap会使用TableSizeFor()函数自动转换为2^n^

loadFactory是决定是否扩容的关键，0.75是经过测试的比较合适的一个值，如果loadFactory过小，将会浪费很多数组的空间，如果loadFactory 过大，会导致链表和红黑树上的元素增加，降低查询速度。

### 5.源码
put()
```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            //初始化扩容.
            n = (tab = resize()).length;
        //数组中没有此hash值，我们就创建一个，p在此处赋值.
        //这里通过取模的 & 【位与】位代表二进制，与代表 and && 这里作用类似于取模运算.
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {//数组中p有了hash值，我们就挂在链表（红黑树）上
            Node<K,V> e; K k;
            //第一元素hash值相同，直接update()
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果是红黑树结构，使用红黑树的put()
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {//如果是链表
                for (int binCount = 0; ; ++binCount) {
                    //如果下一个链表为空
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果 binCount >= 7 就化为红黑树，红黑树之前node已经插入了，
                        //这里是7的原因是没有算上刚刚插入的node.
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果hash值和key值相等，就是相同的key，就update()
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //链表插入成功，e不为空，返回oldValue
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
````
get()
```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
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
resize()
```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
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
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
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
        return newTab;
    }
```
tableSizeFor()
```java

    /**
     * Returns a power of two size for the given target capacity.
     * 结果十分amazing，数字都变成了 2^n -1 的大小，amazing
     * >>> 是无符号右移操作
     * |= 是什么操作，| 是位或 ，= 是等于，这个类似于 += 操作，组合来看的. amazing!!!!
     * 通过>>> 右移 和 | 位或 操作，实现将原来的二进制的 10010 的0 全部变成 11111
     * 然后最后 +1 就是 100000 变为2^n 大小了，十分巧妙.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```