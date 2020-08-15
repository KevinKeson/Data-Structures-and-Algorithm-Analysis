# 《数据结构与算法分析》

# 向量 Vector

> 线性序列 Sequence
>> 向量 Vector / 列表 List

## 1.接口与实现

---

### 1.1 抽象数据类型

*抽象数据类型* = 数据模型 + 定义在该模型上的一组操作

*数据结构* = 基于某种特定语言 实现ADT的一整套算法

<u>高层算法设计者</u>与<u>底层数据结构实现者</u>可以高效地分工协作

---

### 1.2 从数组到向量

***向量：由一组元素按线性次序封装而成，是数组的抽象与泛化***

向量中的元素与[0, n)内的秩（rank）相互对应，并可进行循秩访问（call-by-rank）

操作、管理与维护变得更加简化、统一与安全，元素类型可灵活选取，便于定制复杂数据结构

---

### 1.3 向量ADT接口

| 操作 | 功能 | 适用对象 |
| --- | --- | --- |
| size() | 报告向量当前的规模（元素总数） | 向量 |
| get(r) | 获取秩为r的元素 | 向量 |
| put(r, e) | 用e替换秩为r元素的数值 | 向量 |
| insert(r, e) | e作为秩为r元素插入，原后继依次后移 | 向量 |
| remove(r) | 删除秩为r的元素，返回该元素原值 | 向量 |
| disordered() | 判断所有元素是否已按非降序排列 | 向量 |
| sort() | 调整各元素的位置，使之按非降序排列 | 向量 |
| find(e) | 查找目标元素e | 向量 |
| search(e) | 查找e，返回不大于e且秩最大的元素 | 有序向量 |
| deduplicate() / uniquify() | 剔除重复元素 | 向量/有序向量 |
| traverse() | 遍历向量并统一处理所有元素 | 向量 |

---

### 1.4 Vector模板类

~~~CPP
#define DEFAULT_CAPACITY 3//默认的初始容量（实际应用中可设置为更大）
template <typename T> class Vector {//向量模板类
    private: Rank _size; int _capacity; T * _elem;//规模、容量、数据区
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

---

### 1.5 构造与析构

~~~CPP
Vector(int c = DEFAULT_CAPACITY)
{_elem = new T[_capacity = c]; _size = 0;}//默认

Vector(T const * A, Rank lo, Rank hi)
{copyFrom(A, lo, hi);}//数组区间复制

Vector(Vector<T> const & V, Rank lo, Rank hi)
{copyFrom(V._elem, lo, hi);}//向量区间复制

Vector(Vector<T> const & V)
{copyFrom(V._elem, 0, V._size);}//向量整体复制

~Vector() {delete [] _elem;}//释放内部空间
~~~

---

### 1.6 基于复制的构造

~~~CPP
template <typename T>//T为基本类型，或已重载赋值操作符'='
void Vector<T>::copyFrom(T const * A, Rank lo, Rank hi) {
    _elem = new T[_capacity = 2*(hi - lo)];//分配空间
    _size = 0;//规模清零
    while(lo < hi)//A[lo, hi)内的元素逐一遍历
        _elem[_size++] = A[lo++];//复制至_elem[0, hi - lo)
}//时间复杂度O(hi - lo) = O(n)
~~~

## 2.可扩充向量

---

### 2.1 静态空间管理

> 开辟内部数组_elem[ ]并使用一段地址连续的物理空间
>> _capacity：总容量 / _size：当前的实际规模n

> 若采用静态空间管理策略，容量_capacity固定，则有明显不足
> 1. 上溢（overflow）：_elem[ ]不足以存放所有元素，尽管此时系统仍有足够空间
> 2. 下溢（underflow）：_elem[ ]中的元素寥寥无几，装填因子（load factor）λ = _size / _capacity << 50%

***更糟糕的是，一般的应用环境中，难以准确预测空间的需求量***

---

### 2.2 动态空间管理

在即将发生上溢时，适当扩大内部数组容量，扩容算法实现如下：

~~~CPP
template <typename T> void Vector<T>::expand() {//向量空间不足时扩容
    if(_size < _capacity) return;//尚未满员时，不必扩容
    _capacity = max(_capacity, DEFAULT_CAPACITY);//不低于最小容量
    T * oldElem = _elem; _elem = new T[_capacity <<= 1];//容量加倍（最好的策略）
    for(int i = 0; i < _size; i++)//复制原向量内容
        _elem[i] = oldElem[i];
    delete [] oldElem;//释放原空间
}//得益于向量的封装，尽管扩容之后数据区的物理地址有所改变，却不致出现野指针
~~~

对比<u>容量递增策略</u>与<u>容量加倍策略</u>（空间换取时间）

| 对比项 | 递增策略 | 倍增策略 |
| --- | :---: | :---: |
| 累计增容时间 | O(n&sup2;) | O(n) |
| 分摊增容时间 | O(n) | O(1) |
| 装填因子 | ≈ 100% | > 50% |

---

### 2.3 平均分析与分摊分析

> 平均复杂度或期望复杂度（average/expected complexity）
> * 根据数据结构各种操作出现的概率分布，将对应的成本加权平均
> * 各种可能的操作，作为独立事件分别考查
> * 割裂了操作之间的相关性和连贯性
> * 往往不能准确地评判数据结构和算法的真实性能

> 分摊复杂度（amortized complexity）
> * 对数据结构连续地实施足够多次操作，所需总体成本分摊至单次操作
> * 从实际可行的角度，对一系列操作做整体的考量
> * 更加忠实地刻画了可能出现的操作序列
> * 更为精准地评判数据结构和算法的真实性能

## 3.无序向量

---

### 3.1 元素访问

通过V.get(r)与V.put(r, e)接口，可以读、写向量元素

重载下标操作符'[ ]'，即可沿用借助下标的访问方式

~~~CPP
template <typename T>//0 ≤ r < _size
T & Vector<T>::operator[](Rank r) const {return _elem[r];}
~~~

---

### 3.2 插入

~~~CPP
template <typename T>//将e作为秩为r元素插入，0 ≤ r < _size
Rank Vector<T>::insert(Rank r, T const & e) {//时间复杂度O(n - r)
    expand();//若必要则扩容
    for(int i = _size; i > r; i--)
        _elem[i] = _elem[i - 1];//自后向前，后继元素顺次后移一个单元
    _elem[r] = e; _size++;//置入新元素并更新容量
    return r;//返回秩
}
~~~

---

### 3.3 区间删除

~~~CPP
template <typename T>//删除区间[lo, hi)，0 ≤ lo ≤ hi < _size
int Vector<T>::remove(Rank lo, Rank hi) {//时间复杂度O(n - hi)
    if(lo == hi) return 0;//出于效率考虑，单独处理退化情况，比如remove(0, 0)
    while(hi < _size) _elem[lo++] = _elem[hi++];//[hi, _size)顺次前移
    _size = lo; shrink();//更新规模，若有必要则缩容
    return hi - lo;//返回被删除元素的数目
}
~~~

---

### 3.4 单元素删除

可视作区间删除操作的特例[r] = [r, r + 1)

~~~CPP
template <typename T>//删除向量中秩为r的元素，0 ≤ r < _size
T Vector<T>::remove(Rank r) {//时间复杂度O(n - r)
    T e = _elem[r];//备份被删除元素
    remove(r, r + 1);//调用区间删除算法
    return e;//返回被删除元素
}
~~~

---

### 3.5 查找

> * 无序向量：T为可判等的基本类型，或已重载操作符'=='或'!='
> * 有序向量：T为可比较的基本类型，或已重载操作符'<'或'>'

~~~CPP
template <typename K, typename V> struct Entry {//词条模板类
    K key; V value;//关键码、数值
    Entry(K k = K(), V v = V()) : key(k), value(v) {};//默认构造函数
    Entry(Entry < K, V > const & e) : key(e.key), value(e.value) {};//基于克隆的构造函数
    bool operator <  (Entry < K, V > const & e) {return key <  e.key;}//比较器：小于
    bool operator >  (Entry < K, V > const & e) {return key >  e.key;}//比较器：大于
    bool operator == (Entry < K, V > const & e) {return key == e.key;}//判等器：等于
    bool operator != (Entry < K, V > const & e) {return key != e.key;}//判等器：不等于
};//得益于比较器和判等器，从此往后，不必严格区分词条及其对应的关键码
~~~

~~~CPP
template <typename T>//时间复杂度O(hi - lo) = O(n)，在命中多个元素时返回秩最大者
Rank Vector<T>::find(T const & e, Rank lo, Rank hi) const {//0 ≤ lo < hi ≤ _size
    while((lo < hi--) && (e != _elem[hi]));//逆向查找
    return hi;//若hi < lo，则意味着失败；否则hi即命中元素的秩
}//输入敏感（input-sensitive）：最好O(1)，最差O(n)
~~~

---

### 3.6 唯一化

~~~CPP
template <typename T>//删除重复元素，返回被删除元素数目
int Vector<T>::deduplicate() {
    int oldSize = _size;//记录原规模
    Rank i = 1;//从_elem[1]开始
    while(i < _size)//自前向后逐一考查各元素_elem[i]
        //在前缀中寻找雷同者，若无雷同则继续考查其后继，否则删除雷同者
        (find(_elem[i], 0, i) < 0) ? i++ : remove(i);
    return oldSize - _size;//向量规模变化量，即被删除元素总数
}
~~~

> 每轮迭代中find()和remove()累计耗费线性时间，总体时间复杂度O(n&sup2;)
> 1. 仿照uniquify()高效版的思路，元素移动的次数可降至O(n)，但比较次数依然是O(n&sup2;)，而且稳定性将被破坏
> 2. 先对需删除的重复元素做标记，然后再统一删除，稳定性保持，但因查找长度更长，从而导致更多的比对操作

## 4.有序向量 唯一化

---

### 4.1 有序性及其甄别

| 序列 | 一对 | 相邻元素 |
| --- | --- | :---: |
| 有序 | 任意 | 顺序 |
| 无序 | 总有 | 逆序 |

相邻逆序对的数目，可用以度量向量的逆序程度

无序向量经过预处理转换为有序向量后，相关算法多可优化

~~~CPP
template <typename T>
int Vector<T>::disordered() const {//统计向量中的逆序相邻元素对
    int n = 0;//计数器
    for(int i = 1; i < _size; i++)//逐一检查各对相邻元素
        n += (_elem[i - 1] > _elem[i]);//逆序则计数
    return n;//向量有序当且仅当n = 0
}//若只需判断是否有序，则首次遇到逆序对之后，即可立即终止
~~~

---

### 4.2 低效算法

在有序向量中，重复的元素必然相互紧邻构成一个区间

因此，每一区间只需保留单个元素即可

~~~CPP
template <typename T>
int Vector<T>::uniquify() {//有序向量重复元素剔除算法（低效版）
    int oldSize = _size; int i = 1;//从首元素开始
    while(i < _size)//从前向后，逐一比对各对相邻元素
        //若雷同，则删除后者；否则，转至后一元素
        _elem[i - 1] == _elem[i] ? remove(i) : i++;
    return oldSize - _size;//向量规模变化量，即被删除元素总数
}//注意：其中_size的减小，由remove()隐式地完成
~~~

运行时间主要取决于while循环，次数共计_size - 1 = n - 1

最坏情况下，每次都需调用remove()，耗时O(n - 1) ~ O(1)，累计时间复杂度O(n&sup2;)

尽管省去find()，总体竟与无序向量的deduplicate()相同

---

### 4.3 高效算法

> * 反思：低效的根源在于，同一元素可作为被删除元素的后继多次前移
> * 启示：若能以重复区间为单位，成批删除雷同元素，性能必将改进

共计n - 1次迭代，每次常数时间，累计时间复杂度O(n)

~~~CPP
template <typename T>
int Vector<T>::uniquify() {//有序向量重复元素剔除算法（高效版）
    Rank i = 0, j = 0;//各对互异“相邻”元素的秩
    while(++j < _size)//逐一扫描，直至末元素
        if(_elem[i] != _elem[j])//跳过雷同者
            _elem[++i] = _elem[j];//发现不同元素时，向前移至紧邻于前者右侧
    _size = ++i; shrink();//直接截除尾部多余元素
    return j - i;//向量规模变化量，即被删除元素总数
}//注意：通过remove(lo, hi)批量删除，依然不能达到高效率
~~~

~~~CPP
template <typename T>
void Vector<T>::shrink() {//装填因子过小时压缩向量所占空间
    if(_capacity < DEFAULT_CAPACITY << 1) return;//不致收缩到DEFAULT_CAPACITY以下
    if(_size << 2 > _capacity) return;//以25%为界
    T * oldElem = _elem; _elem = new T[_capacity >>= 1];//容量减半
    for(int i = 0; i < _size; i++) _elem[i] = oldElem[i];//复制原向量内容
    delete [] oldElem;//释放原空间
}
~~~

## 5.有序向量 二分查找

---

### 5.1 统一接口

~~~CPP
template <typename T>//查找算法统一接口，0 ≤ lo < hi ≤ _size
Rank Vector<T>::search(T const & e, Rank lo, Rank hi) const {
    //按各50%的概率随机使用二分查找或Fibonacci查找
    return (rand() % 2) ? binSearch(_elem, e, lo, hi) : fibSearch(_elem, e, lo, hi);
}
~~~

---

### 5.2 语义约定

> * 至少，应该便于有序向量自身的维护
> * 即便失败，也应给出新元素适当的插入位置
> * 若允许重复元素，则每一组也需按其插入的次序排列

> 约定：在有序向量区间V[lo, hi)中，确定不大于e的最后一个元素
> * 若-∞ < e < V[lo]，则返回lo - 1（左侧哨兵）
> * 若V[hi - 1] < e < +∞，则返回hi - 1（末元素 = 右侧哨兵左邻）

---

### 5.3 版本A

减而治之：以任一元素x = S[mi]为界，都可将待查找区间分为三部分，即S[lo, mi) ≤ S[mi] ≤ S(mi, hi)

> 只需将目标元素e与x做一比较，即可分三种情况进一步处理：
> 1. e < x：则e若存在必属于左侧子区间S[lo, mi)，故可递归深入
> 2. x < e：则e若存在必属于右侧子区间S(mi, hi)，亦可递归深入
> 3. e = x：已在此处命中，可随即返回

二分策略：轴点mi总是取作中点，每经过至多两次比较，或者能够命中，或者减半问题规模

~~~CPP
template <typename T>//在有序向量区间[lo, hi)内查找元素e
static Rank binSearch(T * S, T const & e, Rank lo, Rank hi) {
    while(lo < hi) {//每步迭代可能要做两次比较判断，有三个分支
        Rank mi = (lo + hi) >> 1;//以中点为轴点
        if(e < S[mi]) hi = mi;//深入前半段[lo, mi)继续查找
        else if(S[mi] < e) lo = mi + 1;//深入后半段(mi, hi)继续查找
        else return mi;//在mi处命中
    }//查找成功提前终止
    return -1;//查找失败
}//有多个命中元素时，不能保证返回秩最大者；查找失败时，简单地返回-1，而不能指示失败的位置
~~~

> * 线性递归：T(n) = T(n/2) + O(1) = O(logn)，大大优于顺序查找
> * 递归跟踪：轴点总取中点，递归深度O(logn)，各递归实例均耗时O(1)
> * 查找长度（search length）：考查关键码的比较次数，成功、失败时的平均查找长度均大致为O(1.50 · logn)

## 6.有序向量 Fibonacci查找

---

### 6.1 思路及原理

> * 二分查找版本A的效率仍有改进余地：转向左右分支前的关键码比较次数不等，而递归深度却相同
> * 若能通过<u>递归深度的不均衡</u>，对于<u>转向成本的不均衡</u>进行补偿，平均查找长度应能进一步缩短
> * 比如，若设n = fib(k) - 1，则可取mi = fib(k - 1) - 1
> * 于是，前后子向量的长度分别为fib(k - 1) - 1、fib(k - 2)- 1

---

### 6.2 实现

~~~CPP
template <typename T>
static Rank fibSearch(T * S, T const & e, Rank lo, Rank hi) {
    //用O(log_φ(n = hi - lo)时间创建Fib数列
    for(Fib fib(hi - lo); lo < hi;) {
        while(hi - lo < fib.get()) fib.prev();//自后向前顺序查找（分摊O(1)）
        Rank mi = lo + fib.get() - 1;//确定形如Fib(k) - 1的轴点
        if(e < S[mi]) hi = mi;//深入前半段[lo, mi)继续查找
        else if(S[mi] < e) lo = mi + 1;//深入后半段(mi, hi)继续查找
        else return mi;//在mi处命中
    }//查找成功提前终止
    return -1;//查找失败
}//有多个命中元素时，不能保证返回秩最大者；查找失败时，简单地返回-1，而不能指示失败的位置
~~~

---

### 6.3 查找长度

> * 通用策略：对于任何的A[0, n)，总是选取A[λn]作为轴点，0 ≤ λ < 1
> * 比如：<u>二分查找</u>对应于λ = 0.5，<u>Fibonacci查找</u>对应于λ = φ = 0.6180339
> * 在[0, 1)内，λ如何取值才能达到最优？设平均查找长度为α(λ) · log<sub>2</sub>n，何时α(λ)最小？
>> * 递推式：α(λ) · log<sub>2</sub>n = λ · [1 + α(λ) · log<sub>2</sub>(λn)] + (1 - λ) · [2 + α(λ) · log<sub>2</sub>((1 - λ)n)]
>> * 整理后：- ln2 / α(λ) = [λ · lnλ + (1 - λ) · ln(1 - λ)] / (2 - λ)
>> * 当λ = φ（黄金比例）时，α(λ) = 1.440420达到最小

## 7.有序向量 二分查找（改进）

---

### 7.1 改进思路

> * 二分查找中，左右分支转向代价不平衡的问题，也可直接解决
> * 比如，每次迭代（或每个递归实例）仅做1次关键码比较
> * 如此，所有分支只有2个方向，而不再是3个
> * 同样地，轴点mi取作中点，则查找每深入一层，问题规模也缩减一半
>> 1. e < x：则e若存在必属于左侧子区间S[lo, mi)，故可递归深入
>> 2. x ≤ e：则e若存在必属于右侧子区间S[mi, hi)，亦可递归深入
>> 3. 只有当元素数目hi - lo = 1时，才判断该元素是否命中

---

### 7.2 版本B

~~~CPP
template <typename T>
static Rank binSearch(T * S, T const & e, Rank lo, Rank hi) {
    while(1 < hi - lo) {//有效查找区间缩短为1，算法才会终止
        Rank mi = (lo + hi) >> 1;//以中点为轴点，经比较后确定深入
        (e < S[mi]) ? hi = mi : lo = mi;//[lo, mi)或[mi, hi)
    }//出口时hi = lo + 1，查找区间仅含一个元素S[lo]
    return e == S[lo] ? lo : -1;//返回命中元素的秩或者-1
}//相对于版本A，最好（坏）情况下更坏（好）﹔各种情况下的SL更加接近，整体性能更趋稳定
~~~

---

### 7.3 语义约定

> * 以上二分查找及Fibonacci查找算法
> * 均未严格地兑现search()接口的语义约定：返回不大于e的最后一个元素
> * 只有兑现这一约定，才可有效支持相关算法，比如：V.insert(1 + V.search(e), e)
>> 1. 当有多个命中元素时，必须返回最靠后（秩最大）者
>> 2. 失败时，应返回小于e的最大者（含哨兵[lo - 1]）

---

### 7.4 版本C

~~~CPP
template <typename T>
static Rank binSearch(T * S, T const & e, Rank lo, Rank hi) {
    while(lo < hi) {
        Rank mi = (lo + hi) >> 1;//以中点为轴点，经比较后确定深入
        (e < S[mi]) ? hi = mi : lo = mi + 1;//[lo, mi)或(mi, hi)
    }//出口时S[lo = hi]为大于e的最小元素
    return --lo;//故lo - 1即不大于e的元素的最大秩
}
~~~

> 版本C与版本B的差异：
> 1. 待查找区间宽度缩短至0而非1时，算法才结束
> 2. 转入右侧子向量时，左边界取作mi + 1而非mi
> 3. 无论成功与否，返回的秩严格符合接口的语义约定

## 8.有序向量 插值查找

---

### 8.1 原理与算法

> * 假设：已知有序向量中各元素随机分布的规律
> * 比如：均匀且独立的随机分布
> * 于是：[lo, hi)内各元素应大致按照线性趋势增长
> * 因此：通过猜测轴点mi，可以极大地提高收敛速度
> * 以英文词典为例：
>> 1. binary大致位于2/26处
>> 2. search大致位于19/26处

---

### 8.2 性能

> * 最坏：O(hi - lo) = O(n)
> * 平均：每经一次比较，待查找区间宽度由n缩至√n
> * 有效字长logn减半，时间复杂度O(loglogn)
>> 1. 插值查找 = 在字长意义上的折半查找
>> 2. 二分查找 = 在字长意义上的顺序查找

---

### 8.3 综合

> 从O(logn)到O(loglogn)是否值得？
> * 通常优势不明显，除非查找区间宽度极大，或者比较操作成本较高
> * 易受小扰动的“干扰”和“蒙骗”
> * 需要引入乘法、除法运算
> * 实际可行的方法：
>> 1. 首先进行插值查找，将查找范围缩小到一定尺度
>> 2. 然后进行二分查找，完成查找任务

## 9.起泡排序

---

### 9.1 统一入口

~~~CPP
template <typename T>//排序器统一入口
void Vector<T>::sort(Rank lo, Rank hi) {//向量区间[lo, hi)排序
    switch(rand() % 6) {
    case 1:  bubbleSort(lo, hi); break;//起泡排序
    case 2:  selectionSort(lo, hi); break;//选择排序（习题）
    case 3:  mergeSort(lo, hi); break;//归并排序
    case 4:  heapSort(lo, hi); break;//堆排序（第12章）
    case 5:  quickSort(lo, hi); break;//快速排序（第14章）
    default: shellSort(lo, hi); break;//希尔排序（第14章）
    }//随机选择算法以充分测试，实用时可视具体问题的特点灵活确定或扩充
}
~~~

---

### 9.2 起泡排序

~~~CPP
template <typename T>
void Vector<T>::bubbleSort(Rank lo, Rank hi)
{while(!bubble(lo, hi--));}//逐趟做扫描交换，直至全序
~~~

~~~CPP
template <typename T>
bool Vector<T>::bubble(Rank lo, Rank hi) {
    bool sorted = true;//整体有序标志
    while(++lo < hi)
        if(_elem[lo - 1] > _elem[lo]) {
            //若逆序，则意味着尚未整体有序，并需要交换
            sorted = false;
            swap(_elem[lo - 1], _elem[lo]);
        }
    return sorted;//返回有序标志
}//乱序限于[0, √n]时，仍需O(n^1.5)时间，按理O(n)应已足矣
~~~

---

### 9.3 改进

~~~CPP
template <typename T>
void Vector<T>::bubbleSort(Rank lo, Rank hi)
{while(lo < (hi = bubble(lo, hi)));}//逐趟做扫描交换，直至全序
~~~

~~~CPP
template <typename T>
Rank Vector<T>::bubble(Rank lo, Rank hi) {
    Rank last = lo;//最右侧的逆序对初始化为[lo - 1, lo]
    while(++lo < hi)//自左向右，逐一检查各对相邻元素
        if(_elem[lo - 1] > _elem[lo]) {
            //若逆序，则更新最右侧逆序对位置记录，并需要交换
            last = lo;
            swap(_elem[lo - 1], _elem[lo]);
        }
    return last;//返回最右侧逆序对的位置
}//上一版本的逻辑型标志sorted，改为秩last
~~~

## 10.归并排序

---

### 10.1 实现

~~~CPP
template <typename T>//向量归并排序
void Vector<T>::mergeSort(Rank lo, Rank hi) {//0 ≤ lo < hi ≤ _size
    if(hi - lo < 2) return;//单元素区间自然有序
    int mi = (lo + hi) / 2;//以中点为界
    mergeSort(lo, mi); mergeSort(mi, hi);//分别排序
    merge(lo, mi, hi);//归并
}
~~~

---

### 10.2 二路归并

~~~CPP
template <typename T>
void Vector<T>::merge(Rank lo, Rank mi, Rank hi) {//各自有序的子向量[lo, mi)和[mi, hi)
    T * A = _elem + lo;//合并后的向量A[0, hi - lo) = _elem[lo, hi)
    int lb = mi - lo; T * B = new T[lb];//前子向量B[0, lb) = _elem[lo, mi)
    for(Rank i = 0; i < lb; B[i] = A[i++]);//复制前子向量
    int lc = hi - mi; T * C = _elem + mi;//后子向量C[0, lc) = _elem[mi, hi)
    for(Rank i = 0, j = 0, k = 0; j < lb || k < lc;) {//B[j]和C[k]中小者转至A的末尾
        if(j < lb && (lc <= k || B[j] <= C[k])) A[i++] = B[j++];//C[k]已无或不小
        if(k < lc && (lb <= j || C[k] <  B[j])) A[i++] = C[k++];//B[j]已无或更大
    }
    delete [] B;//释放临时空间B
}
~~~

---

### 10.3 精简实现

~~~CPP
for(Rank i = 0, j = 0, k = 0; j < lb;)
    A[i++] = (lc <= k || B[j] <= C[k]) ? B[j++] : C[k++];
~~~

---

### 10.4 复杂度

> * 算法的运行时间主要消耗于for循环，共有两个控制变量：
>> * 初始：j = 0, k = 0
>> * 最终：j = lb, k = lc
>> * 亦即：j + k = lb + lc = hi - lo = n
> * 观察：每经过一次迭代，j和k中至少有一个会加一（j + k也必至少加一）
> * 故知：merge()总体迭代不过O(n)次，累计只需线性时间
> * 这一结论与排序算法的Ω(nlogn)下界并不矛盾，毕竟这里的B和C均已各自有序
> * 注意：待归并子序列不必等长
> * 亦即：允许lb ≠ lc, mi ≠ (lo + hi) / 2
> * 实际上，这一算法及结论也适用于另一类序列（列表）

---

### 10.5 综合评价

> 优点：
> * 实现最坏情况下最优O(nlogn)性能的第一个排序算法
> * 不需随机读写，完全顺序访问，尤其适用于：
>> * 列表之类的序列
>> * 磁带之类的设备
> * 只要实现恰当，可保证稳定（出现雷同元素时，左侧子向量优先）
> * 可扩展性极佳，十分适宜于外部排序（海量网页搜索结果的归并）
> * 易于并行化

> 缺点：
> * 非就地，需要对等规模的辅助空间
> * 即便输入完全（或接近）有序，仍需Θ(nlogn)时间

# 列表 List

## 1.接口与实现

---

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

---

### 1.2 从向量到列表

> * 列表（List）是采用动态储存策略的典型结构
>> * 其中的元素称作节点（node）
>> * 各节点通过指针或引用彼此联接，在逻辑上构成一个线性序列：L = {a<sub>0</sub>, a<sub>1</sub>, ..., a<sub>n-1</sub>}
> * 相邻节点彼此互称前驱（predecessor）或后继（successor）
>> * 前驱或后继若存在，则必然唯一
>> * 没有前驱/后继的唯一节点称作首（first/front）/末（last/rear）节点

---

### 1.3 从秩到位置

> * 向量支持循秩访问（call-by-rank）的方式
> * 根据数据元素的秩，可在O(1)时间内直接确定其物理地址
> * 既然同属线性序列，列表固然也可通过秩来定位节点：
>> * 从头/尾端出发，沿后继/前驱引用
>> * 然而，此时的循秩访问成本过高，已不合时宜
> * 因此，应改用循位置访问（call-by-position）的方式
> * 亦即，应转而利用节点之间的相互引用，找到特定的节点
> * 比喻：找到 我的朋友A 的亲戚B 的同事c 的战友D ...的同学z

---

### 1.4 列表节点ADT接口

| 操作 | 功能 |
| --- | --- |
| pred() | 当前节点前驱节点的位置 |
| succ() | 当前节点后继节点的位置 |
| data() | 当前节点所存数据对象 |
| insertAsPred(e) | 插入前驱节点，存入被引用对象e，返回新节点位置 |
| insertAsSucc(e) | 插入后继节点，存入被引用对象e，返回新节点位置 |

---

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

---

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

---

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

---

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

## 2.无序列表

---

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

---

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

---

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

---

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

---

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

---

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

---

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

---

### 2.8 遍历

~~~CPP
template <typename T>
void List<T>::traverse(void (* visit) (T &))//借助函数指针机制遍历
{for(ListNodePosi(T) p = header -> succ; p != trailer; p = p -> succ) visit(p -> data);}

template <typename T> template <typename VST>//元素类型、操作器
void List<T>::traverse(VST & visit)//借助函数对象机制遍历
{for(ListNodePosi(T) p = header -> succ; p != trailer; p = p -> succ) visit(p -> data);}
~~~

## 3.有序列表

---

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

---

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

## 4.选择排序

---

### 4.1 构思

> 回忆起泡排序：
> * 每经一趟扫描交换，当前的最大元素必然就位
> * 从未排序的元素中挑选出最大者，并使之就位
> * 起泡排序之所以需要O(n&sup2;)时间，是因为：
>> 为挑选每个最大元素M，需做O(n)次比较和O(n)次交换
> * O(n)次比较或许无可厚非，但O(n)次交换绝对没有必要
> * 经O(n)次比较确定M之后，一次交换足矣

---

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

---

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

---

### 4.4 性能

> * 共迭代n次，在第k次迭代中：
>> * selectMax()为Θ(n - k)
>> * remove()和insertB()均为O(1)
> * 故总体复杂度应为Θ(n&sup2;)
> * 尽管如此，元素移动操作远远少于起泡排序
> * 也就是说，Θ(n&sup2;)主要来自于元素比较操作
> * 利用高级数据结构，selectMax()可改进至O(logn)
> * 当然，如此立即可以得到O(nlogn)的排序算法

## 5.插入排序

---

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

---

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

---

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

---

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

## 6.归并排序

---

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

---

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

# 栈与队列 Stack & Queue

## 1.栈接口与实现

---

### 1.1 操作与接口

> * 栈（stack）是受限的序列：
>> * 只能在栈顶（top）插入和删除
>> * 栈底（bottom）为盲端
> * 基本接口：
>> * size() / empty()
>> * push() / pop() / top()
> * 后进先出（LIFO）
> * 先进后出（FILO）
> * 扩展接口：getMax()等等

---

### 1.2 实现

~~~CPP

~~~

## 2.栈与递归

---













