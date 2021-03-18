# 列表 List

---

## 1.接口与实现

### 1.1 从静态到动态

> * 根据是否修改数据结构，所有操作大致分为两类方式：
>> 1. 静态：仅读取，数据结构的内容及组成一般不变：get、search
>> 2. 动态：需写入，数据结构的局部或整体将改变：insert、remove
> * 与操作方式相对应地，数据元素的存储与组织方式也分为两种：
>> 1. 静态：
>>> * 数据空间整体创建或销毁
>>> * 数据元素的<u>物理存储次序</u>与其<u>逻辑次序</u>严格一致
>>> * 可支持高效的静态操作
>>> * 比如向量，元素的物理地址与其逻辑次序线性对应
>> 2. 动态：
>>> * 为各数据元素动态地分配和回收的物理空间
>>> * 逻辑上相邻的元素记录彼此的物理地址，在逻辑上形成一个整体
>>> * 可支持高效的动态操作

### 1.2 从向量到列表

> * 列表（List）是采用动态储存策略的典型结构
>> * 其中的元素称作节点（node）
>> * 各节点通过指针或引用彼此联接，在逻辑上构成一个线性序列：L = {a<sub>0</sub>, a<sub>1</sub>, ..., a<sub>n-1</sub>}
> * 相邻节点彼此互称前驱（predecessor）或后继（successor）
>> * 前驱或后继若存在，则必然唯一
>> * 没有前驱/后继的唯一节点称作首（first/front）/末（last/rear）节点

### 1.3 从秩到位置

> * 向量支持循秩访问（call-by-rank）的方式
> * 根据数据元素的秩，可在O(1)时间内直接确定其物理地址
> * 既然同属线性序列，列表固然也可通过秩来定位节点：
>> * 从头/尾端出发，沿后继/前驱引用
>> * 然而，此时的循秩访问成本过高，已不合时宜
> * 因此，应改用循位置访问（call-by-position）的方式
> * 亦即，应转而利用节点之间的相互引用，找到特定的节点
> * 比喻：找到 我的朋友A 的亲戚B 的同事c 的战友D ...的同学z

### 1.4 列表节点ADT接口

| 操作 | 功能 |
| --- | --- |
| pred() | 当前节点前驱节点的位置 |
| succ() | 当前节点后继节点的位置 |
| data() | 当前节点所存数据对象 |
| insertAsPred(e) | 插入前驱节点，存入被引用对象e，返回新节点位置 |
| insertAsSucc(e) | 插入后继节点，存入被引用对象e，返回新节点位置 |

### 1.5 ListNode模板类

~~~CPP
//列表节点位置
#define ListNodePosi(T) ListNode<T>*
template <typename T>//简洁起见，完全开放而不再过度封装
struct ListNode {//列表节点模板类（以双向链表形式实现）
    //成员
    T data;//数值
    ListNodePosi(T) pred;//前驱
    ListNodePosi(T) succ;//后继
    //构造函数
    ListNode() {}//针对header和trailer的构造
    ListNode(T e, ListNodePosi(T) p = NULL, ListNodePosi(T) s = NULL)
        : data(e), pred(p), succ(s) {}//默认构造器
    //操作接口
    ListNodePosi(T) insertAsPred(T const & e);//紧靠当前节点之前插入新节点
    ListNodePosi(T) insertAsSucc(T const & e);//紧随当前节点之后插入新节点
};
~~~

### 1.6 列表ADT接口

| 操作接口 | 功能 | 适用对象 |
| --- | --- | --- |
| size() | 报告列表当前的规模（节点总数） | 列表 |
| first() / last() | 返回首/未节点的位置 | 列表 |
| insertAsFirst(e)/insertAsLast(e) | 将e当作首/未节点插入 | 列表 |
| insertA(p, e)/insertB(p, e) | 将e当作节点p的直接后继/前驱插入 | 列表 |
| remove(p) | 删除位置p处的节点，返回其引用 | 列表 |
| disordered() | 判断所有节点是否已按降序排列 | 列表 |
| sort() | 调整各节点的位置，使之按非降序排列 | 列表 |
| find(e) | 查找目标元素e，失败时返回NULL | 列表 |
| search(e) | 查找e，返回不大于e且秩最大的节点 | 有序列表 |
| deduplicate() / uniquify() | 剔除重复节点 | 列表/有序列表 |
| traverse() | 遍历列表 | 列表 |

### 1.7 List模板类

~~~CPP
#include "listNode.h"//引入列表节点类
template <typename T> class List {//列表模板类
    private: int _size;//规模
        ListNodePosi(T) header;//头哨兵
        ListNodePosi(T) trailer;//尾哨兵
    protected:
        /*...内部函数*/
    public:
        /*...构造函数*/
        /*...析构函数*/
        /*...只读接口*/
        /*...可写接口*/
        /*...遍历接口*/
};
~~~

### 1.8 构造

~~~CPP
template <typename T>
void List<T>::init() {//列表初始化，在创建列表对象时统一调用
    header = new ListNode<T>;//创建头哨兵节点
    trailer = new ListNode<T>;//创建尾哨兵节点
    header -> succ = trailer; header -> pred = NULL;
    trailer -> pred = header; trailer -> succ = NULL;
    _size = 0;//记录规模
}
~~~

---

## 2.无序列表

### 2.1 秩到位置

通过重载下标操作符，可以模仿向量的循秩访问方式

~~~CPP
template <typename T>//重载下标操作符，通过秩直接访问列表节点（虽方便，效率低，需慎用）
T & List<T>::operator[](Rank r) const {
    ListNodePosi(T) p = first();//从首节点出发
    while(0 < r--) p = p -> succ;//顺数第r个节点即是
    return p -> data;//目标节点，返回其中所存元素
}
~~~

> * 时间复杂度为O(r)，线性正比于待访问节点的秩
> * 以均匀分布为例，单次访问的期望复杂度为：(1 + 2 + 3 + ... + n) / n = (n + 1) / 2 = O(n)

### 2.2 查找

在无序列表节点p（或为trailer）的n个（真）前驱中，找到等于e的最后者

~~~CPP
template <typename T>//从外部调用时，0 ≤ n ≤ rank(p) < _size
ListNodePosi(T) List<T>::find(T const & e, int n, ListNodePosi(T) p) const {
    while(0 < n--)//从右向左，逐个将p的前驱与e比对
        if(e == (p = p -> pred) -> data) return p;//逐个比对，直至命中或范围越界
    return NULL;//若越出左边界，意味着查找失败
}
~~~

### 2.3 插入

~~~CPP
template <typename T> ListNodePosi(T) List<T>::insertAsFirst(T const & e)
{_size++; return header -> insertAsSucc(e);}//e当作首节点插入

template <typename T> ListNodePosi(T) List<T>::insertAsLast(T const & e)
{_size++; return trailer -> insertAsPred(e);}//e当作末节点插入

template <typename T> ListNodePosi(T) List<T>::insertA(ListNodePosi(T) p, T const & e)
{_size++; return p -> insertAsSucc(e);}//e当作p的后继插入（After）

template <typename T> ListNodePosi(T) List<T>::insertB(ListNodePosi(T) p, T const & e)
{_size++; return p -> insertAsPred(e);}//e当作p的前驱插入（Before）
~~~

### 2.4 基于复制的构造

~~~CPP
//基本接口
template <typename T>//时间复杂度O(n)
void List<T>::copyNodes(ListNodePosi(T) p, int n) {
    init();//创建头、尾哨兵节点并做初始化
    //将起自p的n项依次作为末节点插入
    while(n--) {insertAsLast(p -> data); p = p -> succ;}
}
~~~

~~~CPP
//重载接口
List<T>::List(List<T> const & L)//时间复杂度O(_size)
{copyNodes(L.first(), L._size);}

List<T>::List(List<T> const & L, int r, int n)//时间复杂度O(r + n)
{copyNodes(L[r], n);}
~~~

### 2.5 删除

~~~CPP
template <typename T>//删除合法节点p，返回其数值
T List<T>::remove(ListNodePosi(T) p) {//时间复杂度O(1)
    T e = p -> data;//备份待删除节点数值（设类型T可直接赋值）
    p -> pred -> succ = p -> succ;//后继
    p -> succ -> pred = p -> pred;//前驱
    delete p; _size--;//释放节点，更新规模
    return e;//返回备份的数值
}
~~~

### 2.6 析构

~~~CPP
template <typename T> List<T>::~List()//列表析构
{clear(); delete header; delete trailer;}//清空列表，释放头、尾哨兵节点

template <typename T> int List<T>::clear() {//清空列表
    int oldSize = _size;
    while(0 < _size)//反复删除首节点，直至列表变空
        remove(header -> succ);
    return oldSize;
}//时间复杂度O(n)，线性正比于列表规模
~~~

### 2.7 唯一化

~~~CPP
template <typename T>
int List<T>::deduplicate() {//剔除无序列表中的重复节点
    if(_size < 2) return 0;//平凡列表自然无重复
    int oldSize = _size;//记录原规模
    ListNodePosi(T) p = first(); Rank r = 1;//p从首节点起
    while(trailer != (p = p -> succ)) {//依次直到末节点
        //在p的r个（真）前驱中，查找与之雷同者
        ListNodePosi(T) q = find(p -> data, r, p);
        q ? remove(q) : r++;//若的确存在，则删除之；否则秩递增
    }
    return oldSize - _size;//返回删除元素总数
}
~~~

### 2.8 遍历

~~~CPP
template <typename T>
void List<T>::traverse(void (* visit) (T &))//借助函数指针机制遍历
{for(ListNodePosi(T) p = header -> succ; p != trailer; p = p -> succ) visit(p -> data);}

template <typename T> template <typename VST>//元素类型、操作器
void List<T>::traverse(VST & visit)//借助函数对象机制遍历
{for(ListNodePosi(T) p = header -> succ; p != trailer; p = p -> succ) visit(p -> data);}
~~~

---

## 3.有序列表

### 3.1 唯一化

~~~CPP
template <typename T>
int List<T>::uniquify() {//成批剔除重复元素，效率更高
    if(_size < 2) return 0;//平凡列表自然无重复
    int oldSize = _size;//记录原规模
    ListNodePosi(T) p = first();//p为各区段起点
    ListNodePosi(T) q;//q为其后继
    while(trailer != (q = p -> succ))//反复考查紧邻的节点对(p, q)
        if(p -> data != q -> data) p = q;//若互异，则转向下一区段
        else remove(q);//否则雷同，删除后者
    return oldSize - _size;//返回删除元素总数
}//只需遍历整个列表一趟，时间复杂度O(n)
~~~

### 3.2 查找

在有序列表节点p（或为trailer）的n个（真）前驱中，找到不大于e的最后者

~~~CPP
template <typename T>
ListNodePosi(T) List<T>::search(T const & e, int n, ListNodePosi(T) p) const {
    while(0 <= n--)//对于p最近的n个前驱，从右向左
        if(((p = p -> pred) -> data) <= e) break;//逐个比较
    return p;//直至命中、数值越界或范围越界后，返回查找终止的位置
}//最好O(1)，最坏O(n)；等概率时平均O(n)，正比于区间宽度
~~~

> * 语义与向量相似，便于插入排序等后续操作
> * 按照循位置访问的方式，<u>物理存储地址</u>与其<u>逻辑次序</u>无关
> * 依据秩的随机访问无法高效实现，而只能依据元素间的引用顺序访问

---

## 4.选择排序

### 4.1 构思

> 回忆起泡排序：
> * 每经一趟扫描交换，当前的最大元素必然就位
> * 从未排序的元素中挑选出最大者，并使之就位
> * 起泡排序之所以需要O(n&sup2;)时间，是因为：
>> 为挑选每个最大元素M，需做O(n)次比较和O(n)次交换
> * O(n)次比较或许无可厚非，但O(n)次交换绝对没有必要
> * 经O(n)次比较确定M之后，一次交换足矣

### 4.2 selectionSort()

~~~CPP
template <typename T>//对列表中起始于位置p、宽度为n的区间做选择排序
void List<T>::selectionSort(ListNodePosi(T) p, int n) {
    ListNodePosi(T) head = p -> pred; ListNodePosi(T) tail = p;
    for(int i = 0; i < n; i++) tail = tail -> succ;//待排序区间为(head, tail)
    while(1 < n) {//在至少还剩两个节点之前，在待排序区间内
        ListNodePosi(T) max = selectMax(head -> succ, n);//找出最大者（歧义时后者优先）
        insertB(tail, remove(max));//将其移至无序区间末尾（作为有序区间新的首元素）
        tail = tail -> pred; n--;
    }
}
~~~

### 4.3 selectMax()

~~~CPP
template <typename T>//从起始于位置p的n个元素中选出最大者
ListNodePosi(T) List<T>::selectMax(ListNodePosi(T) p, int n) {
    ListNodePosi(T) max = p;//最大者暂定为首节点p
    for(ListNodePosi(T) cur = p; 1 < n; n--)//从首节点p出发，将后续节点逐一与max比较
        if(!lt((cur = cur -> succ) -> data, max -> data))//若当前元素不小于max
            max = cur;//更新最大元素位置记录
    return max;//返回最大节点位置
}
~~~

### 4.4 性能

> * 共迭代n次，在第k次迭代中：
>> * selectMax()为Θ(n - k)
>> * remove()和insertB()均为O(1)
> * 故总体复杂度应为Θ(n&sup2;)
> * 尽管如此，元素移动操作远远少于起泡排序
> * 也就是说，Θ(n&sup2;)主要来自于元素比较操作
> * 利用高级数据结构，selectMax()可改进至O(logn)
> * 当然，如此立即可以得到O(nlogn)的排序算法

---

## 5.插入排序

### 5.1 构思

> * 始终将序列看成两部分：
>> * Sorted + Unsorted
>> * L[0, r) + L[r, n)
> * 初始化：空序列无所谓有序
>> * |S| = r = 0
> * 迭代：关注并处理e = L[r]
>> * 在S中确定适当位置（有序序列的查找）
>> * 插入e，得到有序的L[0, r]（有序序列的插入）
> * 不变性：
>> * 随着r的递增，L[0, r)始终有序
>> * 直到r = n，L即整体有序

### 5.2 实现

~~~CPP
template <typename T>//对列表中起始于位置p、宽度为n的区间做插入排序
void List<T>::insertionSort(ListNodePosi(T) p, int n) {
    for(int r = 0; r < n; r++) {//逐一为各节点
        insertA(search(p -> data, r, p), p -> data);//查找适当的位置并插入
        p = p -> succ; remove(p -> pred);//转向下一节点
    }//n次迭代，每次时间复杂度O(r + 1)
}//仅使用O(1)辅助空间，属于就地算法
~~~

### 5.3 性能

> * 最好情况：完全（或几乎）有序
>> * 每次迭代，只需1次比较，0次交换
>> * 累计O(n)时间
> * 最坏情况：完全（或几乎）逆序
>> * 第k次迭代，需O(k)次比较，1次交换
>> * 累计O(n&sup2;)时间
> * 一般情况：包含I个逆序对
>> * 第k次迭代，只需O(I<sub>k</sub>)次比较，1次交换
>> * 累计O(n + I)时间

### 5.4 平均性能

> * 假定：各元素的取值遵守均匀、独立分布
> * 于是：平均要做多少次元素比较？
> * 考查：L[r]刚插入完成的那一时刻
> * 试问：此时的有序前缀L[0, r]中，哪个元素是此前的L[r]？
> * 观察：其中的r + 1个元素均有可能，且概率均等于1 / (r + 1)
> * 因此，在刚完成的这次迭代中，为引入S[r]所花费时间的数学期望为：
>> * [r + (r - 1) + ... + 3 + 2 + 1 + 0] / (r + 1) + 1 = r / 2 + 1
> * 于是，总体时间的数学期望：
>> * [0 + 1 + ... + (n - 1)] / 2 + 1 = O(n&sup2;)

---

## 6.归并排序

### 6.1 归并排序

~~~CPP
template <typename T>//列表的归并排序算法：对起始于位置p的n个元素排序
void List<T>::mergeSort(ListNodePosi(T) & p, int n) {
    if(n < 2) return;//若待排序范围已足够小，则直接返回
    int m = n >> 1;//以中点为界
    ListNodePosi(T) q = p; for(int i = 0; i < m; i++) q = q -> succ;//均分列表
    mergeSort(p, m); mergeSort(q, n - m);//对前、后子列表分别排序
    merge(p, m, * this, q, n - m);//归并
}//注意：排序后，p依然指向归并后区间的（新）起点
~~~

### 6.2 二路归并

~~~CPP
template <typename T>
//有序列表的归并：当前列表中自p起的n个元素，与列表L中自q起的m个元素归并
void List<T>::merge(ListNodePosi(T) & p, int n, List<T> & L, ListNodePosi(T) q, int m) {
    ListNodePosi(T) pp = p -> pred;
    while(0 < m)//在q尚未移出区间之前
        if((0 < n) && (p -> data <= q -> data))//若p仍在区间内且v(p) ≤ v(q)
        {if(q == (p = p -> succ)) break; n--;}//p归入合并的列表，并替换为其直接后继
        else//若p已超出右界或v(q) < v(p)
        {insertB(p, L.remove((q = q -> succ) -> pred)); m--;}//将q转移至p之前
    p = pp -> succ;//确定归并后区间的（新）起点
}
~~~
