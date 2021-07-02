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

# fe.hpp & fe.cpp
## 类 BasisType
- `class BasisType`
### 类内类、枚举和结构体
- ```C++
  // public 枚举
  enum
   {
      Invalid         = -1,
      GaussLegendre   = 0,  // 开类型
      GaussLobatto    = 1,  // 闭类型
      Positive        = 2,  // Bernstein 多项式
      OpenUniform     = 3,  // 结点: x_i = (i+1)/(n+1), i=0,...,n-1
      ClosedUniform   = 4,  // 结点: x_i = i/(n-1),     i=0,...,n-1
      OpenHalfUniform = 5,  // 结点: x_i = (i+1/2)/n,   i=0,...,n-1
      Serendipity     = 6,  // Serendipity 基 (正方形 / 立方体)
      ClosedGL        = 7,  // 闭 GaussLegendre
      NumBasisTypes   = 8   
   };
  ```
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `static int Check(int b_type)` | 检查 b_type 是否是一个合法的基类型，如是则返回它，否则将报错 |
| `static int CheckNodal(int b_type)` | 如果 b_type 不是一个合法的结点基类型（ b_type 合法并且不是 Positive ），报错，否则返回它 |
| `static int GetQuadrature1D(int b_type)` | 得到 b_type 所对应的 Quadrature1D 类型，当这个对应无意义时，返回 Quadrature1D::Invalid |
| `static int GetNodalBasis(int qpt_type)` | 对输入的 Quadrature1D 类型 qpt_type，返回其对应的结点基类型，当这个对应无意义时，返回 Invalid |
| `static const char *Name(int b_type)` | 当 b_type 合法时，返回其代表的基类型的名称（作为一个字符串） |
| `static char GetChar(int b_type)` | 当 b_type 合法时，返回其代表的基类型的标识符（作为一个字符） |
| `static int GetType(char b_ident)` | 给定一个字符型的标识符，返回其对应的基类型 |

## 类 DofToQuad
- `class DofToQuad`
- 用于表示一个特定的有限元中的基函数在特定积分点上的值、梯度、散度或者旋度的值所张成的矩阵或者张量， DofToQuad 对象通常被对应的有限元对象创建和所有
### 类内类、枚举和结构体
- ```C++
  // public 枚举，用于描述储存在 #B ， #Bt ， #G ， 和 #Gt 中数据的类型
  enum Mode
   {
      // 数据使用全维度表示，而不使用张量积结构，自由度的排序与 #FE 中所定义的一致
      FULL,

      // 数据使用张量积结构，即相应的矩阵\张量（通常对应于高维有限元）由1D有限元上相对应的矩阵\张量的张量积形式所表示 
      // 当表示一个向量值有限元时，将使用两个 DofToQuad 对象来刻画“闭”和“开”的1D基函数
      TENSOR
   };
  ```
### public 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const class FiniteElement *FE` | *FE 是创建和拥有这个 DofToQuad 对象的 FiniteElement 对象，此指针并不被 DofToQuad 对象所拥有 |
| `const IntegrationRule *IntRule` | 有限元 *FE 上的基函数将在 IntegrationRule 对象 *IntRule 所刻画的积分点上取值，此指针并不被 DofToQuad 对象所拥有  |
| `Mode mode` | 描述储存在 #B ， #Bt ， #G ， 和 #Gt 中数据的类型 |
| `int ndof` | 自由度的个数 |
| `int nqpt` | 积分点的个数 |
| `Array<double> B` | 基函数在积分点上的取值，相应的矩阵和张量按列储存 <br> - 对于标量型有限元，长为 #nqpt x #ndof <br> - 对于向量型有限元，长为 #nqpt x dim x #ndof <br> 其中，当 mode = FULL 时， dim 是参考有限元的维数；当 mode = TENSOR 时， dim = 1 |
| `Array<double> Bt` | B 的转置，相应的矩阵和张量按列储存 <br> - 对于标量型有限元，长为 #ndof x #nqpt <br> - 对于向量型有限元，长为 #ndof x #nqpt x dim |
| `Array<double> G` | 基函数在积分点上的梯度/散度/旋度，相应的矩阵和张量按列储存 <br> - 对标量型有限元（此时计算梯度），长为 #nqpt x dim x #ndof <br> - 对于 H(div) 型有限元（此时计算散度），长为 #nqpt x #ndof <br> - 对于 H(curl) 型有限元（此时计算旋度） <br> 其中， <br> - 当 mode = FULL 时， dim 是参考有限元的维数；当 mode = TENSOR 时， dim = 1 <br> - 当 mode = FULL 时，分别对于 1D/2D/3D 情形， cdim 为 1/1/3 ；当 mode = TENSOR 时， cdim = 1 |
| `Array<double> Gt` | G 的转置，相应的矩阵和张量按列储存 <br> - 对于标量型有限元，长度为 #ndof x #nqpt x dim <br> - 对于 H(div) 型有限元，长度为 #ndof x #nqpt <br> - 对于 H(curl) 型有限元，长度为 #ndof x #nqpt x cdim | 

## 类 FunctionSpace
- `class FunctionSpace`
- 刻画在每一个单元上的函数空间
### 类内类、枚举和结构体
- ```C++
  enum
   {
      Pk, //k阶多项式
      Qk, //k阶多项式的张量积
      rQk //k阶多项式的（refined 细？）张量积
   };
  ```

## （基）类 FiniteElement
- `class FiniteElement`
### 类内类、结构体和枚举
- ```C++
  // public 枚举，对于 range_type 和 deriv_range_type 的枚举
  enum RangeType 
   { 
      SCALAR, //值域为标量
      VECTOR  //值域为向量
   };
  ```
- ```C++
  // public 枚举，对于 map_type 的枚举
  enum MapType
   {
      VALUE,     // 对标量场，保持点的值 \f$ u(x) = \hat u(\hat x) \f$ 
      INTEGRAL,  // 对标量场，保持体积分 \f$ u(x) = (1/w) \hat u(\hat x) \f$ 
      H_DIV,     // 对向量场，保持垂直分量的面积分 \f$ u(x) = (J/w) \hat u(\hat x) \f$ 
      H_CURL     // 对向量场，保持切向分量的线积分 \f$ u(x) = J^{-t} \hat u(\hat x) \f$ ( J 为方阵), \f$ u(x) = J(J^t J)^{-1} \hat u(\hat x) \f$ ( J 为一般矩阵) 
   };
  ```
- ```C++
  // public 枚举，对 deriv_type, deriv_range_type, deriv_map_type 的枚举
  enum DerivType
   {
      NONE, // 不执行导数操作
      GRAD, // 需要计算梯度
      DIV,  // 需要计算散度
      CURL  // 需要计算旋度
   };
  ```
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int dim` | 参考空间的维数 |
| `Geometry::Type geom_type` | 参考元的几何类型 |
| `int func_space` | 建立在单元上的函数类型，取值源于类 FunctionSpace |
| `int range_type` | 刻画单元是标量有限元还是向量有限元，取值源于枚举 RangeType |
| `int map_type` | 刻画参考元上的函数是如何映射到实际空间上的，取值源于枚举 MapType |
| `int deriv_type` | 表示使用了哪种导数方法，取值源于枚举 DerivType |
| `int deriv_range_type` | 表示单元导数是标量型的还是向量型的，取值源于枚举 RangeType |
| `int deriv_map_type` | 刻画参考元上函数的导数是如何映射到实际空间上的，取值源于枚举 MapType |
| `mutable int dof` | 自由度的个数 |
| `mutable int order` | 形函数的阶数 |
| `mutable int orders[Geometry::MaxDim]` | 各向异性阶数，即对于向量型有限元，在不同维度上的基函数可以具有不同的阶数 |
| `IntegrationRule Nodes` | 表示有限元的结点信息 |
| `mutable Array<DofToQuad*> dof2quad_array` | 用于储存本 FiniteElement 对象所创建的所有 DofToQuad 对象，当存在不同的积分规则或使用不同的 DofToQuad::Mode 时可能需要多个 DofToQuad 对象 |
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `FiniteElement(int D, Geometry::Type G, int Do, int O, int F = FunctionSpace::Pk)` | 构造函数，令 dim = D ， geom_type = G ， dof = Do ， order = O ， func_space = F ， 并且分别令 range_type ， map_type ， deriv_type ， deriv_range_type ， deriv_map_type 为 SCALAR ， VALUE ， NONE ， SCALAR ， VALUE （默认）， orders 的值初始化为 -1 并且为 Nodes 和 vshape 分配内存 |
| `int GetDim() const` | 返回 dim |
| `Geometry::Type GetGeomType() const` | 返回 geom_type |
| `int GetDof() const` | 返回 dof |
| `int GetOrder() const` | 返回 order |
| `bool HasAnisotropicOrders() const` | 判断有限元基是否“可能”在不同维度上使用了不同的阶数，即如果 orders[0] != -1 则返回真 |
| `const int *GetAnisotropicOrders() const` | 返回 orders |
| `int Space() const` | 返回 func_space |
| `int GetRangeType() const` | 返回 range_type |
| `int GetDerivRangeType() const` | 返回 deriv_range_type |
| `int GetMapType() const` | 返回 map_type |
| `int GetDerivType() const` | 返回 deriv_type |
| `int GetDerivMapType() const` | 返回 deriv_map_type |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const = 0` | （纯虚函数）对一个标量值有限元，计算所有（参考空间上的）形函数在点 ip 上的值，储存到 shape 中， shape 的长度（#dof）必须要预先指定 |
| `void CalcPhysShape(ElementTransformation &Trans, Vector &shape) const` | 对一个标量值有限元，计算所有（实际空间上的）形函数在 Trans 刻画的点上的值（调用 CalcShape ），储存到 shape 中， shape 的长度（#dof）必须要预先指定 |
| `virtual void CalcDShape(const IntegrationPoint &ip, DenseMatrix &dshape) const = 0` | （纯虚函数）对一个标量值有限元，计算所有（参考空间上的）形函数在点 ip 上的梯度，储存到 dshape 中， dshape 的每一行代表一个形函数的梯度， dshape 的尺寸（#dof $\times$ #dim）必须要预先指定 |
| `void CalcPhysDShape(ElementTransformation &Trans, DenseMatrix &dshape) const` | 对一个标量值有限元，计算所有（实际空间上的）形函数在 Trans 所刻画的点上的梯度，储存到 dshape 中（调用 CalcPhysDShape ）， dshape 的每一行代表一个形函数的梯度， dshape 的尺寸（#dof $\times$ Sdim）必须要预先指定，其中 SDim >= #dim 是实际空间的维数，在 Trans 中有所描述 |
| `const IntegrationRule & GetNodes() const` | 返回 Nodes 的 const 引用 |
| `virtual void CalcVShape(const IntegrationPoint &ip, DenseMatrix &shape) const` | （虚函数）对一个向量值有限元，计算所有（参考空间上的）形函数在点 ip 上的值，储存在 shape 中， shape 的每一行代表了一个向量值形函数的取值， shape 的尺寸（#dof $\times$ #dim）必须要预先指定 |
| `virtual void CalcVShape(ElementTransformation &Trans, DenseMatrix &shape) const` | （虚函数）对一个向量值有限元，计算所有（实际空间上的）形函数在 Trans 刻画的点上的值，储存在 shape 中， shape 的每一行代表了一个向量值形函数的取值， shape 的尺寸（#dof $\times$ Sdim）必须要预先指定，其中 SDim >= #dim 是实际空间的维数，在 Trans 中有所描述  |
| `void CalcPhysVShape(ElementTransformation &Trans, DenseMatrix &shape) const` | 等价于方法 CalcVShape() |
| `virtual void CalcDivShape(const IntegrationPoint &ip, Vector &divshape) const` | （虚函数）对一个向量值有限元，计算所有（参考空间上的）形函数在点 ip 上的散度，并储存在 divshape 中， divshape 的长度（#dof）必须要预先指定 |
| `void CalcPhysDivShape(ElementTransformation &Trans, Vector &divshape) const` | 对一个向量值有限元，计算所有（实际空间上的）形函数在 Trans 刻画的点上的散度（调用方法 CalcDivShape() ），并储存在 divshape 中， divshape 的长度（#dof）必须要预先指定 |
| `virtual void CalcCurlShape(const IntegrationPoint &ip, DenseMatrix &curl_shape) const` | （虚函数）对一个向量值有限元，计算所有（参考空间上的）形函数在点 ip 上的旋度，并储存在 curl_shape 中， curl_shape 的每一行代表了一个向量值形函数的旋度的值， curl_shape 的尺寸（#dof $\times$ Cdim）必须要预先指定，其中当 #dim = 3 时，CDim = 3 ，而当 #dim = 2 时，CDim = 1 |
| `void CalcPhysCurlShape(ElementTransformation &Trans, DenseMatrix &curl_shape) const` | 对一个向量值有限元，计算所有（实际空间上的）形函数在 Trans 刻画的点上的旋度，并储存在 curl_shape 中， curl_shape 的每一行代表了一个向量值形函数的旋度的值， curl_shape 的尺寸（#dof $\times$ Cdim）必须要预先指定，其中当 #dim = 3 时，CDim = 3 ，而当 #dim = 2 时，CDim = 1 |
| `virtual void GetFaceDofs(int face, int **dofs, int *ndofs) const` | （虚函数）得到面 face 上的自由度，将这样的自由度编号（作为一个 int 型数组）的首地址储存到 *dofs 中，而这样的自由度个数储存到 *dof 中 |
| `virtual void CalcHessian(const IntegrationPoint &ip, DenseMatrix &Hessian) const` | （虚函数）对一个标量值有限元，计算所有（参考空间上的）形函数在点 ip 上的 Hessian ，并储存在 Hessian 中， Hessian 的每一行储存了一个标量值形函数的 Hessian 的上三角部分（2D时的储存顺序为{u_xx, u_xy, u_yy}）， Hessian 的尺寸（#dof $\times$ (#dim (#dim+1)/2）必须要预先指定 |
| `virtual void CalcPhysHessian(ElementTransformation &Trans, DenseMatrix& Hessian) const` | （虚函数）对一个标量值有限元，计算所有（实际空间上的）形函数在 Trans 刻画的点上的 Hessian （调用方法 CalcHessian() ），并储存在 Hessian 中， Hessian 的每一行储存了一个标量值形函数的 Hessian 的上三角部分（2D时的储存顺序为{u_xx, u_xy, u_yy}）， Hessian 的尺寸（#dof $\times$ (#dim (#dim+1)/2）必须要预先指定 |
| `virtual void CalcPhysLaplacian(ElementTransformation &Trans, Vector& Laplacian) const` | （虚函数）对一个标量值有限元，计算所有（实际空间上的）形函数在 Trans 刻画的点上的 Laplacian ，并储存在 Laplacian 中， Laplacian 的长度（#dof）必须预先指定  |
| `virtual void CalcPhysLinLaplacian(ElementTransformation &Trans, Vector& Laplacian) const` | （虚函数）当 Trans 代表的变换为线性时，方法 CalcPhysLaplacian() 的简单实现 |
| `virtual void GetLocalInterpolation(ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数）<font color=yellow>似乎和网格加细有关，给出粗单元到细单元的插值（对于结点型单元，即是计算粗单元的所有形状函数在细单元结点上的取值），储存到 I 中， I 的每一行代表了细单元的一个自由度， Trans 刻画了基本几何到细单元的变换</font> |
| `virtual void GetLocalRestriction(ElementTransformation &Trans, DenseMatrix &R) const` | （虚函数）<font color=yellow>似乎和网格加细有关，给出粗自由度到细自由度的限制（对于结点型单元，即是计算粗单元的所有形状函数在位于细单元内粗单元结点上的取值），储存到 R 中， I 的每一行代表了一个粗自由度，当一个粗自由度不能被细自由度所表达时， R 的对应行的第一个元素将被设置为 infinity()， Trans 刻画了基本几何到细单元的变换</font> |
| `virtual void GetTransferMatrix(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数）<font color=yellow>似乎和网格加细有关</font> |
| `virtual void Project(Coefficient &coeff, ElementTransformation &Trans, Vector &dofs) const` | （虚函数）计算 Coefficient 对象 coeff 在建立在变换 Trans 所刻画的（实际空间中）的单元上的有限元空间上的投影，即得到基函数的某个线性组合满足某种“逼近性”，线性组合系数储存在 dofs 中 |
| `virtual void Project(VectorCoefficient &vc, ElementTransformation &Trans, Vector &dofs) const` | （虚函数）计算 VectorCoefficient 对象 vc 在建立在变换 Trans 所刻画的（实际空间中）的单元上的有限元空间上的投影，即得到基函数的（ vc.vdim 个）线性组合满足某种“逼近性”，线性组合系数储存在 dofs 中（ dofs 按照 dof > vc.vdim 的顺序排列 ）<font color = yellow>向量型有限元？</font> |
| `virtual void ProjectFromNodes(Vector &vc, ElementTransformation &Trans, Vector &dofs) const` |  |
| `virtual void ProjectMatrixCoefficient( MatrixCoefficient &mc, ElementTransformation &T, Vector &dofs) const` | （虚函数）计算 MatrixCoefficient 对象 mc 在建立在变换 Trans 所刻画的（实际空间中）的单元上的有限元空间上的投影，即得到基函数的线性组合（ mc.height x mc.weight 个）满足某种“逼近性”，线性组合系数储存在 dofs 中（ dofs 按照 dof > mc.weight > mc.height 的顺序排列 ）<font color = yellow>向量型有限元？</font>|
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |
| `virtual void Project(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数）计算从给定的 FiniteElement 对象 fe 到 this 对象的嵌入/投影矩阵 I |
| `virtual void ProjectGrad(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &grad) const` |  |
| `virtual void ProjectCurl(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &curl) const` |  |
| `virtual void ProjectDiv(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &div) const` |  |
| `virtual const DofToQuad &GetDofToQuad(const IntegrationRule &ir, DofToQuad::Mode mode) const` | （虚函数）对于给定的 IntegrationRule 对象 ir （一列积分点）和模式 mode ，计算对应的 DofToQuad 结构体 |
| `virtual ~FiniteElement()` | （虚函数）析构函数，释放分配的内存 |
| `static bool IsClosedType(int b_type)` | 当 b_type 是闭的基类型时（在边界上有 Quadrature1D 点）返回真 |
| `static bool IsOpenType(int b_type)` | 当 b_type 是开的基类型时（在边界上没有 Quadrature1D 点）返回真 |
| `static int VerifyClosed(int b_type)` | 当 b_type 不是闭的基类型时报错，否则返回 b_type |
| `static int VerifyOpen(int b_type)` | 当 b_type 不是开的基类型时报错，否则返回 b_type |
| `static int VerifyNodal(int b_type)` | 当 b_type 不是顶点型基时报错，否则返回 b_type |

## （基）类 ScalarFiniteElement
- `class ScalarFiniteElement : public FiniteElement`
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `ScalarFiniteElement(int D, Geometry::Type G, int Do, int O, int F = FunctionSpace::Pk)` | 构造函数，调用 FiniteElement(D, G, Do, O, F) ，并且令 deriv_type = GRAD ，deriv_range_type = VECTOR ， deriv_map_type = H_CURL |
| `void SetMapType(int M)` | 设置 map_type = M ，其中 M 为 VALUE 或者 INTEGRAL ，并且当 M = VALUE 时设置 deriv_type 为 GRAD ，否则为 NONE |
| `void NodalLocalInterpolation(ElementTransformation &Trans, DenseMatrix &I, const ScalarFiniteElement &fine_fe) const` | 得到 this 单元到细单元 fine_fe 的结点插值矩阵 I， 细单元的范围（作为 this 单元参考元的子集）由变换 Trans 的像所刻画， I 的每一行代表了所有（ this 单元上的）形状函数在细单元节点上的取值（对于 INTEGRAL 单元，还需要乘以一个权） |
| `void ScalarLocalInterpolation(ElementTransformation &Trans, DenseMatrix &I, const ScalarFiniteElement &fine_fe) const` |  |
| `virtual const DofToQuad &GetDofToQuad(const IntegrationRule &ir, DofToQuad::Mode mode) const` | （虚函数实例） mode 必须是 FULL ，如果 dof2quad_array 中已经包含了需要的 DofToQuad 对象（即具有相同的积分点和模式），则直接返回，否则计算单元上所有形函数在积分点上的值和梯度，组成一个 DofToQuad 对象返回并且将其添加在 dof2quad_array 后 |

## （基）类 NodalFiniteElement
- `class NodalFiniteElement : public ScalarFiniteElement`
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `NodalFiniteElement(int D, Geometry::Type G, int Do, int O, int F = FunctionSpace::Pk)` | 构造函数，调用方法 ScalarFiniteElement() |
| `virtual void GetLocalInterpolation(ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数实例）调用 ScalarFiniteElement::NodalLocalInterpolation() |
| `virtual void GetLocalRestriction(ElementTransformation &Trans, DenseMatrix &R) const` | （虚函数实例）给出粗单元（ this 单元）自由度到细单元（由 Trans 的值域 --- this 单元参考空间的子集 --- 所刻画）自由度的限制，即计算粗单元的所有形状函数在位于细单元内粗单元结点上的取值， R 的每一行代表了一个粗结点，当一个粗结点不在细单元上时， R 的这一行将被设置为 infinity() |
| `virtual void GetTransferMatrix(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数实例）确定 fe 是 ScalarFiniteElement 对象，然后调用 fe.NodalLocalInterpolation() |
| `virtual void Project (Coefficient &coeff, ElementTransformation &Trans, Vector &dofs) const` | （虚函数实例）直接调用 coeff.Eval()  |
| `virtual void Project (VectorCoefficient &vc, ElementTransformation &Trans, Vector &dofs) const` | （虚函数实例）直接调用 vc.Eval() ， ( vc.vdim ) @ DOFS -> (dof x vc.vdim) in dofs （ dofs 排列顺序如前括号中内容所示）  |
| `virtual void ProjectMatrixCoefficient( MatrixCoefficient &mc, ElementTransformation &T, Vector &dofs) const` | （虚函数实例）直接调用 mc.Eval() ， (mc.height x mc.width) @ DOFs -> (dof x mc.width x mc.height) in dofs ，（ dofs 排列顺序如前括号中内容所示） |
| `virtual void Project(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数实例）调用 fe.CalcShape() 或者 fe.CalcVShape() 计算 |
| `virtual void ProjectGrad(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &grad) const` | （虚函数实例）调用 fe.CalcDShape() 进行计算 |
| `virtual void ProjectDiv(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &div) const` | （虚函数实例）调用 fe.CalcDivShape() 进行计算 |

--------------------------

## （基）类 PositiveFiniteElement
- `class PositiveFiniteElement : public ScalarFiniteElement`
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `PositiveFiniteElement(int D, Geometry::Type G, int Do, int O, int F = FunctionSpace::Pk)` | 构造函数，调用 `ScalarFiniteElement()` |
| `virtual void GetLocalInterpolation(ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数实例）调用 `ScalarFiniteElement::ScalarLocalInterpolation()` |
| `virtual void GetLocalRestriction(ElementTransformation &Trans, DenseMatrix &R) const` | （虚函数实例）调用 `ScalarFiniteElement::ScalarLocalRestriction()` |
| `virtual void GetTransferMatrix(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数实例）确定 fe 是 ScalarFiniteElement 对象，然后调用 `fe.ScalarLocalInterpolation()` |
| `virtual void Project(Coefficient &coeff, ElementTransformation &Trans, Vector &dofs) const` | （虚函数实例）调用 `coeff.Eval()` |
| `virtual void Project (VectorCoefficient &vc, ElementTransformation &Trans, Vector &dofs) const` | （虚函数实例）调用 `vc.Eval()` |
| `virtual void Project(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | （虚函数实例）<font color=yellow>待补充</font> |

## （基）类 VectorFiniteElement
- `class VectorFiniteElement : public FiniteElement`
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
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


## 类 Poly1D
### 类内类、枚举、结构体
-  ```C++
    enum EvalType
    {
       ChangeOfBasis = 0, // 将 CalcBasis() 中的基函数进行线性变换得到新的基函数，调用方法 Eval() 时运算量为 O(p^2)
       Barycentric   = 1, // 使用重心 Lagrange 插值基函数，调用方法 Eval() 时运算量为 O(p)
       Positive      = 2, // 使用 Bernstain 基函数
       NumEvalTypes  = 3  // 记录 EvalType 个数
    };
    ```
-   ```C++
    // public 类，
    class Basis
    {
    private:
       // EvalType 类型
       int etype;
       // 只在 etype 为 ChangeOfBasis 时起作用，此时每个新基函数是方法 CalcBasis() 中的基函数的线性组合，组合系数储存在 Ai 对应列中
       DenseMatrixInverse Ai;
       /**
            - etype = ChangeOfBasis 时：x 和 w 被分配了内存（长均为 p+1）用于方法 Eval() 中保存中间变量，节省内存
            - etype = Barycentric 时：x 储存 Lagrange 插值的结点， w 储存 Barycentric 型 Lagrange 插值的 Barycentric 权
            - etype = Positive：仅设置 x 的长度为 p+1 ，用于储存 p
       */
       mutable Vector x, w;

    public:
       /**  创建结点型或者正型（Bernstain 型）基，根据 etype 的不同有着不同的实现方法
            - etype = ChangeOfBasis 时：计算一组新的基函数 $\psi_i$，使其在 nodes 代表的 p 个点 $x_j$ 上有 $\psi_i(x_j) = \delta_{ij}$，新基函数的系数信息用 Ai 保存
            - etype = Barycentric 时：对于相对于结点 nodes 的重心型的 lagrange 基函数，计算重心权，保存到 w 中，并且令 x 为 nodes 代表的点
            - etype = Positive 时：设置 x 的长度为 p+1 ，而内容为空，仅为表示 p
       */
       Basis(const int p, const double *nodes, EvalType etype = Barycentric);
       void Eval(const double x, Vector &u) const;// 计算基函数在点 x 上的取值，保存到 u 中
       void Eval(const double x, Vector &u, Vector &d) const;// 计算基函数及其导数在点 x 上的取值，分别保存到 u 和 d 中
       void Eval(const double x, Vector &u, Vector &d, Vector & d2) const;// 计算基函数及其导数和二阶导数在点 x 上的取值，分别保存到 u 、 d 和 d2 中
    };
    ```
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `MemoryType h_mt` | 内存类型 |
| `PointsMap points_containe` | 储存了已经计算过的对应于阶数和基类型的1维积分点，见方法 GetPoints() |
| `BasisMap  bases_container` | 储存了已经计算过的对应于某个阶数和基类型的 Poly_1D::Basis 对象，见方法 GetBasis() |
| `static Array2D<int> binom` | binom(i,0:i) 代表了所有的组合数 $C_i^k$，见方法 Binom() |
| `typedef std::map< int, Array<double*>* > PointsMap` |  |
| `typedef std::map< int, Array<Basis*>* > BasisMap` |  |
| `QuadratureFunctions1D quad_func` |  |

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `Poly_1D()` | 默认构造函数，置 h_mt 为 MemoryType::HOST |
| `static const int *Binom(const int p)` | （静态方法）对给定的整数 p ，返回一个指向 int 型数组的 const 指针，这个数组代表了组合数 $C_n^k(k=0,...,p)$ ， binom 将被更新以避免重复计算 |
| `const double *GetPoints(const int p, const int btype)` | 对于 BasisType btype ，返回一个指向长为 p+1 （对应于 p 阶多项式）的 double 型数组的 const 指针，这个数组代表了一列1维积分点的坐标（调用方法 quad_func.GivePolyPoints() 计算）， points_containe 将被更新以避免重复计算  |
| `const double *OpenPoints(const int p, const int btype = BasisType::GaussLegendre)` | 返回 GetPoints(p, btype) |
| `const double *ClosedPoints(const int p, const int btype = BasisType::GaussLobatto)` | 返回 GetPoints(p, btype) |
| `Basis &GetBasis(const int p, const int btype)` | 给定阶数 p 和基类型 btype ，得到一个 Poly_1D::Basis 对象， bases_container 将被更新以避免重复计算 |
| `static void CalcBasis(const int p, const double x, double *u)` | （静态方法）计算具有层次结构的 p 阶1D基函数（这里是 Chebyshev 函数）在点 x 上的取值，储存在 *u 中（ u 指向一个长至少为 p+1 的数组） |
| `static void CalcBasis(const int p, const double x, double *u, double *d)` | （静态方法）计算具有层次结构的 p 阶1D基函数（这里是 Chebyshev 函数）和其一阶导数在点 x 上的取值，分别储存在 *u 和 *d 中（ u 和 d 分别指向一个长至少为 p+1 的数组） |
| `static void CalcBasis(const int p, const double x, double *u, double *d, double *dd)` | （静态方法）计算具有层次结构的 p 阶1D基函数（这里是 Chebyshev 函数）和其一阶、二阶导数在点 x 上的取值，分别储存在 *u 、 *d 和 *dd 中（ u 、 d 和 dd 分别指向一个长至少为 p+1 的数组） |
| `static double CalcDelta(const int p, const double x)` | （静态方法）<font color = yellow>Delta函数是啥？遇到再说</font> |
| `static void ChebyshevPoints(const int p, double *x)` | （静态方法）计算 p 阶（ p+1 个） Chebyshev 点，保存到 x 指向的已经分配好内存的数组中 |
| `static void CalcBinomTerms(const int p, const double x, const double y, double *u)` | （静态方法）计算二项式 $(x+y)^p$ 的展开项，保存到 u 指向的已经分配好内存的数组中，即 $u(i)=C^i_p x^i y^{p-i},i=0,1,...,p$ |
| `static void CalcBinomTerms(const int p, const double x, const double y, double *u, double *d)` | （静态方法）计算二项式 $(x+y)^p$ 的展开项及其关于 x 的导数（假设 dy/dx=-1 ），分别保存到 u 和 d 指向的已经分配好内存的数组中，即 $u(i)=C^i_p x^i y^{p-i},i=0,1,...,p$ 和 $d(i) = C^i_p x^{i-1} y^{p-i-1}(i(x+y)-px),i=1,...,p-1,d(0)=-py^{p-1},d(p)=px^{p-1}$ |
| `static void CalcDBinomTerms(const int p, const double x, const double y, double *d)` | （静态方法）计算二项式 $(x+y)^p$ 的展开项关于 x 的导数（假设 dy/dx=-1 ），保存到 d 指向的已经分配好内存的数组中，即 $d(i) = C^i_p x^{i-1} y^{p-i-1}(i(x+y)-px),i=1,...,p-1,d(0)=-py^{p-1},d(p)=px^{p-1}$ |
| `static void CalcBernstein(const int p, const double x, double *u)` | （静态方法）计算 p 阶（p+1 个） Bernstein 多项式在点 x 上的值，保存到 u 指向的已经分配好内存的数组中 |
| `static void CalcBernstein(const int p, const double x, double *u, double *d)` | （静态方法）计算 p 阶（p+1 个） Bernstein 多项式在点 x 上的值和导数值，分别保存到 u 和 d 指向的已经分配好内存的数组中 |
| `static void CalcLegendre(const int p, const double x, double *u)` | （静态方法）计算 p 阶（p+1 个） Legendre 多项式在点 x 上的值，保存到 u 指向的已经分配好内存的数组中 |
| `static void CalcLegendre(const int p, const double x, double *u, double *d)` | 静态方法）计算 p 阶（p+1 个） Legendre 多项式在点 x 上的值和导数值，分别保存到 u 和 d 指向的已经分配好内存的数组中 |
| `~Poly_1D()` | 析构函数，释放 points_container 和 bases_container 中保存的指针指向的内存 |


## 类 TensorBasisElement
- `class TensorBasisElement`
- 定义为1D单元的张量积的单元
### 类内类、枚举、结构体
-   ```C++
    // public 枚举 
    enum DofMapType
    {
       L2_DOF_MAP = 0,
       H1_DOF_MAP = 1,
       Sr_DOF_MAP = 2,  // Sr = Serendipity
    };
     ```
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int b_type` | 对应于1D单元的基类型 |
| `Array<int> dof_map` | 一个长为单元自由度个数的 int 型数组，建立起单元内自由度自然排序和字典排序的映射，即如果 dof_map[i] = j ，那么按照字典排序第 i 个自由度对应的自然排序标号为 j ，如果 dof_map 为空，则意味着此映射为单位映射（意味着自由度按照字典排序） |
| `Poly_1D::Basis &basis1d` | 对应于 1D 单元的 Poly_1D::Basis 引用 |
| `Array<int> inv_dof_map` |  |
| `Array<int> inv_dof_map` |  |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `TensorBasisElement(const int dims, const int p, const int btype, const DofMapType dmtype)` | 构造函数，令 b_type 为 btype ，调用方法 `poly1d.GetBasis()` 构造 basis1d ，并根据维数 dims 和自由度映射类型 dmtype 构造 dof_map ：当 dmtype = L2_DOF_MAP 时，置 dof_map 为空，即此时自由度按照字典排序 |
| `int GetBasisType() const` | 返回 b_type |
| `const Poly_1D::Basis& GetBasis1D()` | 返回 basis1d （ const ） |
| `const Array<int> &GetDofMap() const` | 返回 dof_map 的 const 引用 |
| `static Geometry::Type GetTensorProductGeometry(int dim)` | （静态方法）根据维数 dim 返回张量型的几何类型，即 dim = 1 时返回 Geometry::SEGMENT ， dim = 2 时返回 Geometry::SQUARE ， dim = 3 时返回 Geometry::CUBE |
| `static int Pow(int base, int dim)` | （静态方法）返回 $base^{dim}$ |


## 类 NodalTensorFiniteElement
- `class NodalTensorFiniteElement : public NodalFiniteElement, public TensorBasisElement`
- 顶点张量型有限元，定义为1D顶点型有限元的张量积
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `NodalTensorFiniteElement(const int dims, const int p, const int btype, const DofMapType dmtype);` | 构造函数，根据相关信息分别调用父类的构造函数 `NodalFiniteElement()` 和 `TensorBasisElement()` ， 并令 lex_ordering = dof_map |
| `const DofToQuad &GetDofToQuad(const IntegrationRule &ir, DofToQuad::Mode mode) const` | 根据积分点 ir 和模式 mode ，计算有限元对应的 DofToQuad 对象并返回 |
| `virtual void GetTransferMatrix(const FiniteElement &fe, ElementTransformation &Trans, DenseMatrix &I) const` | <font color=yellow>待补充</font> |


## 类 PositiveTensorFiniteElement
- `class PositiveTensorFiniteElement : public PositiveFiniteElement, public TensorBasisElement`
- 正张量型有限元，定义为1D正型有限元的张量积
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| 构造函数和析构函数 |  |
| `PositiveTensorFiniteElement(const int dims, const int p, const DofMapType dmtype)` | 构造函数，分别调用 `PositiveFiniteElement()` 和 `TensorBasisElement()` |
| `const DofToQuad &GetDofToQuad(const IntegrationRule &ir, DofToQuad::Mode mode) const` | 根据积分点 ir 和模式 mode ，计算有限元对应的 DofToQuad 对象并返回 |


## 类 VectorTensorFiniteElement
- `class VectorTensorFiniteElement : public VectorFiniteElement, public TensorBasisElement`
- 张量型向量值有限元
### privite 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `mutable Array<DofToQuad*> dof2quad_array_open` |  |
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `Poly_1D::Basis &cbasis1d, &obasis1d` |  |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `VectorTensorFiniteElement(const int dims, const int d, const int p, const int cbtype, const int obtype, const int M, const DofMapType dmtype)` |  |
| `const DofToQuad &GetDofToQuad(const IntegrationRule &ir, DofToQuad::Mode mode) const` |  |
| `const DofToQuad &GetDofToQuadOpen(const IntegrationRule &ir, DofToQuad::Mode mode) const` |  |
| `const DofToQuad &GetTensorDofToQuad(const IntegrationRule &ir, DofToQuad::Mode mode, const bool closed) const` |  |
| `~VectorTensorFiniteElement()` |  |

## 类 H1_SegmentElement
- `class H1_SegmentElement : public NodalTensorFiniteElement`
- 1D（线段）上任意阶有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1_SegmentElement(const int p, const int btype = BasisType::GaussLobatto)` | 构造函数，构造阶数为 p 的，基类型为 btype 的1D H1有限元，要求 btype 必须是闭类型的，方法中需要对 Nodes 赋值 |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` | （虚函数实例） |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void CalcHessian(const IntegrationPoint &ip, DenseMatrix &Hessian) const` |  |
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |


## 类 H1_QuadrilateralElement
- `class H1_QuadrilateralElement : public NodalTensorFiniteElement`
- 矩形上任意阶张量型有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1_QuadrilateralElement(const int p, const int btype = BasisType::GaussLobatto)` | 构造函数，构造阶数为 p 的，（1D）基类型为 btype 的H1矩形有限元，要求 btype 必须是闭类型的，方法中需要对 Nodes 赋值 |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void CalcHessian(const IntegrationPoint &ip, DenseMatrix &Hessian) const` |  |
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |

## 类 H1_HexahedronElement
- `class H1_HexahedronElement : public NodalTensorFiniteElement`
- 立方体上任意阶张量型有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1_HexahedronElement(const int p, const int btype = BasisType::GaussLobatto)` | 构造函数，构造阶数为 p 的，（1D）基类型为 btype 的H1立方体有限元，要求 btype 必须是闭类型的，方法中需要对 Nodes 赋值 |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void CalcHessian(const IntegrationPoint &ip, DenseMatrix &Hessian) const` |  |
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |


## 类 H1Pos_SegmentElement
- `class H1Pos_SegmentElement : public PositiveTensorFiniteElement`
- 线段上的以 Bernstein 函数为基函数的张量型有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1Pos_SegmentElement(const int p)` | 构造函数，构造阶数为 p 的正型H1线段有限元，方法中需要对 Nodes 赋值（对应于 Bernstein 基函数的结点） |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |

## 类 H1Pos_QuadrilateralElement
- `class H1Pos_QuadrilateralElement : public PositiveTensorFiniteElement`
- 矩形上的以 Bernstein 函数为1D基函数的张量型有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1Pos_QuadrilateralElement(const int p)` | 构造函数，构造阶数为 p 的正型H1矩形有限元，方法中需要对 Nodes 赋值（对应于 Bernstein 基函数的结点的2D张量积） |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |

## 类 H1Pos_HexahedronElement
- `class H1Pos_HexahedronElement : public PositiveTensorFiniteElement`
- 立方体上的以 Bernstein 函数为1D基函数的张量型有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1Pos_HexahedronElement(const int p)` | 构造函数，构造阶数为 p 的正型H1立方体有限元，方法中需要对 Nodes 赋值（对应于 Bernstein 基函数的结点的3D张量积） |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void ProjectDelta(int vertex, Vector &dofs) const` |  |

## 类 H1_TriangleElement
- `class H1_TriangleElement : public NodalFiniteElement`
- 三角形上任意阶结点型有限元，
  - 有限元结点是由线段（1D）上的结点生成的
  - 通过一个例子解释三角形单元上自由度的字典排序：对于三次元和三角形 $\Delta ABC$，假设其自由度按照自然排序排列，即自由度0、1、2分别代表顶点A、B和C上的自由度；自由度{3,4}、{5,6}、{7,8}分别代表边 $\overline{AB}$、$\overline{BC}$和$\overline{CA}$上的自由度；自由度9代表内部自由度。那么自由度的字典排序为 0-3-4-1-8-9-5-7-6-2 ，即按照从左到右、从下到上的顺序排列
  - 单元上的基函数表示为二维p阶多项式空间中某特殊的基（见类 poly1d 和 Ti 的构建）的线性组合的形式，组合系数保存在 Ti 中
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `DenseMatrixInverse Ti` | dof 阶的矩阵，其每一行代表了一个基函数的组合系数（见上面的解释） |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1_TriangleElement(const int p, const int btype = BasisType::GaussLobatto)` | 构造函数，构造三角形上阶数为 p 的，1D基函数类型为 btype 的结点型有限元，方法中需要对 Nodes ， lex_ordering 和 Ti 进行赋值，其中 Ti 实际上储存的是 Ti^(-1) 的LU分解 |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void CalcHessian(const IntegrationPoint &ip, DenseMatrix &Hessian) const` |  |

## 类 H1_TetrahedronElement
- `class H1_TetrahedronElement : public NodalFiniteElement`
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `mutable Vector shape_x, shape_y, shape_z, shape_l` |  |
| `mutable Vector dshape_x, dshape_y, dshape_z, dshape_l, u` |  |
| `mutable Vector ddshape_x, ddshape_y, ddshape_z, ddshape_l` |  |
| `mutable DenseMatrix du, ddu` |  |
| `DenseMatrixInverse Ti` |  |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1_TetrahedronElement(const int p, const int btype = BasisType::GaussLobatto)` |  |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `virtual void CalcHessian(const IntegrationPoint &ip, DenseMatrix &Hessian) const` |  |

## 类 H1Pos_TriangleElement
- `class H1Pos_TriangleElement : public PositiveFiniteElement`
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `mutable Vector m_shape, dshape_1d` |  |
| `mutable DenseMatrix m_dshape` |  |
| `dof_map` |  |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1Pos_TriangleElement(const int p)` |  |
| `static void CalcShape(const int p, const double x, const double y, double *shape)` |  |
| `static void CalcDShape(const int p, const double x, const double y, double *dshape_1d, double *dshape)` |  |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |

## 类 H1Pos_TetrahedronElement
- `class H1Pos_TetrahedronElement : public PositiveFiniteElement`
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `mutable Vector m_shape, dshape_1d` |  |
| `mutable DenseMatrix m_dshape` |  |
| `dof_map` |  |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1Pos_TetrahedronElement(const int p)` |  |
| `static void CalcShape(const int p, const double x, const double y, const double z, double *shape)` |  |
| `static void CalcDShape(const int p, const double x, const double y, const double z, double *dshape_1d, double *dshape)` |  |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` |  |
| `virtual void CalcDShape(const IntegrationPoint &ip DenseMatrix &dshape) const` |  |
| `` |  |
| `` |  |
| `` |  |
| `` |  |


## 类 PointFiniteElement
- `class PointFiniteElement : public NodalFiniteElement`
- 0D 点有限元
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `PointFiniteElement()` | 构造函数，调用`NodalFiniteElement(0, Geometry::POINT, 1, 0)`，并设置 Nodes.IntPoint(0).x 为 0 |
| `virtual void CalcShape(const IntegrationPoint &ip, Vector &shape) const` | （虚函数实例）设置 shape(0) = 1.0 |
| `virtual void CalcDShape(const IntegrationPoint &ip, DenseMatrix &dshape) const` | （虚函数实例）空函数 |
