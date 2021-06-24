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

# vector.hpp & vector.cpp
## 类Vector
### protected成员变量
- `Memory<double> data`：储存向量数据的Memory对象data
- `int size`：向量的长度
### public成员函数
#### 构造函数和析构函数
- `Vector()`：默认构造函数，重置data为NULL，向量长度置为0
- `Vector(const Vector &v)`：拷贝构造函数，为data新分配内存并将v的数据到拷贝到data中
- `explicit Vector(int s)`：创建长度为s的向量，数据并不初始化
- `Vector(double *_data, int _size)`：将_data指向的长为_size的double型数组包装为向量，此向量并不拥有这个数组
- `Vector(int size_, MemoryType mt)`：创建长度为s，data内存类型为mt的向量
- `virtual ~Vector()`：删除data
#### 对向量的size和data进行设置
- `void SetSize(int s)`：将向量的长度重置为s，如果有必要将重新生成data对象
- `void SetSize(int s, MemoryType mt)`：将向量的长度重置为s，并将对象data的内存类型重置为mt，如果有必要重新生成data对象
- `void SetSize(int s, Vector &v)`：将向量的长度重置为s，并将对象data的内存类型重置为向量v中数据的内存类型，如果有必要重新生成data对象
- `void SetData(double *d)`：用d指向的数组设置data，此向量将不拥有d
- `void SetDataAndSize(double *d, int s)`：用d指向的数组的前s个数据设置data，此向量将不拥有d
- `void NewDataAndSize(double *d, int s)`：先重置向量，用d指向的数组的前s个数据设置data，此向量将不拥有d
- `void NewMemoryAndSize(const Memory<double> &mem, int s, bool own_mem)`：先重置向量，令data=mem，size=s，如果own_mem为假，此向量将不拥有mem

#### 对向量的赋值
- `Vector &operator=(const double *v)`：从v中拷贝size个数据到向量中
- `Vector &operator=(const Vector &v)`：拷贝赋值函数，从v中做深度拷贝
- `Vector &operator=(double value)`：将向量每个元素置为value<font color=yellow>实现看不懂</font>
- `void MakeRef(Vector &base, int offset, int s)`：将向量重置为外部向量base的偏移量为offset，长度为s的片段的引用
- `void MakeRef(Vector &base, int offset)`：将向量重置为外部向量base的偏移量为offset，长度为size的片段的引用
- `void Randomize(int seed = 0)`：将向量各个元素的值随机得选取（以seed为srand的输入，如果seed为0，则取seed=(int)time(0)）
- `void SetVector(const Vector &v, int offset)`：

#### 向量的运算
- `double operator*(const double *v) const`：与doble数组v做点乘
- `double operator*(const Vector &v) const`：与Vector对象v做点乘<font color=yellow>需要对不同内存进行判断和操作，在对mfem内存管理理解更深之后再看</font>
- `Vector &operator*=(double c)`：将向量中每个元素都乘以c<font color=yellow>实现看不懂</font>
- `Vector &operator/=(double c)`：将向量中每个元素都除以c<font color=yellow>实现看不懂</font>
- `Vector &operator-=(double c)`：将向量中每个元素都减c<font color=yellow>实现看不懂</font>
- `Vector &operator-=(const Vector &v)`：将向量和外部向量v的对应元素相减<font color=yellow>实现看不懂</font>
- `Vector &operator+=(double c)`：将向量中每个元素都加c<font color=yellow>实现看不懂</font>
- `Vector &operator+=(const Vector &v)`：将向量和外部向量v的对应元素相加<font color=yellow>实现看不懂</font>
- `Vector &Add(const double a, const Vector &Va)`：(*this) += a * Va
- `Vector &Set(const double a, const Vector &x)`：(*this) = a * x
- `void Neg()`：(\*this) = -(\*this)
- `void median(const Vector &lo, const Vector &hi)`：(\*this) = median((\*this),lo,hi)，其中median代表逐元素得取位于中间的值，方法中假设lo <= hi

#### 最值、范数、距离等方法
- `double Norml2() const`：返回向量的l_2范数
- `double Normlinf() const`：返回向量的l_infinity范数
- `double Norml1() const`：返回向量的l_1范数
- `double Normlp(double p) const`：返回向量的l_p范数
- `double Max() const`：返回向量元素的最大值
- `double Min() const`：返回向量元素的最小值
- `double Sum() const`：返回向量元素之和
- `double DistanceSquaredTo(const double *p)`：返回此向量和p之间欧式距离的平方
- `double DistanceTo(const double *p) const`：返回此向量和p之间的欧氏距离
- `int CheckFinite() const`：返回向量各元素中NaN和+/-Inf的个数
- `void Swap(Vector &other)`：互换向量与外部向量other

#### 对向量信息的读取
- `int Size() const`：返回size
- `int Capacity() const`：返回data占据的内存长度
- `double *GetData() const`：返回指向向量数据头部的指针
- `operator double *()`：类型转换：Vector-to-double *，对data进行转换。调用格式为`(double *)VecObj`，使向量对象可以使用运算符[]索引
- `operator const double *() const`：类型转换：Vector-to-const double *，对data进行转换。调用格式为`(const double *)VecObj`，使向量对象可以使用运算符[]索引
- `Memory<double> &GetMemory()`：返回data的引用
- `const Memory<double> &GetMemory() const`：上条的const形式
- `double &Elem(int i)`：访问第i个元素
- `const double &Elem(int i) const`：上条的const形式
- `double &operator()(int i)`：访问第i个元素
- `const double &operator()(int i) const`：上条的const形式

#### 与子向量相关的方法
- `void GetSubVector(const Array<int> &dofs, Vector &elemvect) const`：提取子向量，如果令elemvect的长度为s，假定dofs的长度大于s，那么elemvect[i] = dofs[i] >= 0?(\*this)[dofs[i]] : -(\*this)[-dofs[i]-1]（i < s）
- `void GetSubVector(const Array<int> &dofs, double *elem_data) const`：与上条功效相同，只是输入的elem_data是指向double型数组
- `void SetSubVector(const Array<int> &dofs, const double value)`：设置向量中某些元素的值，当dofs[i] >= 0时，(\*this)[dofs[i]] = value，否则(\*this)[-dofs[i]-1] = -value
- `void SetSubVector(const Array<int> &dofs, const Vector &elemvect)`：设置向量中某些元素的值，当dofs[i] >= 0时，(\*this)[dofs[i]] = elemvect[i]，否则(\*this)[-dofs[i]-1] = -elemvect[i]
- `void SetSubVector(const Array<int> &dofs, double *elem_data)`：与上条功效相同，只是输入的elem_data是指向double型数组
- `void AddElementVector(const Array<int> & dofs, const Vector & elemvect)`：当dofs[i] >= 0时，(\*this)[dofs[i]] += elemvect[i]，否则(\*this)[-dofs[i]-1] += -elemvect[i]
- `void AddElementVector(const Array<int> & dofs, double *elem_data)`：与上条功效相同，只是输入的elem_data是指向double型数组
- `void AddElementVector(const Array<int> & dofs, const double a, const Vector & elemvect)`：当dofs[i] >= 0时，(\*this)[dofs[i]] += a\*elemvect[i]，否则(\*this)[-dofs[i]-1] += -a\*elemvect[i]
- `void SetSubVectorComplement(const Array<int> &dofs, const double val)`：将*this中dofs以外的元素的值置为val

#### 向量的加载和打印
- `void Load(std::istream ** in, int np, int * dim)`：依次在流in[j]中读取dim[j] (j < np)个数据，组成向量（*this）
- `void Load(std::istream &in, int Size)`：从流in中读取Size个数据，组成向量（*this）
- `void Load(std::istream &in)`：首先从流in中读取向量长度s，再读取s个数据，组成向量（*this）
- `void Print(std::ostream &out = mfem::out, int width = 8) const`：将向量打印到流out中，宽度为width
- `void Print_HYPRE(std::ostream &out) const`：将向量以HYPRE的格式打印到流out中

#### 对内存的操作
- `void UseDevice(bool use_dev) const`：如果use_dev为真，将Memory变量data的UseDevice标志设置为真，反之，将data的UseDevice标志设置为假
- `bool UseDevice() const`：返回将Memory变量data的UseDevice标志
- `void MakeDataOwner() const`：设置对象data的（host指针）的拥有权（为真）
- `void Destroy()`：清除向量，释放对象data，设置size为0
- `void SyncMemory(const Vector &v)`：内存同步<font color=yellow>啥用？</font>
- `void SyncAliasMemory(const Vector &v)`：<font color=yellow>啥用？</font>
- `bool OwnsData() const`：返回data对host指针的拥有权
- `void StealData(double **p)`：
- `double *StealData()`：

### 类外方法
- `void add(const Vector &v1, const Vector &v2, Vector &v)`：计算v = v1 + v2
- `void add(const Vector &v1, double alpha, const Vector &v2, Vector &v)`：计算v = v1 + alpha * v2
- `void add(const double a, const Vector &x, const Vector &y, Vector &z)`：计算z = a * (x + y)
- `void add(const double a, const Vector &x, const double b, const Vector &y, Vector &z)`：计算z = a * x + b * y
- `void subtract(const Vector &v1, const Vector &v2, Vector &v)`：计算v = v1 - v2
- `void subtract(const double a, const Vector &x, const Vector &y, Vector &z)`：计算a * (x - y)

------

# matrix.hpp & matrix.cpp
## 类Matrix
- `class Matrix : public Operator`
- 抽象的矩阵类定义，其中的各个（虚）方法对于各个派生的矩阵类，需要具体得定义
### public成员函数
|方法|解释|
|----|----|
| `explicit Matrix(int s)`| 创建一个维数为s的方阵 |
| `explicit Matrix(int h, int w)`| 创建一个h*w的矩阵 |
| `bool IsSquare() const`| 判断矩阵是否是方阵 |
| `virtual double &Elem(int i, int j) = 0`| 返回元素a_{ij}的引用 |
| `virtual const double &Elem(int i, int j) const = 0`| 上条的const形式 |
| `virtual MatrixInverse *Inverse() const = 0`| 返回指向矩阵（近似）逆的指针 |
| `virtual void Finalize(int)`| 完成矩阵的初始化 |
| `virtual void Print (std::ostream & out = mfem::out, int width_ = 4) const`| 把矩阵以width的宽度打印到流out中 |
| `virtual ~Matrix()`| 析构函数 |

-------

## 类MatrixInverse
- `class MatrixInverse : public Solver`
- 抽象的矩阵逆类的定义
- 类Matrix的友元类
### public成员函数
| 方法 | 解释 |
| ---- | ---- |
| `MatrixInverse()` ||
| `MatrixInverse(const Matrix &mat)` | |

---------

## 类AbstractSparseMatriix
- `class AbstractSparseMatrix : public Matrix`
- 抽象的稀疏矩阵类的定义
### public成员函数
| 方法 | 解释 |
| ---- | ---- |
| `explicit AbstractSparseMatrix(int s = 0)` | 创建一个维数为s的稀疏方阵 |
| `explicit AbstractSparseMatrix(int h, int w)` | 创建一个h*w的稀疏矩阵 |
| `virtual int NumNonZeroElems() const = 0` | 返回稀疏矩阵非零元的个数 |
| `virtual int GetRow(const int row, Array<int> &cols, Vector &srow) const = 0` ||
| `virtual void EliminateZeroRows(const double threshold = 1e-12) = 0` ||
| `virtual void Mult(const Vector &x, Vector &y) const = 0` |矩阵-向量乘 y = A*x|
| `virtual void AddMult(const Vector &x, Vector &y, const double val = 1.) const = 0` | 矩阵-向量乘 y = y + val\*A\*x |
| `virtual void MultTranspose(const Vector &x, Vector &y) const = 0` | 转置矩阵-向量乘 y = A'*x |
| `virtual void AddMultTranspose(const Vector &x, Vector &y, const double val = 1.) const = 0` | 转置矩阵-向量乘 y = y + val\*A'\*x |
| `virtual ~AbstractSparseMatrix()` | 析构函数 |

-------

# densemat.hpp & densemat.cpp
## 类DenseMatrix
- `class DenseMatrix : public Matrix`
- 用于表示按列存储的稠密矩阵
### private成员变量
| 变量 | 解释 |
| ---- | ---- |
| `Memory<double> data` ||
### public成员函数
#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `DenseMatrix()` | 默认构造函数，设置data=NULL，height=width=0 |
| `DenseMatrix(const DenseMatrix &m)` | 拷贝构造函数 |
| `explicit DenseMatrix(int s)` | 创建一个s维的方阵，矩阵元素的值初始化为0 |
| `DenseMatrix(int m, int n)` | 创建一个m*n的矩阵，矩阵元素的值初始化为0 |
| `DenseMatrix(const DenseMatrix &mat, char ch)` | 创建一个矩阵为mat的转置 |
| `DenseMatrix(double *d, int h, int w)` | 用一个指向数组的指针d创建矩阵，height=h，width=w，并且矩阵并不拥有指针d |
| `virtual ~DenseMatrix()` | 释放分配给data的内存 |
<br/> 

#### 读取和设置height、width和data
| 方法 | 解释 |
| ---- | ---- |
| `int Size() const` | 方法`Operator::Width()`的别名，返回width |
| `double *Data() const` | 将data类型转换为double *返回 |
| `double *GetData() const` | 同上 |
| `Memory<double> &GetMemory()` | 返回data |
| `const Memory<double> &GetMemory() const` | 上条的const形式 |
| `double &operator()(int i, int j)` | 返回a_{i,j}的引用 |
| `const double &operator()(int i, int j) const` | 上条的const形式 |
| `virtual double &Elem(int i, int j)` | 返回a_{i,j}的引用 |
| `virtual const double &Elem(int i, int j) const` | 上条的const形式 |
| `bool OwnsData() const` | 返回data的（host指针）的拥有权标志 |
| `int CheckFinite() const` | 统计矩阵中 NaN \ Inf \ -Inf 的个数并返回 |
| `long MemoryUsage() const` | 返回矩阵所占用内存大小 |
| `void UseExternalData(double *d, int h, int w)` | 使用指向数组的指针d设置矩阵，height=h，width=w，并且矩阵并不拥有指针d。注意到这个方法不能在矩阵拥有自己的数据时调用 |
| `void Reset(double *d, int h, int w)` | 使用指向数组的指针d设置矩阵，height=h，width=w，并且矩阵并不拥有指针d。与上一个方法不同的是，当矩阵拥有自己的数据时，方法将先释放相应内存 |
| `void ClearExternalData()` | 重置data，将矩阵的行列置为0。注意到这个方法并不释放任何内存，从而不能在矩阵拥有自己的数据时调用 |
| `void Clear()` | 释放data，将矩阵的行列置为0。与上个方法的区别是本方法将释放data拥有的内存（如果存在的话） |
| `void SetSize(int s)` | SetSize(s,s) |
| `void SetSize(int h, int w)` | 将矩阵的尺寸更改为h*w，如果内存不够，将重新分配内存并且将矩阵元素的值初始化为0，否则不对分配的内存操作 |
| `void Set(double alpha, const double *A)` | 设置矩阵为 alpha * A，假设A的维数与矩阵相同并且是按列排列的 |
| `void Set(double alpha, const DenseMatrix &A)` | 设置矩阵为 alpha * A，如果有必要将会重新为矩阵分配内存|
| `DenseMatrix &operator=(double c)` | 将矩阵元素的值都设置为c |
| `DenseMatrix &operator=(const double *d)` | d指向的数组的值设置矩阵元素（按列） |
| `DenseMatrix &operator=(const DenseMatrix &m)` | 用外部矩阵m来设置矩阵的行、列和各元素的值 |
<br/> 

#### 矩阵行列、对角的读取和赋值
| 方法 | 解释 |
| ---- | ---- |
| `void GetRow(int r, Vector &row) const` | 提取矩阵第r行的元素到row中 |
| `void GetColumn(int c, Vector &col) const` | 提取矩阵第c列元素到col中 |
| `double *GetColumn(int col)` | 返回指向第col列第一个元素的指针 |
| `const double *GetColumn(int col) const` | 上条的const形式 |
| `void GetColumnReference(int c, Vector &col)` |  |
| `void SetRow(int r, const double* row)` | 用row指向的数组中的值设置矩阵的第r行元素 |
| `void SetRow(int r, const Vector &row)` | 用向量row设置矩阵的第r行元素 |
| `void SetCol(int c, const double* col)` | 用col指向的数组中的值设置矩阵的第c列元素 |
| `void SetCol(int c, const Vector &col)` | 用向量col设置矩阵的第c列元素 |
| `void SetRow(int row, double value)` | 将矩阵第row行元素的值设置为value |
| `void SetCol(int col, double value)` | 将矩阵第col列元素的值设置为value |
| `void GetDiag(Vector &d) const` | 提取方阵的对角线元素到d中 |
| `void Getl1Diag(Vector &l) const` | 计算矩阵每一行的l_1范数，储存到l中 |
| `void GetRowSums(Vector &l) const` | 计算矩阵每一行的和，储存到l中 |
| `void Diag(double c, int n)` | 创建一个n维的对角阵，对角元为c |
| `void Diag(double *diag, int n)` | 创建一个n维的对角阵，对角元从diag指向的数组中提取 |
<br/> 

#### 用外部矩阵的（部分行列对应的）元素设置矩阵的（部分行列对应的）元素
| 方法 | 解释 |
| ---- | ---- |
| `void CopyRows(const DenseMatrix &A, int row1, int row2)` | (*this) = A(row1:row2,:) |
| `void CopyCols(const DenseMatrix &A, int col1, int col2)` | (*this) = A(:,col1:col2) |
| `void CopyMN(const DenseMatrix &A, int m, int n, int Aro, int Aco)` | (*this) = A(Aro:Aro+m-1,Aco:Aco+n-1) |
| `void CopyMN(const DenseMatrix &A, int ro, int co)` | (*this)(ro:ro+A.height-1,co:co+A.width-1) = A |
| `void CopyMNt(const DenseMatrix &A, int ro, int co)` | (*this)(ro:ro+A.height-1,co:co+A.width-1) = A' |
| `void CopyMN(const DenseMatrix &A, int m, int n, int Aro, int Aco, int ro, int co)` | (*this)(ro:ro+m-1,co:co+n-1) = A(Aro:Aro+m-1,Aco:Aco+n-1) |
| `void CopyMNDiag(double c, int n, int ro, int co)` | (*this)(ro:ro+n-1,co:co+n-1) = diag(c) （即对角线全为c的对角阵）|
| `void CopyMNDiag(double *d, int n, int ro, int co)` | (*this)(ro:ro+n-1,co:co+n-1) = diag(d) （即对角线来自于d指向的数组的对角阵）|
| `void CopyExceptMN(const DenseMatrix &A, int m, int n)` | (*this) = A([1:m-1,m+1:end],[1:n-1,n+1:end]) |
| `void AddMatrix(DenseMatrix &A, int ro, int co)` | (*this)(ro:ro+A.Height-1,co:co+A.Weight-1) += A |
| `void AddMatrix(double a, const DenseMatrix &A, int ro, int co)` | (*this)(ro:ro+A.Height-1,co:co+A.Weight-1) += a * A |
| `void AddToVector(int offset, Vector &v) const` | v[offset+i] += data[i], 0 <= i < height * width |
| `void GetFromVector(int offset, const Vector &v)` | data[i] = v[offset+i], 0 <= i < height * width |
<br/> 

#### 矩阵、向量运算，矩阵的自加自减
| 方法 | 解释 |
| ---- | ---- |
| `void Mult(const double *x, double *y) const` | 矩阵-向量乘 y = (\*this)*x，相应维数应该匹配 |
| `virtual void Mult(const Vector &x, Vector &y) const` | 矩阵-向量乘 y = (\*this)*x，相应维数应该匹配 |
| `void MultTranspose(const double *x, double *y) const` | 转置矩阵-向量乘 y = (\*this)'*x，相应维数应该匹配 |
| `virtual void MultTranspose(const Vector &x, Vector &y) const` | 转置矩阵-向量乘 y = (\*this)'*x，相应维数应该匹配 |
| `void AddMult(const Vector &x, Vector &y) const` | y += A*x |
| `void AddMultTranspose(const Vector &x, Vector &y) const` | y += A'*x |
| `void AddMult_a(double a, const Vector &x, Vector &y) const` | y += a * A * x |
| `void AddMultTranspose_a(double a, const Vector &x, Vector &y) const` | y += a * A' * x |
| `double InnerProduct(const double *x, const double *y) const` | 计算A内积 y' * A * x |
| `double InnerProduct(const Vector &x, const Vector &y) const` | 计算A内积 y' * A * x |
| `void Add(const double c, const DenseMatrix &A)` | 将 c * A 加到矩阵上，矩阵维数需要匹配 |
| `DenseMatrix &operator+=(const double *m)` | *this自加m代表的矩阵 |
| `DenseMatrix &operator+=(const DenseMatrix &m)` | *this += m |
| `DenseMatrix &operator-=(const DenseMatrix &m)` | *this -= m |
| `DenseMatrix &operator*=(double c)` | *this *= c |
<br/> 

#### 对矩阵进行某些变换
| 方法 | 解释 |
| ---- | ---- |
| `void Neg()` | (*this) = -(*this) |
| `void LeftScaling(const Vector & s)` | 矩阵左乘 diag(s) |
| `void InvLeftScaling(const Vector & s)` | 矩阵左乘 1./diag(s) |
| `void RightScaling(const Vector & s)` | 矩阵右乘 diag(s) |
| `void InvRightScaling(const Vector & s)` | 矩阵右乘 1./diag(s) |
| `void SymmetricScaling(const Vector & s)` | 矩阵两端分别乘 diag(s) |
| `void InvSymmetricScaling(const Vector & s)` | 矩阵两端分别乘 1./diag(s) |
| `void Invert()` | 将当前矩阵替换为其逆矩阵 |
| `void SquareRootInverse()` | 将当前矩阵替换为其平方根逆矩阵 |
| `void Transpose()` | (\*this) = (\*this)' |
| `void Transpose(const DenseMatrix &A)` | (*this) = A' |
| `void Symmetrize()` | (\*this) = 1/2 * ((\*this) + (\*this)') |
| `void Lump()` | 将矩阵非对角的元素置为0，而对角元则置为所在行所有元素之和 |
| `void AdjustDofDirection(Array<int> &dofs)` | 如果 (dofs[i] < 0 并且 dofs[j] >= 0) 或者 (dofs[i] >= 0 并且 dofs[j] < 0) 那么 (\*this)(i,j) = -(\*this)(i,j) |
| `void Threshold(double eps)` | 将满足 abs(a_ij) <= eps 的元素置为 0 |
<br/> 

#### 计算矩阵相关的一些值（如行列式、范数、特征值等）
| 方法 | 解释 |
| ---- | ---- |
| `double operator*(const DenseMatrix &m) const` | 返回矩阵内积 tr(A'*B) |
| `double Trace() const` | 返回方阵的迹 |
| `double Det() const` | 计算矩阵的行列式 |
| `void Norm2(double *v) const` | 计算矩阵每一列的2范数，储存在v中 |
| `double MaxMaxNorm() const` | 计算矩阵的最大-最大范数 $\max_{i,j}a_{ij}$ |
| `double FNorm() const` | 计算矩阵的Frobenius范数 |
| `double FNorm2() const` | 计算矩阵的Frobenius范数的平方 |
| `void Eigenvalues(Vector &ev)` | 计算矩阵的特征值，储存在ev中 |
| `void Eigenvalues(Vector &ev, DenseMatrix &evect)` | 计算矩阵的特征值和特征向量，分别储存在ev和evect中 |
| `void Eigensystem(Vector &ev, DenseMatrix &evect)` | 同上条 |
| `void Eigenvalues(DenseMatrix &b, Vector &ev)` | 计算广义特征值问题 A * x = ev * b * x的特征值，储存在ev中 |
| `void Eigenvalues(DenseMatrix &b, Vector &ev, DenseMatrix &evect)` | 计算广义特征值问题 A * x = ev * b * x的特征值和特征向量，分别储存在ev和evect中 |
| `void Eigensystem(DenseMatrix &b, Vector &ev, DenseMatrix &evect)` | 同上条 |
| `void SingularValues(Vector &sv) const` | 计算矩阵的奇异值，储存在sv中（需要LAPACK支持） |
| `int Rank(double tol) const` | 通过计算矩阵的奇异值，根据阈值tol计算矩阵的秩 |
| `double CalcSingularvalue(const int i) const` | 返回1、2、3维方阵的第i个奇异值（按照降序） |
| `void CalcEigenvalues(double *lambda, double *vec) const` | 计算一个2维或3维对称矩阵的特征值和特征向量，分别储存在lamba和vec（按矩阵的列储存）指向的数组中 |
<br/> 

#### 矩阵的打印
| 方法 | 解释 |
| ---- | ---- |
| `virtual void Print(std::ostream &out = mfem::out, int width_ = 4) const` | 将矩阵以宽度 width_ 打印到流 out 中 <font color=yellow>具体格式待补充</font> |
| `virtual void PrintMatlab(std::ostream &out = mfem::out) const` | 将矩阵以Matlab格式打印到流 out 中 <font color=yellow>具体格式待补充</font> |
| `virtual void PrintT(std::ostream &out = mfem::out, int width_ = 4) const` | 将转置矩阵以宽度 width_ 打印到流 out 中 <font color=yellow>具体格式待补充</font> |
<br/> 

#### 快捷方式，用于得到内存读写的途径
| 方法 | 解释 |
| ---- | ---- |
| `const double *Read(bool on_dev = true) const` | `mfem::Read( GetMemory(), TotalSize(), on_dev)`的快捷方式 |
| `const double *HostRead() const` | `mfem::Read(GetMemory(), TotalSize(), false)`的快捷方式 |
| `double *Write(bool on_dev = true)` | `mfem::Write(GetMemory(), TotalSize(), on_dev)`的快捷方式 |
| `double *HostWrite()` | `mfem::Write(GetMemory(), TotalSize(), false)`的快捷方式 |
| `double *ReadWrite(bool on_dev = true)` | `mfem::ReadWrite(GetMemory(), TotalSize(), on_dev)`的快捷方式 |
| `double *HostReadWrite()` | `mfem::ReadWrite(GetMemory(), TotalSize(), false)`的快捷方式 |
<br/> 

#### 待补充
| 方法 | 解释 |
| ---- | ---- |
| `virtual MatrixInverse *Inverse() const` | 返回一个指向逆矩阵的指针 |
| `void TestInversion()` | 测试逆矩阵的准确度 <font color=yellow>具体形式待补充</font> |
| `double Weight() const` |  |
| `void GradToCurl(DenseMatrix &curl)` |  |
| `void GradToDiv(Vector &div)` |  |

--------------------
## 类外方法，主要进行矩阵之间的运算
| 方法 | 解释 |
| ---- | ---- |
| `void Add(const DenseMatrix &A, const DenseMatrix &B, double alpha, DenseMatrix &C)` | C = A + alpha*B |
| `void Add(double alpha, const double *A, double beta,  const double *B, DenseMatrix &C)` | C = alpha\*A + beta\*B |
| `void Add(double alpha, const DenseMatrix &A, double beta,  const DenseMatrix &B, DenseMatrix &C)` | C = alpha\*A + beta\*B |
| `bool LinearSolve(DenseMatrix& A, double* X, double TOL = 1.e-9)` |  |
| `void Mult(const DenseMatrix &B, const DenseMatrix &C, DenseMatrix &A)` | A = B * C |
| `void AddMult(const DenseMatrix &B, const DenseMatrix &C, DenseMatrix &A)` | A += B * C |
| `void AddMult_a(double alpha, const DenseMatrix &b, const DenseMatrix &c, DenseMatrix &a)` | A += alpha * B * C |
| `void CalcAdjugate(const DenseMatrix &a, DenseMatrix &adja)` |  |
| `void CalcAdjugateTranspose(const DenseMatrix &a, DenseMatrix &adjat)` |  |
| `void CalcInverse(const DenseMatrix &a, DenseMatrix &inva)` |  |
| `void CalcInverseTranspose(const DenseMatrix &a, DenseMatrix &inva)` |  |
| `void CalcOrtho(const DenseMatrix &J, Vector &n)` |  |
| `void MultAAt(const DenseMatrix &A, DenseMatrix &AAt)` | AAt = A * A' |
| `void MultADAt(const DenseMatrix &A, const Vector &D, DenseMatrix &ADAt)` | ADAt = A * diag(D) * A' |
| `void AddMultADAt(const DenseMatrix &A, const Vector &D, DenseMatrix &ADAt)` | ADAt += A * diag(D) * A' |
| `void MultABt(const DenseMatrix &A, const DenseMatrix &B, DenseMatrix &ABt)` | ABt = A * B' |
| `void MultADBt(const DenseMatrix &A, const Vector &D, const DenseMatrix &B, DenseMatrix &ADBt)` | ADBt = A * diag(D) * B' |
| `void AddMultABt(const DenseMatrix &A, const DenseMatrix &B, DenseMatrix &ABt)` | ABt += A * B' |
| `void AddMultADBt(const DenseMatrix &A, const Vector &D, const DenseMatrix &B, DenseMatrix &ADBt)` | ADBt += A * diag(D) * B'  |
| `void AddMult_a_ABt(double a, const DenseMatrix &A, const DenseMatrix &B, DenseMatrix &ABt)` | ABt += a * A * B' |
| `void MultAtB(const DenseMatrix &A, const DenseMatrix &B, DenseMatrix &AtB)` | AtB = A' * B |
| `void AddMult_a_AAt(double a, const DenseMatrix &A, DenseMatrix &AAt)` | AAt += a * A * A' |
| `void Mult_a_AAt(double a, const DenseMatrix &A, DenseMatrix &AAt)` | AAt = a * A * A' |
| `void MultVVt(const Vector &v, DenseMatrix &vvt)` | VVt = v * v' |
| `void MultVWt(const Vector &v, const Vector &w, DenseMatrix &VWt)` | VWt = v * w' |
| `void AddMultVWt(const Vector &v, const Vector &w, DenseMatrix &VWt)` | VWt += v * w' |
| `void AddMultVVt(const Vector &v, DenseMatrix &VVt)` | VVt += v * v' |
| `void AddMult_a_VWt(const double a, const Vector &v, const Vector &w, DenseMatrix &VWt);` | VWt += a * v * w' |
| `void AddMult_a_VVt(const double a, const Vector &v, DenseMatrix &VVt)` | VVt += a * v * v' |

----------------------

## 类LUFactors
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |