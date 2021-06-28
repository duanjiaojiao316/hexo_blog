---
title: Java 基础
categories:
  - Java
tags:
  - Java基础
translate_title: java-basics
date: 2019-12-15 20:25:17
---
# Java 基础

## 1、Java特性

### 继承

#### super关键字的使用

super关键字的两种用法：

1. 调用超类的构造函数
2. 访问超类中被子类的某个成员隐藏的成员（同名隐藏）

注意：

调用超类构造函数```super()```必须是子类构造函数中的第一条语句。

 `必须在构造器的第一行放置super或者this构造器，否则编译器会自动地放一个空参数的super构造器的，其他的构造器也可以调用super或者this，调用成一个递归构造链，最后的结果是父类的构造器（可能有多级父类构造器）始终在子类的构造器之前执行，递归的调用父类构造器。无法执行当前的类的构造器。也就不能实例化任何对象，这个类就成为一个无为类。 从另外一面说，子类是从父类继承而来，继承了父类的属性和方法，如果在子类中先不完成父类的成员的初始化，则子类无法使用，因为在java中不允许调用没初始化的成员。在构造器中是顺序执行的，也就是说必须在第一行进行父类的初始化。而super能直接完成这个功能。this()通过调用本类中的其他构造器也能完成这个功能。 因此，this()或者super()必须放在第一行。` 

#### 向上转型

```java
 Person s = new Student(15,"djj",96);
```

但是用过s引用不能查找子类中的变量实现等。

如果子类覆写父类的一个方法，s调用这个方法，实现是student的实现。

```java
public class Person {
    protected int age;
    protected String name;

    public Person(int age, String name) {
        this.age = age;
        this.name = name;
    }
    public void print(){
        System.out.println("the Person");
    }
}

class Student extends Person{
    protected int score;

    public Student(int age, String name, int score) {
        super(age, name);
        this.score = score;
    }
    public void print(){
        System.out.println("the Student");
    }
}

public class Main {
    public static void main(String[] args) {
        Person s = new Student(15,"djj",96);
        s.print();
    }
}


执行结果：
    the Student
```

#### 向下转型

向下转型可能会失败，抛出 ```ClassCastException ```异常。

#### 区分继承和组合

is关系采用继承，但是has关系使用组合

```java
class Student extends Person{
    protected Book book;
    protected int score;
}
```

### 多态

 多态是指，针对某个类型的方法调用，其真正执行的方法取决于运行时期实际类型的方法。 

 利用多态，`totalTax()`方法只需要和`Income`打交道，它完全不需要知道`Salary`和`StateCouncilSpecialAllowance`的存在，就可以正确计算出总的税。如果我们要新增一种稿费收入，只需要从`Income`派生，然后正确覆写`getTax()`方法就可以。把新的类型传入`totalTax()`，不需要修改任何代码。 

```java
public class Income {
    protected double income;
    //计算税
    public double getTax() {
        return income * 0.1;
    }
}
//工资
class Salary extends Income {
    @Override
    public double getTax() {
        if (income <= 5000) {
            return 0;
        }
        return (income - 5000) * 0.2;
    }
}
//国家津贴
class StateCouncilSpecialAllowance extends Income {
    @Override
    public double getTax() {
        return 0;
    }
}
public class Main {
    public static void main(String[] args) {
        Income income = new Income();
        income.income = 1000;
        Salary salary = new Salary();
        salary.income = 6000;
        StateCouncilSpecialAllowance state = new StateCouncilSpecialAllowance();
        state.income = 5000;
        System.out.println(totalTax(salary, state));
        System.out.println(totalTax(state));
        System.out.println(totalTax(salary));
        System.out.println(totalTax(income, salary));
    }
    public static double totalTax(Income... incomes) {
        double total = 0;
        for (Income income: incomes) {
            total = total + income.getTax();
        }
        return total;
    }
}

class A
{
    void foo(){}
}
class B：public A
{
    void foo(){}
}
A *pa = new B();
pa->foo();

执行结果：
200.0
0.0
200.0
300.0
```

## 2、包

### 2.1包和成员访问

|                      | private | 无访问修饰符 | protected | public |
| -------------------- | ------- | ------------ | --------- | ------ |
| 在同一个类中是否可见 | 是      | 是           | 是        | 是     |
| 相同的包中的子类     | 否      | 是           | 是        | 是     |
| 相同的包中的非子类   | 否      | 是           | 是        | 是     |
| 不同包中的子类       | 否      | 否           | 是        | 是     |
| 不同包中的非子类     | 否      | 否           | 否        | 是     |

- public 可以在任何位置访问
- 默认访问级别 子类或者相同包的下的其他类可以访问
- protected 可以在包外访问，但是仅限子类可以访问
- private只能在自己的类中访问

## 3、接口

### 3.1接口中的变量

可以使用接口将共享的变量导入到多个类中。

```java
interface SharedConstants{
    
    int NO = 0;
    int Yes = 1;
    ...
}
```

<font color="red">这些变量在作用域内会被作为常量。</font>

### 3.2默认接口方法

在JDK1.8之前没有默认接口方法，默认方法之前使用关键字default。

动机：

1. 给接口添加新的方法，不会破坏之前实现该接口的代码。由于必须实现接口中的所有方法，如果接口加入新的方法，那么实现它的类就必须实现该方法，这样就会破坏现有的代码。
2. 让新方法的实现具有可选性。有些类可以选择不使用。

### 3.3多级继承问题

一个类实现两个接口，这两个接口提供了相同的方法，这个方法会产生冲突。

类的实现的优先级高于接口默认方法的优先级。所以可以通过类的实现解决冲突。

### 3.4接口的静态方法

同类的静态方法，无须实现接口，也无须接口的实例，只需要接口名就可以使用接口的默认静态方法。

### 3.5私有的接口方法

## 4、异常

### 4.1链式异常

链式异常可以为异常关联另一个异常，为了描述造成以第一个异常的原因。

构造函数关联

```java
	public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }
    public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
    }
```

支持链式异常的方法

```java
    public synchronized Throwable getCause() {
        return (cause==this ? null : cause);
    }
    public synchronized Throwable initCause(Throwable cause) {
        if (this.cause != this)
            throw new IllegalStateException("Can't overwrite cause with " +
                                            Objects.toString(cause, "a null"), this);
        if (cause == this)
            throw new IllegalArgumentException("Self-causation not permitted", this);
        this.cause = cause;
        return this;
    }
```

getCause方法返回引起当前异常的异常，没有就返回null。

对于每个异常对象只能进行一次initCause，如果使用构造函数关联就不能再使用initCause进行设置。通常initCause方法是为了解决之前不支持链式异常的异常类（不包含支持链式异常的构造方法）不能关联异常设置的。

## 5、IO (BIO) NIO AIO

### 5.1 BIO、NIO、AIO的区别

1. BIO 就是传统的 [java.io](http://java.io/) 包，它是基于流模型实现的，交互的方式是**同步、阻塞**方式，也就是说在读入输入流或者输出流时，在读写动作完成之前，线程会一直阻塞在那里，它们之间的调用时可靠的线性顺序。它的有点就是代码比较简单、直观；缺点就是 IO 的效率和扩展性很低，容易成为应用性能瓶颈。
2. NIO 是 Java 1.4 引入的 java.nio 包，提供了 Channel、Selector、Buffer 等新的抽象，可以构建**多路复用**的、**同步非阻塞** IO 程序，同时提供了更接近操作系统底层高性能的数据操作方式。
3. AIO 是 Java 1.7 之后引入的包，是 NIO 的升级版本，提供了**异步非堵塞**的 IO 操作方式，所以人们叫它 AIO（Asynchronous IO），异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

### 5.2 IO

#### 传统的 IO 大致可以分为4种类型：

- InputStream、OutputStream 基于字节操作的 IO
- Writer、Reader 基于字符操作的 IO
- File 基于磁盘操作的 IO
- Socket 基于网络操作的 IO

#### **InputStream OutputStream Writer Reader**

 ![img](D:\面试\springbootimages\javacore-io-001.png)  ![img](http://icdn.apigo.cn/blog/javacore-io-002.png)  



```java
InputStream inputStream = new FileInputStream("D:\\log.txt");
byte[] bytes = new byte[inputStream.available()];
inputStream.read(bytes);
String str = new String(bytes, "utf-8");
System.out.println(str);
inputStream.close();

OutputStream outputStream = new FileOutputStream("D:\\log.txt",true); // 参数二，表示是否追加，true=追加
outputStream.write("你好，老王".getBytes("utf-8"));
outputStream.close();
```

![img](http://icdn.apigo.cn/blog/javacore-io-004.png)  

![img](http://icdn.apigo.cn/blog/javacore-io-003.png) 

```java
Writer writer = new FileWriter("D:\\log.txt",true); // 参数二，是否追加文件，true=追加
writer.append("老王，你好");
writer.close();

Reader reader = new FileReader("D:\\log.txt");
BufferedReader bufferedReader = new BufferedReader(reader);
StringBuffer bf = new StringBuffer();
String str;
while ((str = bufferedReader.readLine()) != null) {
    bf.append(str + "\n");
}
bufferedReader.close();
reader.close();
System.out.println(bf.toString());
```

#### Java 7 引入了Files（java.nio包下）的，大大简化了文件的读写，如下：

```java
// 写入文件（追加方式：StandardOpenOption.APPEND）
Files.write(Paths.get(filePath), Content.getBytes(StandardCharsets.UTF_8), StandardOpenOption.APPEND);

// 读取文件
byte[] data = Files.readAllBytes(Paths.get(filePath));
System.out.println(new String(data, StandardCharsets.UTF_8));

// 创建多（单）层目录（如果不存在创建，存在不会报错）
new File("D://a//b").mkdirs();
```

一个简单的 Socket，服务器端只发给客户端信息，再由客户端打印出来的例子，代码如下：

```java
int port = 4343; //端口号
// Socket 服务器端（简单的发送信息）
Thread sThread = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            ServerSocket serverSocket = new ServerSocket(port);
            while (true) {
                // 等待连接
                Socket socket = serverSocket.accept();
                Thread sHandlerThread = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try (PrintWriter printWriter = new PrintWriter(socket.getOutputStream())) {
                            printWriter.println("hello world！");
                            printWriter.flush();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                });
                sHandlerThread.start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});
sThread.start();

// Socket 客户端（接收信息并打印）
try (Socket cSocket = new Socket(InetAddress.getLocalHost(), port)) {
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(cSocket.getInputStream()));
    bufferedReader.lines().forEach(s -> System.out.println("客户端：" + s));
} catch (UnknownHostException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
}
```

- 调用 accept 方法，阻塞等待客户端连接；
- 利用 Socket 模拟了一个简单的客户端，只进行连接、读取和打印；

在 Java 中，线程的实现是比较重量级的，所以线程的启动或者销毁是很消耗服务器的资源的，即使使用线程池来实现，使用上述传统的 Socket 方式，当连接数极具上升也会带来性能瓶颈，原因是线程的上线文切换开销会在高并发的时候体现的很明显，并且以上操作方式还是同步阻塞式的编程，性能问题在高并发的时候就会体现的尤为明显。

 ![img](http://icdn.apigo.cn/blog/javacore-io-005.png) 

Writer、Reader 基于字符操作的 IO

File 基于磁盘操作的 IO

Socket 基于网络操作的 IO

## 6、

### Hashmap是怎么实现的，底层原理？

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    private static final long serialVersionUID = 362498820763181265L;
    
    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    // 当resize操作时候发现链表长度小于6时，从红黑树退化为链表
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    
     /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    /**
    * 用于存储map中key和value的结构体
    */
    transient Node<K,V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;
```

`transient Node<K,V>[] table;`这表示HashMap是Node数组构成，其中Node类的实现如下，为HashMap的内部类，可以看出这其实就是个链表，链表的每个结点是一个<K,V>映射。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

`transient int modCount;``modCount`在`HashMap`，`ArrayList`和`LinkedList`中都有，不同之处ArrayList和LinkedList中的modCount继承自`AbstractList`的`protected transient int modCount = 0；`

> 在一个迭代器初始的时候会赋予它调用这个迭代器的对象的modCount，如果在迭代器遍历的过程中，**一旦发现这个对象的modCount和迭代器中存储的modCount不一样那就抛异常。**
>
> 
>
> **Fail-Fast机制**：java.util.HashMap不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。这一策略在源码中的实现是通过modCount域，modCount顾名思义就是修改次数，**对HashMap内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。**在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map。

**注意初始容量和扩容后的容量都必须是2的次幂**，为什么呢?

**hash方法**

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

HashMap的散列方法如上，其实就是将hash值的高16位和低16位异或，我们将马上看到hash在与n - 1相与的时候，高位的信息也被考虑了，能使碰撞的概率减小，散列得更均匀。



在JDK 8中，HashMap的putVal方法中有这么一句

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

关键就是这句`(n - 1) & hash`，**这行代码是把待插入的结点散列到数组中某个下标中**。

为什么HashMap的容量要始终保持2的次幂？

- **使散列值分布均匀**
- **位运算的效率比取余的效率高**

注意table.length是数组的容量，而`transient int size`表示存入Map中的键值对数。

`int threshold`表示临界值，当键值对的个数大于临界值，就会扩容。threshold的更新是由下面的方法完成的。

```java
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

该方法返回大于等于cap的最小的二次幂数值。比如cap为16，就返回16，cap为17就返回32。

**put方法**

put方法主要由putVal方法实现：

- 如果没有产生hash冲突，直接在数组`tab[i = (n - 1) & hash]`处新建一个结点；
- 否则，发生了hash冲突，此时key如果和头结点的key相同，找到要更新的结点，直接跳到最后去更新值
- 否则，如果数组下标中的类型是TreeNode，就插入到红黑树中
- 如果只是普通的链表，就在链表中查找，找到key相同的结点就跳出，到最后去更新值；到链表尾也没有找到就在尾部插入一个新结点。接着判断此时链表长度若大于8的话，还需要将链表转为红黑树（注意在要将链表转为红黑树之前，再进行一次判断，若数组容量小于64，则用resize扩容，放弃转为红黑树）

**get方法**

get方法由getNode方法实现：

- 如果在数组下标的链表头就找到key相同的，那么返回链表头的值
- 否则如果数组下标处的类型是TreeNode，就在红黑树中查找
- 否则就是在普通链表中查找了
- 都找不到就返回null

remove方法的流程大致和get方法类似。

**HashMap的扩容，resize()过程？**

newCap = oldCap << 1;

resize方法中有这么一句，说明是扩容后数组大小是原数组的两倍。

```java
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 如果数组中只有一个元素，即只有一个头结点，重新哈希到新数组的某个下标
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 数组下标处的链表长度大于1，非红黑树的情况
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // oldCap是2的次幂，最高位是1，其余为是0，哈希值和其相与，根据哈希值的最高位是1还是0，链表被拆分成两条，哈希值最高位是0分到loHead。
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 哈希值最高位是1分到hiHead
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
                            // loHead挂到新数组[原下标]处；
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // hiHead挂到新数组中[原下标+oldCap]处
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
```

举个例子，比如oldCap是16，二进制表示是10000，hash值的后五位和oldCap相与，因为oldCap的最高位（从右往左数的第5位）是1其余位是0，因此hash值的该位是0的所有元素被分到一条链表，挂到新数组中原下标处，hash值该位为1的被分到另外一条链表，挂到新数组中原下标+oldCap处。举个例子：桶0中的元素其hash值后五位是0XXXX的就被分到桶0种，其hash值后五位是1XXXX就被分到桶4中。

### ```HashTable```通过synchronized实现线程安全

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

### ```ConcurrentHashMap```

ConcurrentHashMap采用了非常精妙的"分段锁"策略，ConcurrentHashMap的主干是个Segment数组。

```java
 final Segment<K,V>[] segments;
```

　　Segment继承了```ReentrantLock```，所以它就是一种可重入锁。在```ConcurrentHashMap```，一个Segment就是一个子哈希表，Segment里维护了一个```HashEntry```数组，并发环境下，对于不同Segment的数据进行操作是不用考虑锁竞争的。（就按默认的```ConcurrentLevel```为16来讲，理论上就允许16个线程并发执行）

**所以，对于同一个Segment的操作才需考虑线程同步，不同的Segment则无需考虑。**　　 

## 7、transient关键词使用

Java语言的关键字，变量修饰符，如果用**transient**声明一个实例变量，当对象存储时，它的值不需要维持。换句话来说就是，用**transient**关键字标记的成员变量不参与序列化过程。

Java的**serialization提供了一种持久化对象实例的机制**。当持久化对象时，有一个特殊的对象数据成员，不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。当一个对象被序列化的时候，transient型变量的值不包括在序列化的表示中，然而非transient型的变量是被包括进去的。

## 8、Java中的错误和异常？

Java中的所有异常都是Throwable的子类对象，Error类和Exception类是Throwable类的两个直接子类。

Error：包括一些严重的、程序不能处理的系统错误类。这些错误一般不是程序造成的，比如StackOverflowError和OutOfMemoryError。

Exception：异常分为运行时异常和检查型异常。

- 检查型异常要求必须对异常进行处理，要么往上抛，要么try-catch捕获，不然不能通过编译。这类异常比较常见的是IOException。
- 运行时异常，可处理可不处理，在编译时可以通过，异常在运行时才暴露。比如数组下标越界，除0异常等。

## 9、Java的集合类框架介绍一下？

首先接口Collection和Map是平级的，Map没有实现Collection。

Map的实现类常见有HashMap、TreeMap、LinkedHashMap和HashTable等。其中HashMap使用散列法实现，低层是数组，采用链地址法解决哈希冲突，每个数组的下标都是一条链表，当长度超过8时，转换成红黑树。TreeMap使用红黑树实现，可以按照键进行排序。LinkedHashMap的实现综合了HashMap和双向链表，可保证以插入时的顺序（或访问顺序，LRU的实现）进行迭代。HashTable和HashMap比，前者是线程安全的，后者不是线程安全的。HashTable的键或者值不允许null，HashMap允许。

Collection的实现类常见的有List、Set和Queue。List的实现类有ArrayList和LinkedList以及Vector等，ArrayList就是一个可扩容的对象数组，LinkedList是一个双向链表。Vector是线程安全的（ArrayList不是线程安全的）。Set里的元素不可重复，实现类常见的有HashSet、TreeSet、LinkedHashSet等，HashSet的实现基于HashMap，实际上就是HashMap中的Key，同样TreeSet低层由TreeMap实现，LinkedHashSet低层由LinkedHashMap实现。Queue的实现类有LinkedList，可以用作栈、队列和双向队列，另外还有PriorityQueue是基于堆的优先队列。

## 10、Java反射是什么？为什么要用反射，有什么好处，哪些地方用到了反射？

反射：允许任意一个类在运行时获取自身的类信息，并且可以操作这个类的方法和属性。这种动态获取类信息和动态调用对象方法的功能称为Java的反射机制。

反射的核心是JVM在运行时才动态加载类或调用方法/访问属性。它不需要事先（写代码的时候或编译期）知道运行对象是谁，如`Class.ForName()`根本就没有指定某个特定的类，完全由你传入的类全限定名决定，而通过new的方式你是知道运行时对象是哪个类的。 反射避免了将程序“写死”。

反射可以降低程序耦合性，提高程序的灵活性。new是造成紧耦合的一大原因。比如下面的工厂方法中，根据水果类型决定返回哪一个类。

```java
public class FruitFactory {
    public Fruit getFruit(String type) {
        Fruit fruit = null;
        if ("Apple".equals(type)) {
            fruit = new Apple();
        } else if ("Banana".equals(type)) {
            fruit = new Banana();
        } else if ("Orange".equals(type)) {
            fruit = new Orange();
        }
        return fruit;
    }
}

class Fruit {}
class Banana extends Fruit {}
class Orange extends Fruit {}
class Apple extends Fruit {}
```

但是我们事先并不知道之后会有哪些类，比如新增了Mango，就需要在if-else中新增；如果以后不需要Banana了就需要从if-else中删除。这就是说只要子类变动了，我们必须在工厂类进行修改，然后再编译。如果用反射呢？

```java
public class FruitFactory {
    public Fruit getFruit(String type) {
        Fruit fruit = null;
        try {
            fruit = (Fruit) Class.forName(type).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return fruit;
    }
}

class Fruit {}
class Banana extends Fruit {}
class Orange extends Fruit {}
class Apple extends Fruit {}
```

如果再将子类的全限定名存放在配置文件中。

```properties
class-type=com.fruit.Apple
```

那么不管新增多少子类，根据不同的场景只需修改文件就好了，上面的代码无需修改代码、重新编译，就能正确运行。

哪些地方用到了反射？举几个例子

- 加载数据库驱动时
- Spring的IOC容器，根据XML配置文件中的类全限定名动态加载类
- 工厂方法模式中（如上）

### 反射的使用

##### 获取class实例的方式

1. 直接通过一个`class`的静态变量`class`获取：

   ```java
   Class cls = String.class;
   ```

2. 如果我们有一个实例变量，可以通过该实例变量提供的`getClass()`方法获取：

   ```java
   String s = "Hello";
   Class cls = s.getClass();
   ```

3. 如果知道一个`class`的完整类名，可以通过静态方法`Class.forName()`获取：

   ```java
   Class cls = Class.forName("java.lang.String");
   ```

因为`Class`实例在JVM中是唯一的。所以获取的String实例是相同的

```java
Class a = String.class;
Class b = "Hello".getClass();
a == b; //true
```

##### == 和instanceof的区别

 用`instanceof`不但匹配当前类型，还匹配当前类型的子类。而用`==`判断`class`实例可以精确地判断数据类型，但不能作子类型比较。 也就是如果该类是子类```instanceof```也返回true，但是==返回false

```java
Integer n = new Integer(123);

boolean b3 = n instanceof Integer; // true
boolean b4 = n instanceof Number; // true

boolean b1 = n.getClass() == Integer.class; // true
boolean b2 = n.getClass() == Number.class; // false
```

 注意到数组（例如`String[]`）也是一种`Class`，而且不同于`String.class`，它的类名是`[Ljava.lang.String`。此外，JVM为每一种基本类型如int也创建了`Class`，通过`int.class`访问。 

 获取到了一个`Class`实例，我们就可以通过该`Class`实例来创建对应类型的实例。

##### JVM动态加载

JVM在执行Java程序的时候，并不是一次性把所有用到的class全部加载到内存，而是第一次需要用到class时才加载。

##### 访问字段

通过`Class`实例获取字段信息。`Class`类提供了以下几个方法来获取字段：

- Field getField(name)：根据字段名获取某个public的field（包括父类）
- Field getDeclaredField(name)：根据字段名获取当前类的某个field（不包括父类）
- Field[] getFields()：获取所有public的field（包括父类）
- Field[] getDeclaredFields()：获取当前类的所有field（不包括父类

一个`Field`对象包含了一个字段的所有信息：

- `getName()`：返回字段名称，例如，`"name"`；
- `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
- `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

 先获取`Class`实例，再获取`Field`实例，然后，用`Field.get(Object)`获取指定实例的指定字段的值。 

 通过`Field.set(Object, Object)`实现修改字段的值，其中第一个`Object`参数是指定的实例，第二个`Object`参数是待修改的值。 

```java
    public static void main(String[] args) throws Exception {
        Student s = new Student(15,"djj",96);
        Student s1 = new Student(16,"why",97);
        Class cls = s.getClass();
        Field f = cls.getDeclaredField("score");
        System.out.println(f.get(s)); //96
        System.out.println(f.get(s1)); //97
        f.set(s, 56);
        System.out.println(s.getScore());//56
    }
```

##### 调用方法

通过`Class`实例获取所有`Method`信息。`Class`类提供了以下几个方法来获取`Method`：

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）



一个`Method`对象包含一个方法的所有信息：

- `getName()`：返回方法名称，例如：`"getScore"`；
- `getReturnType()`：返回方法返回值类型，也是一个Class实例，例如：`String.class`；
- `getParameterTypes()`：返回方法的参数类型，是一个Class数组，例如：`{String.class, int.class}`；
- `getModifiers()`：返回方法的修饰符，它是一个`int`，不同的bit表示不同的含义。调用非public方法

###### 调用静态方法

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 获取Integer.parseInt(String)方法，参数为String:
        Method m = Integer.class.getMethod("parseInt", String.class);
        // 调用该静态方法并获取结果:
        Integer n = (Integer) m.invoke(null, "12345");
        // 打印调用结果:
        System.out.println(n);
    }
}

int *a;

a = new int[45];

delete a;
```



###### 调用非静态方法

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // String对象:
        String s = "Hello world";
        // 获取String substring(int)方法，参数为int:
        Method m = String.class.getMethod("substring", int.class);
        // 在s对象上调用该方法并获取结果:
        String r = (String) m.invoke(s, 6);
        // 打印调用结果:
        System.out.println(r);
    }
}
```

###### 调用非public方法

和Field类似，对于非public方法，我们虽然可以通过`Class.getDeclaredMethod()`获取该方法实例，但直接对其调用将得到一个`IllegalAccessException`。为了调用非public方法，我们通过`Method.setAccessible(true)`允许其调用：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Person p = new Person();
        Method m = p.getClass().getDeclaredMethod("setName", String.class);
        m.setAccessible(true);
        m.invoke(p, "Bob");
        System.out.println(p.name);
    }
}
```

##### 构造实例

```java
Person p = Person.class.newInstance();
```

 局限：它只能调用该类的public无参数构造方法。如果构造方法带有参数，或者不是public，就无法直接通过```Class.newInstance()```来调用。 

 Java的反射API提供了Constructor对象，它包含一个构造方法的所有信息，可以创建一个实例。Constructor对象和Method非常类似，不同之处仅在于它是一个构造方法，并且，调用结果总是返回实例 

通过Class实例获取Constructor的方法如下：

- `getConstructor(Class...)`：获取某个`public`的`Constructor`；
- `getDeclaredConstructor(Class...)`：获取某个`Constructor`；
- `getConstructors()`：获取所有`public`的`Constructor`；
- `getDeclaredConstructors()`：获取所有`Constructor`。

`Constructor`总是当前类定义的构造方法，和父类无关，因此不存在多态的问题。

调用非`public`的`Constructor`时，必须首先通过`setAccessible(true)`设置允许访问。`setAccessible(true)`可能会失败。

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 获取构造方法Integer(int):
        Constructor cons1 = Integer.class.getConstructor(int.class);
        // 调用构造方法:
        Integer n1 = (Integer) cons1.newInstance(123);
        System.out.println(n1);

        // 获取构造方法Integer(String)
        Constructor cons2 = Integer.class.getConstructor(String.class);
        Integer n2 = (Integer) cons2.newInstance("456");
        System.out.println(n2);
    }
}
```

##### 获取父类，实现接口

通过`Class`对象可以获取继承关系：

- `Class getSuperclass()`：获取父类类型；
- `Class[] getInterfaces()`：获取当前类实现的所有接口。

通过`Class`对象的`isAssignableFrom()`方法可以判断一个向上转型是否可以实现。

## 11、说说你对面向对象、封装、继承、多态的理解？

- 封装：隐藏实现细节，明确标识出允许外部使用的所有成员函数和数据项。 防止代码或数据被破坏。
- 继承：子类继承父类，拥有父类的所有功能，并且可以在父类基础上进行扩展。实现了代码重用。子类和父类是兼容的，外部调用者无需关注两者的区别。
- 多态：一个接口有多个子类或实现类，在运行期间（而非编译期间）才决定所引用的对象的实际类型，再根据其实际的类型调用其对应的方法，也就是“动态绑定”。

Java实现多态有三个必要条件**：继承、重写、向上转型。**

- 继承：子类继承或者实行父类
- 重写：在子类里面重写从父类继承下来的方法
- 向上转型：父类引用指向子类对象

```java
public class OOP {
    public static void main(String[] args) {
        /*
         * 1. Cat继承了Animal
         * 2. Cat重写了Animal的eat方法
         * 3. 父类Animal的引用指向了子类Cat。
         * 在编译期间其静态类型为Animal;在运行期间其实际类型为Cat，因此animal.eat()将选择Cat的eat方法而不是其他子类的eat方法
         */
        Animal animal = new Cat();
        printEating(animal);
    }

    public static void printEating(Animal animal) {
        animal.eat();
    }
}

abstract class Animal {
    abstract void eat();
}
class Cat extends Animal {
    @Override
    void eat() {
        System.out.println("Cat eating...");
    }
}
class Dog extends Animal {
    @Override
    void eat() {
        System.out.println("Dog eating...");
    }
}
```

## 12、实现不可变对象的策略？比如JDK中的String类。

- 不提供setter方法（包括修改字段、字段引用到的的对象等方法）
- 将所有字段设置为final、private
- 将类修饰为final，不允许子类继承、重写方法。可以将构造函数设为private，通过工厂方法创建。
- 如果类的字段是对可变对象的引用，不允许修改被引用对象。 1）不提供修改可变对象的方法；2）不共享对可变对象的引用。对于外部传入的可变对象，不保存该引用。如要保存可以保存其复制后的副本；对于内部可变对象，不要返回对象本身，而是返回其复制后的副本。

## 13、Java序列话中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用**transient**关键字修饰。功能是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法。

## 14、==和equals的区别？

== 对于基本类型，比较值是否相等，对于对象，比较的是两个对象的地址是否相同，即是否是指相同一个对象。

equals的默认实现实际上使用了==来比较两个对象是否相等，但是像Integer、String这些类对equals方法进行了重写，比较的是两个对象的内容是否相等。

对于Integer，如果依然坚持使用==来比较，有一些要注意的地方。对于[-128,127]区间里的数，有一个缓存。因此

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true

Integer a = 128;
Integer b = 128;
System.out.println(a == b); // false

// 不过采用new的方式，a在堆中，这里打印false
Integer a = new Integer(127);
Integer b = 127;
System.out.println(a == b);
```

对于String，因为它有一个常量池。所以

```java
String a = "gg" + "rr";
String b = "ggrr";
System.out.println(a == b); // true

// 当然牵涉到new的话，该对象就在堆上创建了，所以这里打印false
String a = "gg" + "rr";
String b = new String("ggrr");
System.out.println(a == b);
```

## 15、接口和抽象类的区别？

- Java不能多继承，一个类只能继承一个抽象类；但是可以实现多个接口。
- 继承抽象类是一种IS-A的关系，实现接口是一种LIKE-A的关系。
- 继承抽象类可以实现对父类代码的复用，也可以重写抽象方法实现子类特有的功能。实现接口可以为类新增额外的功能。
- 抽象类定义基本的共性内容，接口是定义额外的功能。
- 调用者使用动机不同, 实现接口是为了使用他规范的某一个行为；继承抽象类是为了使用这个类属性和行为.

## 16、给你一个Person对象p，如何将该对象变成JSON表示？

本质是考察Java反射，因为要实现一个通用的程序。实现可能根本不知道该类有哪些字段，所以不能通过get和set等方法来获取键-值。使用反射的getDeclaredFields()可以获得其声明的字段。如果字段是private的，需要调用该字段的`f.setAccessible(true);`，才能读取和修改该字段。

```java
import java.lang.reflect.Field;
import java.util.HashMap;

public class Object2Json {
    public static class Person {
        private int age;
        private String name;

        public Person(int age, String name) {
            this.age = age;
            this.name = name;
        }
    }

    public static void main(String[] args) throws IllegalAccessException {
        Person p = new Person(18, "Bob");
        Class<?> classPerson = p.getClass();
        Field[] fields = classPerson.getDeclaredFields();
        HashMap<String, String> map = new HashMap<>();
        for (Field f: fields) {
            // 对于private字段要先设置accessible为true
            f.setAccessible(true);
            map.put(String.valueOf(f.getName()), String.valueOf(f.get(p)));
        }
        System.out.println(map);
    }
}

```

得到了map，再弄成JSON标准格式就好了。

## 17、JDBC中sql查询的完整过程？操作事务呢？

```java
@Test
public void fun2() throws SQLException, ClassNotFoundException {
    // 1. 注册驱动
    Class.forName("com.mysql.jdbc.Driver");
    String url = "jdbc:mysql://localhost:3306/xxx?useUnicode=true&characterEncoding=utf-8";
    // 2.建立连接
    Connection connection = DriverManager.getConnection(url, "root", "admin");
    // 3. 执行sql语句使用的Statement或者PreparedStatment
    Statement statement = connection.createStatement();
    String sql = "select * from stu;";
    ResultSet resultSet = statement.executeQuery(sql);

    while (resultSet.next()) {
        // 第一列是id，所以从第二行开始
        String name = resultSet.getString(2); // 可以传入列的索引，1代表第一行，索引不是从0开始
        int age = resultSet.getInt(3);
        String gender = resultSet.getString(4);
        System.out.println("学生姓名：" + name + " | 年龄：" + age + " | 性别：" + gender);
    }
    // 关闭结果集
    resultSet.close();
    // 关闭statemenet
    statement.close();
    // 关闭连接
    connection.close();
}
```

**ResultSet维持一个指向当前行记录的cursor（游标）指针**

- 注册驱动
- 建立连接
- 准备sql语句
- 执行sql语句得到结果集
- 对结果集进行遍历
- 关闭结果集（ResultSet）
- 关闭statement
- 关闭连接（connection）

由于JDBC默认自动提交事务，每执行一个update ,delete或者insert的时候都会自动提交到数据库，无法回滚事务。所以若需要实现事务的回滚，要指定`setAutoCommit(false)`。

- `true`：sql命令的提交（commit）由驱动程序负责
- `false`：sql命令的提交由应用程序负责，程序必须调用commit或者rollback方法

JDBC操作事务的格式如下，在捕获异常中进行事务的回滚。

```java
try {
  con.setAutoCommit(false);//开启事务…
  ….
  …
  con.commit();//try的最后提交事务
} catch() {
  con.rollback();//回滚事务
}
```

## 18、实现单例，有哪些要注意的地方？

就普通的实现方法来看。

- 不允许在其他类中直接new出对象，故构造方法私有化
- 在本类中创建唯一一个static实例对象
- 定义一个public static方法，返回该实例

```java
public class SingletonImp {
    // 饿汉模式
    private static SingletonImp singletonImp = new SingletonImp();
    // 私有化（private）该类的构造函数
    private SingletonImp() {
    }

    public static SingletonImp getInstance() {
        return singletonImp;
    }
}
```

饿汉模式：线程安全，不能延迟加载。

```java
public class SingletonImp4 {
    private static volatile SingletonImp4 singletonImp4;

    private SingletonImp4() {}

    public static SingletonImp4 getInstance() {
        if (singletonImp4 == null) {
            synchronized (SingletonImp4.class) {
                if (singletonImp4 == null) {
                    singletonImp4 = new SingletonImp4();
                }
            }
        }

        return singletonImp4;
    }
}
```

双重检测锁+volatile禁止语义重排。因为`singletonImp4 = new SingletonImp4();`不是原子操作。

```java
public class SingletonImp6 {
    private SingletonImp6() {}

    // 专门用于创建Singleton的静态类
    private static class Nested {
        private static SingletonImp6 singletonImp6 = new SingletonImp6();
    }

    public static SingletonImp6 getInstance() {
        return Nested.singletonImp6;
    }
}
```

静态内部类，可以实现延迟加载。

最推荐的是单一元素枚举实现单例。

- 写法简单
- 枚举实例的创建默认就是线程安全的
- 提供了自由的序列化机制。面对复杂的序列或反射攻击，也能保证是单例

```java
public enum Singleton {
    INSTANCE;
    public void anyOtherMethod() {}
}
```

## 19、覆写Object方法

- `toString()`：把instance输出为`String`；
- `equals()`：判断两个instance是否逻辑相等；
- `hashCode()`：计算一个instance的哈希值。

 在子类的覆写方法中，如果要调用父类的被覆写的方法，可以通过`super`来调用 。

 继承可以允许子类覆写父类的方法。如果一个父类不允许子类对它的某个方法进行覆写，可以把该方法标记为`final`。用`final`修饰的方法不能被`Override` 。`final`修饰符有多种作用：

## 20、final关键词

- `final`修饰的方法可以阻止被覆写；
- `final`修饰的class可以阻止被继承；
- `final`修饰的field必须在创建对象时初始化，随后不可修改。

## 21、抽象类和接口

##### default方法

|            | abstract class       | interface                   |
| :--------- | :------------------- | :-------------------------- |
| 继承       | 只能extends一个class | 可以implements多个interface |
| 字段       | 可以定义实例字段     | 不能定义实例字段            |
| 抽象方法   | 可以定义抽象方法     | 可以定义抽象方法            |
| 非抽象方法 | 可以定义非抽象方法   | 可以定义default方法         |

实现类可以不必覆写`default`方法。`default`方法的目的是，当我们需要给接口新增一个方法时，会涉及到修改全部子类。如果新增的是`default`方法，那么子类就不必全部修改，只需要在需要覆写的地方去覆写新增方法。

`default`方法和抽象类的普通方法是有所不同的。因为`interface`没有字段，`default`方法无法访问字段，而抽象类的普通方法可以访问实例字段。

#####  interface字段

因为`interface`是一个纯抽象类，所以它不能定义实例字段。但是，`interface`是可以有静态字段的，并且静态字段必须为`final`类型： 

```java
public interface Person {
    public static final int MALE = 1;
    public static final int FEMALE = 2;
}

//因为interface的字段只能是public static final类型，所以我们可以把这些修饰符都去掉，上述代码可以简写为：
public interface Person {
    // 编译器会自动加上public statc final:
    int MALE = 1;
    int FEMALE = 2;
}
```

## 22、作用域

##### package包作用域

最后，包作用域是指一个类允许访问同一个`package`的没有`public`、`private`修饰的`class`，以及没有`public`、`protected`、`private`修饰的字段和方法。

`public`、`protected`、`private`略

##### 局部变量

在方法内部定义的变量称为局部变量，局部变量作用域从变量声明处开始到对应的块结束。方法参数也是局部变量。

## 23、异常和错误

Error和RuntimeException是非受查异常，其他的异常为受查异常。

在程序中无须将非受查异常进行catch或者throws。

在测试阶段可以使用断言来进行。

##### 常见的RuntimeException异常

NullPointerException - 空指针引用异常
ClassCastException - 类型强制转换异常。
IllegalArgumentException - 传递非法参数异常。
ArithmeticException - 算术运算异常
ArrayStoreException - 向数组中存放与声明类型不兼容对象异常
IndexOutOfBoundsException - 下标越界异常

##### 常见的CheckedException异常

SQLException
OException
ClassNotFoundException
NamingException,
ServletException,

## 24、迭代器

迭代器就是提供一种方法对一个容器对象中的各个元素进行访问，而又不暴露该对象容器的内部细节。

##### ```Iterator```接口的实现如下

1. 迭代器在迭代期间可以从集合中移除元素。

 2. 方法名得到了改进，Enumeration的方法名称都比较长。

```java
package java.util;

import java.util.function.Consumer;
public interface Iterator<E> {
    /**
     * Returns {@code true} if the iteration has more elements.
     */
    boolean hasNext();

    /**
     * Returns the next element in the iteration.
     */
    E next();
    
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

##### ```Iterable```

```java
package java.lang;

import java.util.Iterator;
import java.util.Objects;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.function.Consumer;
public interface Iterable<T> {
    //返回一个Iterator对象
    Iterator<T> iterator();
    
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
    
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}

```

```Iterable```接口包含一个能产生```Iterator```对象的方法，并且```Iterator```被```foreach```用来在序列中移动。因此如果创建了实现```Iterable```接口的类，都可以将它用于```foreach```中。 

###### 两种遍历方式

使用迭代器遍历和使用```foreach```遍历

```java
		LinkedList<String> linkedList = new LinkedList<>();
        linkedList.push("first");
        linkedList.push("second");
        linkedList.push("third");
        linkedList.push("forth");
        linkedList.push("fifth");
        Iterator<String> linkedListIterable = linkedList.iterator();

        while(linkedListIterable.hasNext()){
            System.out.println(linkedListIterable.next());
        }

        for (String string:linkedList) {
            System.out.println(string);
        }
```

 在使用Iterator的时候禁止对所遍历的容器进行改变其大小结构的操作。例如: 在使用Iterator进行迭代时，如果对集合进行了add、remove操作就会出现```ConcurrentModificationException```异常。 

##### ```ListIterator```

```java
package java.util;

public interface ListIterator<E> extends Iterator<E> {
    
    boolean hasNext();
    
    E next();

    boolean hasPrevious();

    E previous();

    int nextIndex();

    int previousIndex();
    
    void set(E e);

    void add(E e);
}
```
