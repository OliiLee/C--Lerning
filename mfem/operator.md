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

# operator.hpp & operator.cpp
## （基）类Operator
### 类内结构体、枚举和类
- (public) `enum DiagonalPolicy{DIAG_ZERO, DIAG_ONE, DIAG_KEEP}`
  - 定义
- (public) `enum Type`
### protected成员变量
|定义|解释|
|----|----|
| `int height` | 输出的维数/矩阵的行数 |
| `int width` | 输入的维数/矩阵的列数 |
### public成员函数
#### 构造函数和析构函数
|定义|解释|
|----|----|
| `explicit Operator(int s = 0)` | 令height=width=s |
| `Operator(int h, int w)` | 令height=h，width=w |
| `virtual ~Operator()` | |

#### 
|定义|解释|
|----|----|
| `void InitTVectors(const Operator *Po, const Operator *Ri, const Operator *Pi, Vector &x, Vector &b, Vector &X, Vector &B) const` | |
| `int Height()` | 返回height |
| `int NumRows()` | 返回height |
| `int Width()` | 返回width |
| `int NumCols()` | 返回width |
| `virtual MemoryClass GetMemoryClass()` | 返回当前Operator的内存类型（默认为Memory::HOST）|
| `virtual void Mult(const Vector &x, Vector &y) const = 0` | 算子的实现：y=A(x) |
| `virtual void MultTranspose(const Vector &x, Vector &y) const` | 算子转置的实现：y=A^t(x) |
| `virtual Operator &GetGradient(const Vector &x) const` | 计算点x处的梯度算子 |
| `virtual const Operator *GetProlongation() const` | 返回从线性系统向量到算子输入向量的插值算子 |
| `virtual const Operator *GetRestriction() const` | 返回从算子输入向量到线性系统向量的限制算子 |
| `virtual const Operator *GetOutputProlongation() const` | 返回从线性系统向量到算子输出向量的插值算子 |
| `virtual const Operator *GetOutputRestriction() const` | 返回从线性系统向量到算子输出向量的限制算子 |
| `void FormLinearSystem(const Array<int> &ess_tdof_list, Vector &x, Vector &b, Operator* &A, Vector &X, Vector &B, int copy_interior = 0)` | |
| `void FormRectangularLinearSystem(const Array<int> &trial_tdof_list, const Array<int> &test_tdof_list, Vector &x, Vector &b, Operator* &A, Vector &X, Vector &B)` | |
| `void FormSystemOperator(const Array<int> &ess_tdof_list, Operator* &A)` | |
| `void FormRectangularSystemOperator(const Array<int> &trial_tdof_list, const Array<int> &test_tdof_list, Operator* &A)` | |
| `void FormDiscreteOperator(Operator* &A)` | |
| `void PrintMatlab(std::ostream & out, int n = 0, int m = 0) const` | |
| `Type GetType() const` | |