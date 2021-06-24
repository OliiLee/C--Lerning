<style>
table {
    width:100%;
}
table th:first-of-type{
    width:50%;
}
table th:nth-of-type(2){
    width:50%;
}
</style>

# table.hpp & table.cpp
## 结构体 Connection
- `struct Connection`
### 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int from` |  |
| `int to` |  |
### 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `Connection()` |  |
| `Connection(int from, int to)` |  |
| `bool operator== (const Connection &rhs) const` |  |
| `bool operator< (const Connection &rhs) const` |  |

## 类 Table
- `class Table`
- 抽象类 Table 建立两个集合 A 和 B 之间的映射关系（集合 A 中的元素称为 TYPE I 元素而集合 B 中的元素称为 TYPE II 元素），由两个数组 I 和 J 表示， I 长为 size+1 （ size 为集合 A 的大小）而 J 的长度为联系的个数，与 TYPE I 元素 $A_i(0\leq i\leq {\rm size}-1)$ 有联系的 TYPE II 元素的标号分别为 J[k]，$I[i]\leq k<I[i+1]$，换句话说， I[i] 和 I[i+1]-1 分别记录了第 i 个 TYPE I 元素所对应的联系的编号，而 J[k] 中记录了第 k 个联系的末端（即集合 B 中的一个元素）
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int size` | TYPE I 元素的个数 |
| `Memory<int> I, J` | 用于表示映射关系的两个数组 |

### 成员函数
#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `Table()` | 构造一个空表 |
| `Table(const Table &)` | 拷贝构造函数 |
| `Table& operator=(const Table &rhs)` | 赋值函数（深度拷贝） |
| `Table (int dim, int connections_per_row = 3)` | 构造一个表，每个 TYPE I 元素的连接数不多于 connections_per_row ， TYPE I 元素个数为 dim |
| `Table(int nrows, Array<Connection> &list)` | 从一个 Connection 数组出发构造一个表， TYPE I 元素个数为 nrows ，调用`MakeFromList()`完成构造 |
| `void MakeFromList(int nrows, const Array<Connection> &list)` | 从一列 connections 结构体 {(from,to)} 中构造表，其中'from'是一个Type I的序号，'to'是一个Type II序号。这个结构体数列需要是排过序并且没有重复的，也即在调用此方法之前要调用`Array::Sort`和`Array::Unique` |
| `Table (int nrows, int *partitioning)` | 创建一个表，每行仅有一个元素，对应数组 partitioning 的相应的元素，即 I 记录 partitioning 的索引，J 记录此索引对应的 partitioning 的值 |
| `~Table()` | 析构函数，释放 Memory 对象 I 和 J |

#### 读取成员变量I，J的信息
| 方法 | 解释 |
| ---- | ---- |
| `int Size() const` | 获得 Type I 元素的个数 |
| `int Size_of_connections()` | 获取连接的个数 |
| `int operator() (int i, int j) const` | 获得第 i 个 Type I 元素和第 j 个 Type II 元素之间连接的索引 k （即$J[k]=j, I[i]\leq k< I[i+1]$），如果这个连接不存在则返回-1 |
| `void GetRow(int i, Array<int> &row) const` | 获取第 i 行的连接信息，即将与第 i 个 TYPE I 元素相连接的 TYPE II 元素对应的标号保存到 row 中 |
| `int RowSize(int i) const` | 获取第 i 个元素所对应的连接数 |
| `const int *GetRow(int i) const` | 上条的 const 形式 |
| `int *GetRow(int i)` | 获取第 i 个元素所对应的连接数组的首地址（J+I[i]） |
| `int *GetI()` | 以整形数组形式返回对象 I |
| `const int *GetI() const` | 上条的 const 形式 |
| `int *GetJ()` | 以整形数组形式返回对象 J |
| `const int *GetJ() const` | 上条的 const 形式 |
| `Memory<int> &GetIMemory()` | 以 Memory 类型返回对象 I |
| `const Memory<int> &GetIMemory() const` | 上条的 const 形式 |
| `Memory<int> &GetJMemory()` | 以 Memory 类型返回对象 J |
| `const Memory<int> &GetJMemory() const` | 上条的 const 形式 |
| `int Width() const` | 返回 Type II 元素的个数（假定每个 TYPE II 元素均与某个 TYPE I 元素所连接，即 Table 对象表示的映射是满的） |
| `long MemoryUsage() const` | 返回当前表对象所占用的内存 |


#### 对表的操作
| 方法 | 解释 |
| ---- | ---- |
| `void SetSize(int dim, int connections_per_row)` | 设置表的行数（ size ）为dim，每行（最大）连接数为 connections_per_row ，对象 J 的元素均赋值为 -1 |
| `void SetDims(int rows, int nnz)` | 设置表的行数（ size ）为 rows ，连接数为 nnz ，除了 I[0]=0 和 I[rows]=nnz 外不进行初始化 |
| `void SetIJ(int *newI, int *newJ, int newsize = -1)` | 将 I 和 J 替换为 newI 和 newJ ，如果 newsize<0 ，那么当前 Table 对象的 size 不变 |
| `void SortRows()` | 将每一个 Type I 元素对应的连接按照相应的 TYPE II 元素的大小进行排序（从小到大），即将 J[I[i]] 至 J[I[i]-1] 的元素进行排列 |
| `int Push( int i, int j )` | 建立 Type I 元素 i 和 Type II 元素 j 的联系，返回这个联系的索引 k（即$J[k]=j$），当联系无法建立时将返回-1。当`Finalize()`调用后，可能没有足够的内存建立这个联系 |
| `void Finalize()` | 完成表的初始化，刨除数组J中的-1。从某种角度讲，这个方法将“固定”当前的对象，`Push()`将无法正常工作 |
| `void Copy(Table & copy) const` | 得到当前表的一个拷贝 copy |
| `void Swap(Table & other)` | 将当前表和外部表 other 进行交换 |
| `void Clear()` | 释放分配的内存，重置当前表 |


#### 表的打印、保存和加载
| 方法 | 解释 |
| ---- | ---- |
| `void Print(std::ostream & out = mfem::out, int width = 4) const` | 打印表， width 为每行能容纳的最大的数据个数 |
| `void PrintMatlab(std::ostream & out) const` | 以 Matlab 风格打印表 |
| `void Save(std::ostream &out) const` | 保存当前表到流 out 中，即保存 size 、数组 I 和数组 J （格式见源码） |
| `void Load(std::istream &in)` | 从流 in 中加载表（流的格式见源码，与`Save()`得到的相同） |

## 对table的操作（类外函数）
| 方法 | 解释 |
| ---- | ---- |
| `template <> inline void Swap<Table>(Table &a, Table &b)` | 模板函数 Swap 对类 Table 的特例，用于交换两个 Table 对象 a 和 b |
| `void Transpose (const Table &A, Table &At, int _ncols_A = -1)` | 得到表 A 的转置 At ， _ncols_A 代表At的行数，即对前 _ncols_A 个 Type II 的元素建立转置（可能不安全？见源码），如果 _ncols_A 小于0，将对整个表建立转置 |
| `Table * Transpose (const Table &A)` | 得到表 A 的转置，返回指向转置的 Table 型指针 |
| `void Transpose(const Array<int> &A, Table &At, int _ncols_A = -1)` | 得到 Array<int> 型的对象 A 的转置 At （在一定程度上， A 可以看做是一个特殊的表）， _ncols_A 代表 At 的行数，即对前 _ncols_A 个 Type II 型元素建立转置（可能不安全？见源码），如果 _ncols_A 小于0，将对整个表建立转置 |
| `void Mult (const Table &A, const Table &B, Table &C)` |  Table 类型可以作为保存 bool 型稀疏矩阵指标的数据结构（因为无法储存稀疏矩阵的各数值）， Mult 计算两个 bool 型的稀疏矩阵的乘积，保存在 C 中 |
| `Table * Mult (const Table &A, const Table &B)` | 计算两个 bool 型稀疏矩阵的乘积，返回指向结果的 Table 型指针 |


---

## 类STable
- 对称表，即Type I和Type II是相同类型的，也即建立相同的集合上不同元素的连接关系，对元素a和元素b，默认对a&#60;=b建立连接关系
- `class STable : public Table`
### 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `STable (int dim, int connections_per_row = 3)` | 调用`Table(int dim, int connections_per_row)` |
| `~STable()` | 默认析构函数 |
| `int STable::operator() (int i, int j) const` | 当i&#60;j时，返回`Table::operator()(i,j)`，否则返回`Table::operator()(j,i)` |
| `int STable::Push( int i, int j )` | 当i&#60;j时，返回`Table::Push()(i,j)`，否则返回`Table::Push()(j,i)` |


---

## 类DSTable
- 由一列Node对象构成的表，建立相同的集合上不同元素的连接关系，对元素a和元素b，默认对a&#60;=b建立连接关系
### 内部类
- `class Node`(private)：结点类
  - 成员变量 
    - `Node *Prev`，`int Column,Index`：Prev储存前一个结点的地址，Column表示当前行与Column列建立了联系，Index保存了这个联系的序号
- `class RowIterator`(public)：DSTable对象的行迭代器，是从后向前迭代的
  - 成员变量
    - `Node *n`
  - 成员函数
    - `RowIterator (const DSTable &t, int r)`：用DSTable对象t的第r行初始化迭代器
    - `int operator!()`：判断指针n是否为空
    - `void operator++()`：将n更新为n.Prev
    - `int Column()`：返回n.Column
    - `int Index()`：返回n.Index
    - `void SetIndex(int new_idx)`：将n.Index更新为new_idx
### 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int NumRows, NumEntries` | 分别储存行数（元素的个数）和连接的个数 |
| `Node **Rows` | 类似于单向链表，DSTable的主要数据结构 |

### 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `DSTable(int nrows)` | 构造一个DSTable，其包含nrows行（即Rows长度为nrows，NumRows = nrows），每一行初始化为空指针，且不包含任何元素（NumEntries=0） |
| `int NumberOfRows() const` | 获得行数NumRows |
| `int NumberOfEntries() const` | 获得所有连接数 |
| `int Push(int a, int b)` | 建立第a个元素和第b个元素之间的连接，如果这个连接已经存在，则返回连接编号 |
| `int operator()(int a, int b) const` | 返回第a个元素和第b个元素之间连接的编号，非法操作将返回-1 |
| `~DSTable()` | 析构函数，释放分配的内存 |


---

# stable3d.hpp & stable3d.cpp
## 类STable3DNode
- 建立一个链表，这个链表实际上保存了某一个行指标对应的所有联系
### 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `STable3DNode *Prev` | 保存指向前一个结点的指针（链表） |
| `int Column, Floor, Number` | 分别保存联系的列指标、层指标以及给联系分配的值 |


## 类STable3D
- 建立一个3维对称表，对每一个（行）元素建立一个链表保存包含此元素的所有联系，对称意味着对于行、列、层指标(r,c,f)，对它们的任意置换得到的新的指标对应了相同的联系，同时，要求行、列、层指标是互不相同的
### 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int Size, NElem` | 分别保存（行）元素的个数和联系的个数 |
| `STable3DNode **Rows` | 保存指向每个链表表头的指针 |
### 成员函数
#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `STable3D (int nr)` | 将列数（Size）设置为nr，并为Rows开辟内存并初始化为空指针，元素个数NElem初始化为0 |
| `~STable3D ()` | 释放分配的内存 |
#### 信息的读取
| 方法 | 解释 |
| ---- | ---- |
| `int operator() (int r, int c, int f) const` | 返回为第r行、第c列、第f层的联系所分配的值Number(见类STable3DNode)，如果这个联系不存在，将报错 |
| `int operator() (int r, int c, int f, int t) const` | 返回数(r,c,f,t)中最小的三个数为行、列、层索引得到的联系对应的值（调用`operator()()`），如果这个联系不存在，则返回-1 |
| `int NumberOfElements()` | 返回联系的个数NElem |
| `int Index (int r, int c, int f) const` | 返回为第r行、第c列、第f层的联系所分配的值Number(见类STable3DNode)，如果这个联系不存在，将返回-1 |

#### 建立联系
| 方法 | 解释 |
| ---- | ---- |
| `int Push (int r, int c, int f)` | 建立第r行、第c列、第f层元素之间的联系，并为这个联系分配的值（Number）NElem，不管联系是否存在，均返回为这个联系分配的值Number |
| `int Push4 (int r, int c, int f, int t)` | 以数(r,c,f,t)中最小的三个数为行、列、层索引建立联系（调用`Push`），返回给这个联系分配的值（`Push`的返回值） |

#### 表打印
| 方法 | 解释 |
| ---- | ---- |
| `void Print(std::ostream &out = mfem::out) const` | 将表打印到流out中，依次打印联系数NElem和每个联系对应的行、列、层指标以及为这个联系分配的值 |




