> List、Set继承Collection，Map是独立的接口

## 一、List

> List遍历的两种方式：
>
> 1. foreach
> 2. iterator【线程安全】

<!-- tabs:start -->

#### **ArrayList**

集合内元素有序且可重复

数据结构是数组，查询快，增删慢

线程不安全，效率高

#### **Vector**

集合内元素有序且可重复

数据结构是数组，查询快，增删慢

线程安全，效率低

#### **LinkedList**

集合内元素有序且可重复

数据结构是链表，查询慢，增删快

线程不安全，效率高

<!-- tabs:end -->

## 二、Set

>以下三者线程不安全，如果要使用线程安全可以Collections.synchronizedSet()

<!-- tabs:start -->

#### **HashSet**

集合内数据无序且唯一

数据结构为哈希表，保证元素唯一依赖的两个方法：hashCode() 和 equals()

允许null数据，但只能有一个null

> 存储元素首先会使用hash()算法函数生成一个int类型hashCode散列值，然后已经的所存储的元素的hashCode值比较，如果hashCode不相等，则所存储的两个对象一定不相等，此时存储当前的新的hashCode值处的元素对象；如果hashCode相等，存储元素的对象还是不一定相等，此时会调用equals()方法判断两个对象的内容是否相等，如果内容相等，那么就是同一个对象，无需存储；如果比较的内容不相等，那么就是不同的对象，就该存储了，此时就要采用哈希的解决地址冲突算法，在当前hashCode值处类似一个新的链表， 在同一个hashCode值的后面存储存储不同的对象，这样就保证了元素的唯一性。
>
> HashSet采用哈希算法。默认初始化容量16，加载因子0.75。 

#### **LinkedHashSet**

集合内元素有序且唯一，FIFO

数据结构是链表和哈希表，链表保证数据有序，哈希表保证数据唯一

允许null数据，但只能有一个null

#### **SortedSet——>TreeSet**

TreeSet实现了SortedSet接口，SortedSet实现了Set接口

集合内元素有序且唯一

数据结构是红黑树

保证元素排序：自然排序和比较器排序

保证元素唯一：根据比较的返回值是否是0来决定

> 一、基本数据类型的Set可以自己完成排序；
>
> 二、引用数据类型需要声明排序规则；
>
>  1. 自然排序
>
>     引用数据类实现Comparable接口并重写Comparable接口中的Compareto方法。
>
>  2. 比较器排序
>
>     单独创建一个比较类，实现Comparator接口并重写Comparator接口中的Compare方法
>
>     在声明Set时引用此比较器

<!-- tabs:end -->

## 三、Queue

LinkedList既可以实现Queue接口,也可以实现List接口.只不过呢, LinkedList实现了Queue接口。Queue接口窄化了对LinkedList的方法的访问权限（即在方法中的参数类型如果是Queue时，就完全只能访问Queue接口所定义的方法 了，而不能直接访问 LinkedList的非Queue的方法），以使得只有恰当的方法才可以使用。

## 四、Map

>map遍历的两种方式：
>
>1. KeySet()
>
>   先获取map所有键存入set集合，因为set具备迭代器，所以通过迭代器获取key后再根据key获取map的值。
>
>   例如：
>
>   Iterator it = map.keySet().iterator();
>
>   while(it.hasNext()){
>    	Object key = it.next();
>    	System.out.println(map.get(key));
>    };
>
>2. entrySet()【效率高】
>
>   Iterator it = map.entrySet().iterator();
>   while(it.hasNext()){
>   	Entry e =(Entry) it.next();
>   	System.out.println("键"+e.getKey () + "的值为" + e.getValue());
>   }

<!-- tabs:start -->

#### **HashMap**

基于哈希表实现，保证元素唯一依赖的两个方法：hashCode() 和 equals()

元素排列无序

方法是非同步，线程不安全，效率高

允许key和value是null值，key只允许有一个null，value允许有多个null

> HashMap初始容量16，负荷系数0.75，初始阈值即容量乘以负荷系数为12；如果map的大小超过阈值，会进行扩容，map的内容会重新哈希；容量总是2的幂，

#### **LinkedHashMap**

有HashMap全部特性

> 与HashMap的区别：LinkedHashMap保存了记录的插入顺序，在用Iteraor遍历LinkedHashMap时，先得到的记录肯定是先插入的。

#### **TreeMap**

基于红黑树实现

元素排列有序

方法是非同步，线程不安全

#### **HashTable**

基于哈希表实现

元素排列无序

方法是同步，线程安全，效率低

不允许有null值

#### **IdentityHashMap**

IdentityHashMap使用 == 判断两个key是否相等，而HashMap使用的是equals方法比较key值。

> 对于==，如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等； 如果作用于引用类型的变量，则比较的是所指向的对象的地址。
>
> 对于equals方法，（注意：equals方法不能作用于基本数据类型的变量）如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址； 诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。 

#### **ConcurrentHashMap**

线程安全，并且锁分离。

基于哈希表实现

> ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。

<!-- tabs:end -->

## 五、Iterator

#### 1. Iterator的fail-fast属性：

​		每次我们尝试获取下一个元素的时候，Iterator fail-fast属性检查当前集合结构里的任何改动。如果发现任何改动，它抛出ConcurrentModificationException。Collection中所有Iterator的实现都是按fail-fast来设计的（ConcurrentHashMap和CopyOnWriteArrayList这类并发集合类除外）。

#### 2. fail-fast与fail-safe区别：

​		Iterator的fail-fast属性与当前的集合共同起作用，因此它不会受到集合中任何改动的影响。Java.util包中的所有集合类都被设计为fail-fast的，而java.util.concurrent中的集合类都为fail-safe的。Fail-fast迭代器抛出ConcurrentModificationException，而fail-safe迭代器从不抛出ConcurrentModificationException。

#### 3. Iterator对比ListIterator:

1. Iterator可遍历List和Set，ListIterator只能遍历List
2. Iterator只可向前遍历，ListIterator可双向遍历

