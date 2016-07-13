---
layout: post
date: 2016-06-15
title: 算法
excerpt: 算法与数据结构
java: true
comments: true
tag: [算法]
---

# 第一章：基础

## 1.3 背包、队列和栈

> 背包：Bag  
> 队列：Queue 先进先出  
> 栈：Stack 后进先出  


### 1.3.1 API

#### 背包
背包是一种不支持从中进行元素删除的集合数据类型，其存在意义是收集元素并对其进行遍历，**无序**迭代。

#### 队列
使用队列的主要原因是在用集合保存元素的同时保存他们的相对顺序

#### 下压栈（栈）
比如邮件系统、浏览网页。push/pop。
利用栈来计算算是表达式：

1. 将操作数压入操作数栈
2. 将运算符压入运算符栈
3. 忽略左括号
4. 遇到右括号时，弹出一个运算符，弹出所需数量的操作数，并将运算符和操作数的运算结果压入操作数栈。

~~~java
// Dijkstra’s 双栈算术表达式求值算法
~~~

### 1.3.2 集合类数据类型的实现

#### 定容栈

~~~java
// 含有范型的定容栈
public class FixedCapacityStackOfStrings<Item> {
    private Item[] a; // stack entries
    private int N; // size

    public FixedCapacityStackOfStrings(int cap) {
        a = (Item[]) new Object[cap];
    }

    public boolean isEmpty() {
        return N == 0;
    }

    public int size() {
        return N;
    }

    public void push(Item item) {
        a[N++] = item;
    }

    public Item pop() {
        return a[--N];
    }
}
~~~

#### 调整数组大小

~~~java
private void resize(int max)
{  // Move stack of size N <= max to a new array of size max.
   Item[] temp = (Item[]) new Object[max];
   for (int i = 0; i < N; i++)
      temp[i] = a[i];
   a = temp;
}

public void push(String item)
{  // Add item to top of stack.
   if (N == a.length) resize(2*a.length);
   a[N++] = item;
}

public String pop()
{  // Remove item from top of stack.
   String item = a[--N];
   a[N] = null;  // Avoid loitering (see text).
   if (N > 0 && N == a.length/4) resize(a.length/2);
   return item;
}
~~~

> 避免对象游离——将其引用设置为null。  
> Queue的实现方法，使用head和tail索引。

#### 迭代
实现一个可以迭代的类：

1. 实现 Iterable<Item> 接口
2. 实现接口内的 iterator() 方法，返回一个 Iterator<Item> 类型的对象。所以，必须
3. 实现 Iterator<Item> 接口，构建自己的迭代器

练习：实现一个可以动态调整数组大小的栈：ResizingArrayStack

~~~java
package bag_Queue_Stack_1_3;

import java.util.Iterator;

public class ResizingArrayStack<Item> implements Iterable<Item> {

    private Item[] a = (Item[]) new Object[1];
    private int N = 0;

    public boolean isEmpty() {
        return N == 0;
    }

    public int size() {
        return N;
    }

    public void resize(int max) {
        Item[] temp = (Item[]) new Object[max];
        for (int i = 0; i < N; i++) {
            temp[i] = a[i];
        }
        a = temp;
    }

    public void push(Item item) {
        if (N == a.length)
            resize(2 * a.length);
        a[N++] = item;
    }

    public Item pop() {
        Item item = a[--N];
        a[N] = null;
        if (N > 0 && N == a.length / 4)
            resize(a.length / 2);
        return item;
    }

    @Override
    public Iterator<Item> iterator() {
        return new ReverseArrayIterator();
    }

    private class ReverseArrayIterator implements Iterator<Item> {

        private int i = N;

        @Override
        public boolean hasNext() {
            return i > 0;
        }

        @Override
        public Item next() {
            return a[--i];
        }
    }

}
~~~

### 1.3.3 链表 *（重点掌握）*

> 定义：*链表*是一种递归的数据结构，它或者为空，或者是指向另一个节点的引用，该节点含有一个范型元素和一个指向另一个链表的引用

#### 构造链表

~~~java
Private class Node {
    Item item;
    Node next;
}
~~~

#### 链表需要实现的一些功能：

1. 从表头插入／删除节点
2. 从表尾插入／删除节点
3. 从任意位置插入／删除节点——双向链表
4. 遍历

~~~java
for(Node x = first; x != null; x = x.next()) {
    x.item;
}
~~~

#### 算法与数据结构的结合：

* 栈的实现
从头插入，从头拿出来

~~~java
public class Stack<Item> {
    private Node first; // 栈顶（最新添加的元素）
    private int N; // quantity of elements

    private class Node {
        // difine a inner class of node
        Item item;
        Node next;
    }

    public void push(Item item) {
        // add elements from the stack top
        Node oldFirst = first;
        first = new Node();
        first.item = item;
        first.next = oldFirst;
        N++;
    }

    public Item pop() {
        // delete element from stack top
        Item item = first.item;
        first = first.next; // important
        N--;
        return item;
    }
}
~~~

* 队列的实现
从屁股插入，从头拿出来

~~~java
public class Queue<Item> {
    private Node first;
    private Node last;
    private int N = 0;
    
    private class Node {
        Item item;
        Node next;
    }
    
    public boolean isEmpty() {
        return N == 0;
    }
    
    public int size() {
        return N;
    }
    
    public void enqueue(Item item) {
        // 向屁股插入一个元素
        Node oldLast = last;
        last = new Node();
        last.item = item;
        last.next = null;
        if(isEmpty()) {
            first = last;
        } else 
            oldLast.next = last;
        N++;
    }
    
    public Item dequeue() {
        // 从头拿出一个元素
        Item item = first.item;
        first = first.next;
        if(isEmpty()) {
            last = null;
        }
        N--;
        return item;
    }
}
~~~

* 背包的实现
重点是在集合数据类型中实现迭代

~~~java
public class Bag<Item> implements Iterable<Item> {
    private Node first;
    private class Node {
        Item item;
        Node next;
    }

    ...

    public Iterator<Item> iterator() {
        return new ListIterator();
    }

    private class ListIterator implements Iterator<Item> {
        private Node current = first;
        public boolean hasNext() {
            return current != null;
        }

        public Item next() {
            Item item = current.item;
            current = current.next();
            return item;
        }
    }
}

~~~

### 1.3.4 总结

两种基本的数据结构：数组和链表->顺序存储和链式存储

数组：可以通过索引直接访问任意元素，但是需要在初始化时知道元素的数量。

链表：使用的空间大小和元素数量成正比，但是需要通过引用访问任意元素。

练习：

~~~
//实现一个打印N的二进制表示：
Stack<Integer> stack = new Stack<integer>()
while(N > 0) {
    stack.push(N % 2);
    N = N / 2;
}

for(int d : stack) {
    System.out.println(d);
}
~~~

## 1.4 算法分析

两个问题：

* 我的程序会运行多长时间
* 为什么我的程序耗尽了所有内存  

幂次法则

### 1.4.3 数学模型

一个程序运行的总时间主要和两点有关：
1. 执行每条语句的耗时
2. 执行每条语句的频率

涉及两个概念：首项近似／增长的数量级

\\[
	f(N^b)(logN)^c
\\]

总结：对于大多数程序，得到其运行时间的数学模型所需的步骤如下：

1. 确定输入模型，定义问题规模
2. 识别内循环
3. 根据内循环中的操作确定成本模型
4. 对于给定的输入，判断这些操作的执行频率

### 1.4.4 增长数量级的分类

* 常数级别——普通语句
* 对数级别——二分查找
* 线性级别——循环
* 线性对数级别——归并排序
* 平方级别——双层循环
* 立方级别——三层循环
* 指数级别——穷举

## 1.5 Union-Find 算法

### 1.5.1 关于动态连通性

### 1.5.2 实现

* quick-find 算法

~~~java
//id[i]为触点i所对应的点连通分量的值
public int find(int p) {
    return id[p];
}

public void union(int p, int q) {
    int pId = find(p);
    int qId = find(q);

    if(qId == pId) return;

    for(int i=0; i<id.length; i++) {
        if(id[i] == pId) id[i] = qId;
        count--;
    }
}
~~~

* quick-union 算法：森林的表示－树结构。每个节点对应的值是另一个节点的名称。

~~~java
// id[i] 表示节点i所链接的另一个节点
public int find(int p) {
    while(p != id[p]) p = id[p];
    return p;
}

public void union(int p, int q) {
    int pRoot = find(p);
    int qRoot = find(q);
    if(pRoot == qRoot) return;

    id[pRoot] = qRoot;
    count--;
}
~~~
> 树：树的大小是其中节点数量，树的一个节点的*深度*是它到根节点的路径上的链接数。树的高度是它的所有节点中的最大深度。

该方法比较依赖于输入的数据结构，在某些情况下，树的深度可能非常大。

* 加权 quick-union算法：

    对上一算法的改良，不再随意将一棵树与另一棵树链接，记录每棵树的大小，并总是将较小的树连接到较大的树上。—— 加权，从而减小了树的深度。

~~~java
public class WeightQuickUnionUF {
    private int count;
    private int[] id;
    private int[] sz;

    public WeightQuickUnionUF(int n) {
        count = n;
        id = new int[n];
        sz = new int[n];

        for(int i=0; i<n; i++) {
            id[i] = i;
            sz[i] = i;
        }
    }

    public int count() {
        return count;
    }

    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    public int find(int p) {
        while(p != id[p]) p = id[p];
        return p;
    }

    public union(int p, int q) {
        int qRoot = find(q);
        int pRoot = find(p);

        if(qRoot == pRoot) return;
        // 将小树的根节点连接到大树上
        if(sz[qRoot] < sz[pRoot]) {
            id[qRoot] = pRoot;
            sz[pRoot] += sz[qRoot];
        } else {
            id[pRoot] = qRoot;
            sz[qRoot] += pRoot;
        }
        count--;
    }
}
~~~

* 最优算法：路径压缩的加权 quick-union 算法，将每一个节点都直接链接到根节点上。

# 第二章 排序

> 经典、优雅和高效（选择排序、插入排序、希尔排序、归并排序、快速排序、堆排序）

## 2.1 初级排序算法

~~~java
//排序类算法模板
public static boolean less(Comparable a, Comparable b) {
    return a.compareTo(b) < 0;
}

public static void exch(Comparable[] a, int i, int j) {
    Comparable t = a[i]; 
    a[i] = a[j];
    a[j] = t;
}
~~~

### 2.1.2 选择排序

> 首先找到数组中最小的那个元素，然后，将他和数组中的第一个元素交换。再次，在剩下的元素中找到最小的元素，将它与数组的第二个元素交换位置，如此往复，直到整个数组排序。

两个特点：

* 运行时间和输入无关
* 数据移动是最少的 

~~~java

public static void sort(Comaprable[] a) {
    int n = a.length;
    for(int i=0; i<n; i++) {
        int min = i;
        for(int j = i+1; j<n; j++) {
            if(less(a[j], a[min])) min = j;
        }
        exch(a, i, min);
    }
}
~~~

最后，比较排序需要 N 次交换和（n-1) + （n-2) + ... + 2 + 1次比较：n*(n-1)/2

### 2.1.3 插入排序

> 类似整理桥牌，需要一张一张的来，将每一张牌牌插入到其他已经有序排列的牌中的适当位置。但是在计算机的实现中，为了给要插入的元素腾出空间，*还需要将其余所有元素在插入之前都向右移动一位。*

~~~java
public static void sort(Comparable[] a) {
    int n = a.length;
    for(int i=1; i<n; i++) {
        for(int j=i; j>0 && less(a[j] < a[j-1]); j--) {
            exch(a, j, j-1);
        }
    }
}
~~~


### 2.1.6 希尔排序

> 基于插入排序的快速的排序算法，原理是交换不相邻的元素以对数组的局部进行排序（形成 h有序数组），并最终用插入排序将局部有序的数组排序。

~~~java
public static void sort(Comparable a) {
    int n = a.length;
    int h = 1;
    while(h < n/3) {
        // 找到最大步长
        h += 3*h + 1;
    }
    while(h >= 1) {
        for(int i=h; i<n; i++) {
            for(int j=i; j>=h && less(a[j], a[j-h]; j-= h)) {
                exch(a, j, j-h);
            }
        }
        h = h / 3;
    }
}
~~~

[WIKI 解释](https://zh.wikipedia.org/wiki/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F)

例如，假设有这样一组数[ 13 14 94 33 82 25 59 94 65 23 45 27 73 25 39 10 ]，如果我们以步长为5开始进行排序，我们可以通过将这列表放在有5列的表中来更好地描述算法，这样他们就应该看起来是这样：

~~~
13 14 94 33 82
25 59 94 65 23
45 27 73 25 39
10
~~~

然后我们对每列进行排序：

~~~
10 14 73 25 23
13 27 94 33 39
25 59 94 65 82
45
~~~

将上述四行数字，依序接在一起时我们得到：[ 10 14 73 25 23 13 27 94 33 39 25 59 94 65 82 45 ].这时10已经移至正确位置了，然后再以3为步长进行排序：

~~~
10 14 73
25 23 13
27 94 33
39 25 59
94 65 82
45
~~~

排序之后变为：

~~~
10 14 13
25 23 33
27 25 59
39 65 73
45 94 82
94
~~~

最后以1步长进行排序（此时就是简单的插入排序了）。


## 2.2 归并排序

所谓归并，既将两个有序数组归并为一个更大的有序数组。

### 2.2.1 原地归并的抽象方法

~~~java
public static void merge(int[] a, int lo, int mid, int hi){
    int i = lo;
    int j = mid + 1;

    for(int k=lo; k<=hi; k++) {
        aux[k] = a[k]; // copy a[lo..hi] to aut[lo..hi]
    }
    
    for(int k=lo; k<=hi; k++) {
        if(i > mid) { 
            a[k] = aux[j++];
        }
        else if (j > hi) {
            a[k] = aux[i++];
        }
        else if(less(aux[j], aux[i])) {
            a[k] = a[j++];
        }
        else {
            a[k] = a[i++];
        }
    }
}
~~~

。

> 归并排序，其的基本思路就是将数组分成二组A，B，如果这二组组内的数据都是有序的，那么就可以很方便的将这二组数据进行排序。如何让这二组组内数据有序了？
可以将A，B组各自再分成二组。依次类推，当分出来的小组只有一个数据时，可以认为这个小组组内已经达到了有序，然后再合并相邻的二个小组就可以了。这样通过先递归的分解数列，再合并数列就完成了归并排序


### 2.2.2 自顶向下的归并排序

> 以下代码归纳证明算法能够正确地将数组排序的基础：如果它能够将两个子数组排序，它就能通过归并两个子数组将整个数组排序。

排序所需时间与 NlgN 成正比（N 为数组长度）

~~~java
public class Merge {
    private static Comparable[] aux;

    public static void sort(Comparable[] a) {
        aux = new Comparable[a.length];
        sort(a, 0, a.length-1);
    }

//sort 方法的作用实际在于安排多次 merge 方法的调用的正确顺序
    private static void sort(Comparable[] a, int lo, int hi) {
        if(hi <= lo) return;
        int mid = lo + (hi - lo) / 2;
        sort(a, lo, mid); // divide the left
        sort(a, mid+1, hi) // divide the right
        merge(a, lo, ,mid, hi) // merge
    }
}

// 
~~~

改进方法：

1. 对于小规模子数组使用插入排序
2. 另外可以添加一个判断条件：if(a[mid] < a[mid+1])，如果为真，就认为数组是有序的并跳过 merge（）方法。
3.不将元素复制到辅助数组 


> 但在整体上，我们希望学习的是为每种应用找打最合适的算法。我们并不推荐你一定要实现所提到的这些改进方法，而是提醒大家不要对算法的初始实现的性能盖棺定论。研究一个新问题时，最好的方法是*先实现一个你能先到的最简单的程序，当它成为瓶颈的时候再继续改进它。*

### 2.2.3 自底向上的归并排序

~~~java
// 注意最后 N-1的使用。只有当最后一个子数组的大小只有在数组大小是 sz 的偶数倍时才会等于sz。__
// 另外，同2.2.2中类似，最好把 aux 在另一个 sort 方法中进行声明，避免每次归并都重新创建对象，此处进行了
public class MergeBU {
    private static Comparable[] aux;

    public static void sort(Comparable[] a) {
        int N = a.length;
        aux = new Comparable[N]();
        for(int sz=1; sz<N; sz = sz+sz) {
            for(int lo=0; lo<N-sz; lo += sz+sz) {
                merge(a, lo, lo+sz-1, Math.min(lo+sz+sz-1, N-1)) 
            }
        }
    }
}
~~~


### 2.2.4 排序算法的复杂度

>没有任何基于比较的算法能够保证使用少于 lg(N!) ~ NlgN 次比较将长度为 N的数组排序

如果用二叉树表示所有比较，假设有 N 个主键，高度为h 一棵树至少有N!个叶子节点，最多有2<sup>h</sup>个叶子节点

N! <= 叶子节点的数量 <= 2<sup>h</sup>

h 的值就是最坏情况下比较次数，所以两边取对数，得到结论。

## 2.3 快速排序

### 2.3.1 基本算法

于归并排序相似，不同之处在于：

* 归并排序是先进行递归调用，再处理数组，而快速排序是递归调用发生在处理整个数组之后 
* 归并排序将一个数组切分成两半，快速排序中，切分的位置取决于数组的内容

递归的调用切分来进行排序。

~~~java
// partition method
public int partition(Comparable[]a, int lo, int hi) {
    int i = lo;
    int j = hi + 1;
    Comparable v = a[lo]; // 随机选取一个切分值
    
    while(true) {
        // 从左向右扫描，找到左边大于 v 的值
        while(less(a[++i] < v)) if(i == hi) break;
        // 从右向左扫描，找到右边小于 v 的值
        while(less(a[--j] > v)) if(j == lo) break;
        // 如果两个指针相遇，结束循环,交换 j 与 lo 的位置
        if(i >= j) break; 
        // 否则交换 i j 元素的位置
        exch(a, i, j);
    }
    exch(a, lo, j);
    return j;
}

// sort method
public void sort(Comparable[] a) {
    // shuffle 数组
    StdRandom.shuffle(a);
    sort(a, 0, a.length-1);
}

public void sort(Comparable[] a, int lo, int hi) {
    if(lo >= hi) return;
    int j = partition(a, lo, hi);
    sort(a, lo, j-1);
    sort(a, j+1, hi);
}
~~~

### 2.3.2 算法改进

* 在排序到小数组时自动切换到插入排序。

~~~java
// 替换 if(lo >= hi)
// 5 < M < 15
if(hi <= lo + M) {
    Insertion.sort(a, lo, hi);
    return;
}
~~~

* 三取样切分

使用子数组的一小部分元素的中位数来切分数组。

* 熵最优的排序（三项切分的快速排序）

用于处理大量重复元素出现的情况：将数组切分为3部分，分别对应小于、等于、大于切分元素的数组元素。

~~~java

~~~

## 2.4 优先队列

### 2.4.3 堆的定义（二叉堆）

> 当一棵二叉树的每个节点都大于等于他的两个子节点，它被称为堆有序

二叉堆的表示法：

* 使用三个指针表示每个元素
* 完全二叉树：可以只使用数组进行实现，因为只需要将每个元素从上至下，从左至右按照层级顺序仿佛数组中即可。

> 二叉堆是一组能够用堆有序的完全二叉树排序的元素，并在数组中按照层及储存。（不使用数组的第一个位置）
>   
> 在一个堆中，位置 k 的节点的父节点的位置为[k/2]，而他的两个子节点的位置则分别为[2k]和[2k+1]

### 2.4.4 堆的算法

#### 2.4.4.1 由下至上的堆有序化（上浮）

某个节点变得比他的父节点更大，需要交换两个节点。不断重复，直至没有比该节点大的父节点出现。

~~~java
public void swim(int k) {
    while(k > 1 && less(k/2, k)) {
        exch(k/2, k);
        k = k/2;
    }
}
~~~

#### 2.4.4.2 由上至下的堆有序化（下沉）

当某个节点变得比他的两个子节点更小，或者其中之一更小时，交换它和它的两个子节点中较大的来恢复堆序。

~~~java
public void sink(int k) {
    while(2*k <= N) {
        int j = 2*k;
        if(j < N && less(j, j+1)) j++;
        if(!less(k, j)) break;
        exch(k, j);
        k = j;
    }
}
~~~

插入元素：将新元素添加到数组的末尾，增加堆的大小，并让这个新元素上浮至合适的位置：swim()  
删除最大元素：从数组顶端删除最大的元素，并将数组的最后一个元素放在顶端，让其下沉至合适的位置：sink()  

~~~java
public class MaxPQ<Key extends Comparable<Key>> {
    private Key[] pq;   // 基于堆的完全二叉树
    private int N = 0;      // 用于记录大小，注意 pq[0]没有使用

    public MaxPQ(int maxN) {
        pq = (Key[]) new Comparable[maxN+1];
    }

    public boolean isEmpty() {
        return N == 0;
    }

    public int size() {
        return N;
    }

    public void insert(Key k) {
        pq[++N] = k;
        swim(N);
    }

    public Key delMax() {
        Key max = pq[1];
        exch(1, N--);
        pq[N + 1] = null;
        sink(1);
        return max;
    }

    public boolean less(int i, int j) {...}
    public void exch(int i, int j) {...}
    public void swim(int k) {...}
    public void sink(int k) {...}
}
~~~

#### 2.4.4.6 索引优先队列


### 2.4.5 堆排序

分为两个阶段：堆的构造阶段，堆的下沉排序阶段。

~~~java
public void sort(Comaprabel[] a) {
    int N = a.length;
    //构造堆
    for(int k=N/2; k>=1; k--) {
        sink(a, k, N);
    }

    //下沉排序
    while(N > 1) {
        exch(a, 1, N--);
        sink(a, 1, N);
    }
}
~~~


# 3. 查找

