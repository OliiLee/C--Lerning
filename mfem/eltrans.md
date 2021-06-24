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

# eltrans.hpp & eltrans.cpp
## （基）类ElementTransformation
- `class ElementTransformation`
- 用于表示参考元到实际单元之间变换的抽象类，主要功能是提供点的变换和逆变换的方法，并且计算与变换有关的算子和值
### 类内类、枚举和结构体
```C++
// protected 枚举，用于标识与变换相关的各个矩阵的状态（是否需要计算），具体见成员变量 EvalState 的解释
enum StateMasks
{
    JACOBIAN_MASK = 1,
    WEIGHT_MASK   = 2,
    ADJUGATE_MASK = 4,
    INVERSE_MASK  = 8,
    HESSIAN_MASK  = 16
};
```

```C++
// public 枚举，用于表示 ElementTransformation 对象对应的单元的类型
enum
{
    ELEMENT     = 1,//单元
    BDR_ELEMENT = 2,//边界单元
    EDGE        = 3,//边
    FACE        = 4,//面
    BDR_FACE    = 5//边界面（边界单元的面）
};
```

### protected成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const IntegrationPoint *IntPoint` | 位于参考元中的一个积分点 |
| `DenseMatrix dFdx, adjJ, invJ` | dFdx 是变换在积分点 IntPoint 上的的 Jacobian 矩阵， adjJ 是矩阵 dFdx 的伴随矩阵， invJ 是矩阵 dFdx 的逆矩阵|
| `DenseMatrix d2Fdx2` | d2Fdx2 是变换在积分点 IntPoint 上的 Hessian 矩阵 |
| `double Wght` | Wght 代表矩阵 dFdx 的权，即$\sqrt{\lvert J^T J \rvert}$ |
| `int EvalState` | 变换相关矩阵状态（是否需要计算）的位标识符，例如，如果 EvalState = 21(0b10101) ，意味着变换的 Jacobian 矩阵、 Jacobian 矩阵的伴随矩阵、 Hessian 矩阵已经计算完毕而变换的 Jacobian 矩阵的权、 Jacobian 矩阵的逆需要计算|
| `Geometry::Type geom` | 用于记录参考元的几何类型 |
</br>

### public成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int Attribute` | 单元分类 |
| `int ElementNo` | 单元标号 |
| `int ElementType` | 单元类型 |
</br>

### public成员函数
</br>

#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `ElementTransformation()` | 默认构造函数，置 IntPoint 为 NULL ， EvalState 为0， geom 为 Geometry::INVALID ， Attribute 为-1，  ElementNo 为-1 |
| `virtual ~ElementTransformation()` | （虚函数）析构函数 |
</br>

#### 设置或读取 ElementTransformation 有关的信息
| 方法 | 解释 |
| ---- | ---- |
| `void Reset()` | 置 EvalState 为0，在以后相关函数的调用中重新计算 |
| `void SetIntPoint(const IntegrationPoint *ip)` | 将积分点 IntPoint 置为 ip ，同时 EvalState 置为0 |
| `const IntegrationPoint &GetIntPoint()` | 返回当前积分点 *IntPoint 的 const 引用 |
| `Geometry::Type GetGeometryType() const` | 返回参考元的几何类型 geom |
| `int GetDimension() const` | 返回参考元的拓扑维数 |
| `virtual int GetSpaceDim() const = 0` | 返回实际单元所在的欧式空间的维数 |
</br>

#### 点变换和逆变换有关的方法
| 方法 | 解释 |
| ---- | ---- |
| `virtual void Transform(const IntegrationPoint &, Vector &pt) = 0` | （纯虚函数）将参考元上积分点变换到实际单元上 |
| `virtual void Transform(const IntegrationRule &, DenseMatrix &) = 0` | （纯虚函数）将参考元上的一系列积分点（用一个 IntergrationRule 对象表示）变换到实际单元上 |
| `virtual void Transform(const DenseMatrix &matrix, DenseMatrix &result) = 0` | （纯虚函数）将参考元上的一系列积分点（用一个 DenseMatrix 对象表示）变换到实际单元上 |
| `virtual int TransformBack(const Vector &pt, IntegrationPoint &ip) = 0` | （虚函数）将实际空间上的点 pt 变换为参考元上的点 ip |
</br>

#### 计算与变换相关的算子或值
| 方法 | 解释 |
| ---- | ---- |
| `const DenseMatrix &Jacobian()` | 返回变换在积分点 IntPoint 上的 Jacobi 矩阵 dFdx 的 const 引用，如果 EvalState 认为 Jacobi 矩阵没有被计算（即便 dFdx  存在），那么将（重新）计算它 |
| `const DenseMatrix &Hessian()` | 返回变换在积分点IntPoint 上的 Hessian 矩阵 d2Fdx2 的 const 引用，如果 EvalState 认为 Hessian 矩阵没有被计算（即便 d2Fdx2 存在），那么将（重新）计算它 |
| `double Weight()` | 返回变换在积分点 IntPoint 上的 Jacobi 矩阵的权 Wght ，如果 EvalState 认为 Jacobi 矩阵的权没有被计算（即便 Wght 存在），那么将（重新）计算它 |
| `const DenseMatrix &AdjugateJacobian()` | 返回变换在积分点 IntPoint 上的 Jacobi 矩阵的伴随矩阵 adjJ 的 const 引用，如果 EvalState 认为 Jacobi 矩阵的伴随矩阵没有被计算（即便 adjJ  存在），那么将（重新）计算它 |
| `const DenseMatrix &InverseJacobian()` | 返回变换在积分点 IntPoint 上的 Jacobi 矩阵的逆矩阵 invJ 的 const 引用，如果 EvalState 认为 Jacobi 矩阵的逆矩阵没有被计算（即便 invJ  存在），那么将（重新）计算它 |
</br>

#### 阶数相关
| 方法 | 解释 |
| ---- | ---- |
| `virtual int Order() const = 0` | （纯虚函数）返回变换的阶数 |
| `virtual int OrderJ() const = 0` | （纯虚函数）返回变换 Jacobian 元素的阶数 |
| `virtual int OrderW() const = 0` | （纯虚函数）返回变换 Jacobian 的行列式（ weight ）的阶数 |
| `virtual int OrderGrad(const FiniteElement *fe) const = 0` | （纯虚函数）返回矩阵 ${\rm adj}(J)^T\nabla f_i$ 中元素的阶数 |

-------------

## 类 InverseElementTransformation
- `class InverseElementTransformation`
- 对一个给定的 ElementTransformation 对象，表示其逆变换的类，主要的功能是给出将实际单元上点变换到参考元上的方法，类内还存有实现此方法的算法（ Newton 法）相关的信息
</br>

### 类内类、枚举和结构体
```C++
// public 枚举，用于表示迭代算法起始点的选取方法
enum InitGuessType
{
    Center = 0, //使用参考元的中心
    ClosestPhysNode = 1, //使用方法 FindClosestPhysPoint() 的返回点，即选择实际单元上的一个“网格”（对应有一个已知的参考元上的“网格”）中距离点 pt 最近的结点所对应的参考元上的结点，而“网格”是由 rel_qpts_order 和 qpts_type 所决定的
    ClosestRefNode = 2, //使用方法 FindClosestRefPoint() 的返回点，与前一条的区别是
    GivenPoint = 3 //使用一个给定的点 ip0
};
```
```C++
// public 枚举，用于表示使用的迭代算法
enum SolverType
{
    Newton = 0, //使用 Newton 算法，不要求每个迭代点都位于参考元的内部
    NewtonSegmentProject = 1, //使用 Newton 算法，但是要求每一步中，如果新的迭代点 x_new 落在参考元外部，那么选择 x_new 和 x_old 连线与单元边界的交点作为新的迭代点
    NewtonElementProject = 2 //使用 Newton 算法，但是要求每一步中，如果新的迭代点 x_new 落在参考元外部，那么选择参考元边界上距离 x_new 最近的一个点作为新的迭代点
};
```
```C++
// public 枚举，用于表示变换算法能否得到正确的结果
enum TransformResult
{
    Inside  = 0, //点在单元内部
    Outside = 1, //点在单元外部
    Unknown = 2  //算法无法确定点在哪里
};
```
</br>

### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `ElementTransformation *T` | T 指向此 InverseElementTransformation 对象所对应的 ElementTransformation 对象 |
| `const IntegrationPoint *ip0` | 用于表示迭代算法的起始点 |
| `int init_guess_type` | 表示迭代算法起始点的选取方法，见枚举 InitGuessType |
| `int qpts_type, rel_qpts_order` | 在确定确定起始点时，如果采用 ClosestPhysNode 或者 ClosestRefNode 的方法，我们需要对参考元进行剖分（见类 GeometryRefiner ），在剖分结点中按照不同的方法选择一个作为起始点，剖分参数为 type = qpts_type, Times = T->Order()+rel_qpts_order （ type 和 Times 的解释见类 GeometryRefiner 和类 RefinedGeometry ） |
| `int solver_type` | 表示迭代算法的类型，见枚举 SolverType |
| `int max_iter` | 迭代算法的最大迭代数 |
| `double ref_tol, phys_rtol` | 在 Newton 算法中，当 $\Vert F(x)-y\Vert_\infty<\text{phys\_rtol}\cdot\Vert y\Vert_\infty$或者$\Vert J^{-1}(x)(F(x)-y)\Vert_\infty<\text{ref\_tol}$（在$J$不是方阵时为$\Vert (J^{T}(x)J(x))^{-1}(F(x)-y)\Vert_\infty<\text{ref\_tol}$）时，认为算法收敛 |
| `double ip_tol` | 用于检测一个点是否在参考元内部的阈值 |
| `int print_level` | 决定打印迭代算法中的什么信息，用于 debug 。-1 - 从不打印，0 - 只打印错误，1 - 打印第一个和最后一个迭代步，2 - 打印每一个迭代步， 3 - 打印每一个迭代步和相应的点坐标 |
</br>

### public 成员函数
</br>

### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `InverseElementTransformation(ElementTransformation *Trans = NULL)` | 默认构造函数，置 T = Trans ， ip0 = NULL ， init_guess_type = Center ， qpts_type = Quadrature1D::OpenHalfUniform ， rel_qpts_order = -1 ， solver_type = NewtonElementProject ， max_iter = 16 ， ref_tol = phys_rtol = 1e-15 ， ip_tol = 1e-8 ， print_level = -1  |
| `virtual ~InverseElementTransformation()` | 默认析构函数 |
</br>

#### 设置成员变量
| 方法 | 解释 |
| ---- | ---- |
| `void SetTransformation(ElementTransformation &Trans)` | 令 T = &Trans |
| `void SetInitialGuessType(InitGuessType itype)` | 令 init_guess_type = itype |
| `void SetInitialGuess(const IntegrationPoint &init_ip)` | 令 ip0 = &init_ip，并且令 init_guess_type = GivenPoint |
| `void SetInitGuessPointsType(int q_type)` | 令 qpts_type = q_type |
| `void SetInitGuessRelOrder(int order)` | 令 rel_qpts_order = order |
| `void SetSolverType(SolverType stype)` | 令 solver_type = stype |
| `void SetMaxIter(int max_it)` | 令 max_iter = max_it |
| `void SetReferenceTol(double ref_sp_tol)` | 令 ref_tol = ref_sp_tol |
| `void SetPhysicalRelTol(double phys_rel_tol)` | 令 phys_rtol = phys_rel_tol |
| `void SetElementTol(double el_tol)` | 令 ip_tol = el_tol |
| `void SetPrintLevel(int pr_level)` | 令 print_level = pr_level |
</br>

#### 寻找一系列点中距离目标点最近的点（用于设置迭代算法起始点）
| 方法 | 解释 |
| ---- | ---- |
| `int FindClosestPhysPoint(const Vector& pt, const IntegrationRule &ir)` | 对参考元中一系列点（储存在 ir 中），在变换作用下对应了实际单元中的一系列点，寻找其中距离 pt 点最近的一个（欧式范数），返回对应 ir 的指标 |
| `int FindClosestRefPoint(const Vector& pt, const IntegrationRule &ir)` | 对参考元中一系列点（储存在 ir 中），在变换作用下对应了实际单元中的一系列点，寻找其中距离 pt 点最近的一个（与上一个方法不同的是，这里采用的距离是$\Vert J^{-1}(x)(F(x)-y)\Vert_2$），返回对应 ir 的指标 |
</br>

#### 逆变换
| 方法 | 解释 |
| ---- | ---- |
| `virtual int Transform(const Vector &pt, IntegrationPoint &ip)` | （虚函数）对实际空间中的点 pt ，求解其逆变换（参考元上的点），储存在 ip 中 |

-------------

## 类 IsoparametricTransformation
- `class IsoparametricTransformation : public ElementTransformation`
- 用于表示等参变换的类，此变换和定义在单元上的有限元有关。对于一个结点型有限元，将变换$F$表示为$x=F(\hat{x})=P\hat{\Phi}(\hat{x})$的形式，其中矩阵$P$每一列是实际单元上的结点，而向量$\hat{\Phi}(\hat{x})$是形函数在点$\hat{x}$上的取值，即将变换后的点表示为结点的线性组合，组合系数为形状函数在参考元上点的取值
</br>

### private成员变量
| 变量 | 解释 |
| ---- | ---- |
| `DenseMatrix dshape,d2shape` |  |
| `Vector shape` | 表示 FiniteElement 对象 *FElem 上的所有形状函数（基函数）在某个积分点上的取值 |
| `const FiniteElement *FElem` | 建立在参考元上的（const） FiniteElement 对象 |
| `DenseMatrix PointMat` | 一个 spacedim*dof 的矩阵，其中 spacedim 指空间维数而 dof 指 FiniteElement 对象 *FElem 上自由度的个数（即基函数的个数），PointMat 的每一列是实际空间中的一个控制点的坐标。对于一个参考元中的点 $\hat{x}$，向量 shape 代表了向量 $\hat{\Phi}(\hat{x})$，即 FiniteElement 对象 *FElemt 上的所有形状函数（基函数）在点 $\hat{x}$ 上的取值，用于表示点 $\hat{x}$ 经过变换得到的实际单元中的点 $x$，$x=F(\hat{x})=P\hat{\Phi}(\hat{x})$，其中 $P$ 储存在 PointMat 中 |
</br>

### public成员函数
</br>

#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `IsoparametricTransformation()` | 构造函数，令 FElem = NULL |
| `virtual ~IsoparametricTransformation()` | （虚函数实例）默认析构函数 |
</br>

#### 设置和读取有关信息
| 方法 | 解释 |
| ---- | ---- |
| `void SetFE(const FiniteElement *FE)` | 设置 FElem 为 FE ，同时，如果 FElem 没有发生变化， EvalState 将不变，否则设置为0； geom 设置为 *FE 的几何类型 |
| `const FiniteElement* GetFE() const` | 返回（ const ）型的有限元 FElem  |
| `void SetPointMat(const DenseMatrix &pm)` | 设置矩阵 PointMat 为 pm ，并且将 EvalState 置为0 |
| `DenseMatrix &GetPointMat()` | 返回矩阵 PointMat 的引用（此方法需要谨慎使用） |
| `const DenseMatrix &GetPointMat() const` | 上条的 const 形式 |
| `void SetIdentityTransformation(Geometry::Type GeomType)` | 根据 GeomType 设置相应的有限元，同时令 geom 为 GeomType 并且根据相应的有限元设置 PointMat 使得相应变换为恒等变换，即 PointMat 的每一列就是有限元结点（在参考元中）的坐标 |
| `virtual int GetSpaceDim() const` | （虚函数实例）返回单元所处的欧式空间维数 |
</br>

#### 点变换和逆变换有关的方法
| 方法 | 解释 |
| ---- | ---- |
| `virtual void Transform(const IntegrationPoint &ip, Vector &trans)` | （虚函数实例）将参考元上的一个积分点 ip 变换到真实单元上，并保存到 trans 中 |
| `virtual void Transform(const IntegrationRule &ir, DenseMatrix &tr)` | （虚函数实例）将参考元上的一个积分规则 ir （即一系列积分点）变换到到真实单元上，并保存到矩阵 tr 中，其中 tr 中的每一列代表一个积分点在真实单元中的坐标 |
| `virtual void Transform(const DenseMatrix &matrix, DenseMatrix &result)` | （虚函数实例）将 matrix 代表的一系列参考元上的点（每一列代表一个点）变换到真实单元上，并保存在 result 中，其中 result 中的每一列代表一个点 |
| `virtual int TransformBack(const Vector & v, IntegrationPoint & ip)` | （虚函数实例）将实际空间中的点 v 变换到参考元中，储存在 ip 中 |
</br>

#### （多项式）阶数相关
| 方法 | 解释 |
| ---- | ---- |
| `virtual int Order() const` | （虚函数实例）返回用于刻画等参变换的有限元对象 FElem 的阶数（调用 FElem->GetOrder() ） |
| `virtual int OrderJ() const` | （虚函数实例）返回变换的 Jacobian 中元素的阶数（作为一个多项式） |
| `virtual int OrderW() const` | （虚函数实例）返回变换的 Jacobian 的行列式（ weight ）的阶数（作为一个多项式） |
| `virtual int OrderGrad(const FiniteElement *fe) const` | （虚函数实例）返回矩阵 ${\rm adj}(J)^T\nabla f_i$ 中元素的阶数（作为一个多项式），其中 $J$ 为变换的 Jacobian 而 $f_i$ 为有限元对象 fe 中的形函数， fe 应该和 FElem 具有相同的函数空间类型（即为 Pk 或者 Qk ） |

-------------------

## 类IntegrationPointTransformation
### public 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `IsoparametricTransformation Transf` |  |
| `void Transform (const IntegrationPoint &ip1, IntegrationPoint &ip2)` |  |
| `void Transform (const IntegrationRule  &ir1, IntegrationRule  &ir2)` |  |


-------------------

## FaceElementTransformations
- `class FaceElementTransformations : public IsoparametricTransformation`
### 类内枚举
- ```C++
  enum ConfigMasks
   {
      HAVE_ELEM1 =  1, ///< Element on side 1 is configured
      HAVE_ELEM2 =  2, ///< Element on side 2 is configured
      HAVE_LOC1  =  4, ///< Point transformation for side 1 is configured
      HAVE_LOC2  =  8, ///< Point transformation for side 2 is configured
      HAVE_FACE  = 16  ///< Face transformation is configured
   };
  ```
### private成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int mask` |  |
| `IntegrationPoint eip1, eip2` |  |
### public成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int Elem1No, Elem2No` |  |
| `Geometry::Type &FaceGeom` |  |
| `ElementTransformation *Elem1, *Elem2` |  |
| `ElementTransformation *Face` |  |
| `IntegrationPointTransformation Loc1, Loc2` |  |
### public成员函数
| 方法 | 解释 |
| ---- | ---- |
| `FaceElementTransformations()` |  |
| `void SetGeometryType(Geometry::Type g)` |  |
| `int GetConfigurationMask() const` |  |
| `void SetIntPoint(const IntegrationPoint *face_ip)` |  |
| `void SetAllIntPoints(const IntegrationPoint *face_ip)` |  |
| `const IntegrationPoint &GetElement1IntPoint()` |  |
| `const IntegrationPoint &GetElement2IntPoint()` |  |
| `virtual void Transform(const IntegrationPoint &, Vector &)` |  |
| `virtual void Transform(const IntegrationRule &, DenseMatrix &)` |  |
| `virtual void Transform(const DenseMatrix &matrix, DenseMatrix &result)` |  |
| `ElementTransformation & GetElement1Transformation()` |  |
| `ElementTransformation & GetElement2Transformation()` |  |
| `IntegrationPointTransformation & GetIntPoint1Transformation()` |  |
| `IntegrationPointTransformation & GetIntPoint2Transformation()` |  |
| `double CheckConsistency(int print_level = 0, std::ostream &out = mfem::out)` |  |

