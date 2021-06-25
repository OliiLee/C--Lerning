# fe_coll.hpp & fe_coll.cpp
## 类 FiniteElementCollection
- `class FiniteElementCollection`
### 类内类、枚举和结构体
-   ```C++
    // public 枚举，对 ContType 的枚举
    enum 
    { 
        CONTINUOUS,   // 场跨过单元边界是连续的
        TANGENTIAL,   // 向量场的切向分量是连续的
        NORMAL,       // 向量场的法向分量是连续的
        DISCONTINUOUS // 场跨过单元边界是不连续的
    };

    ```
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `virtual const FiniteElement * FiniteElementForGeometry(Geometry::Type GeomType) const = 0` | （纯虚函数）返回一个 const FiniteElement 指针，其指向的 FiniteElement 对象是属于当前 FiniteElementCollection 类型的几何类型为 GeomType 的有限元 |
| `virtual int DofForGeometry(Geometry::Type GeomType) const = 0` | （纯虚函数）返回属于当前 FiniteElementCollection 类型的几何类型为 GeomType 的有限元上的自由度 |
| `virtual const int *DofOrderForOrientation(Geometry::Type GeomType, int Or) const = 0` | （纯虚函数）返回一个数组，其值对应着当前 FiniteElementCollection 类型的几何类型为 GeomType 的有限元自由度的某个置换，由 Or 所决定 |
| `virtual const char * Name() const` | （虚函数）返回当前 FiniteElementCollection 对象的名称（即类型） |
| `virtual int GetContType() const = 0` | （虚函数）返回当前 FiniteElementCollection 对象的连续性（ ContType ）（见类内枚举） |
| `int HasFaceDofs(Geometry::Type GeomType) const` | 返回属于当前 FiniteElementCollection 类型的几何类型为 GeomType 的有限元面上自由度个数（面可以视为一个新的有限元，如果 GeomType 具有两种不同几何类型的面，那么返回最大自由度个数），否则返回0 |
| `virtual const FiniteElement *TraceFiniteElementForGeometry( Geometry::Type GeomType) const` | （虚函数）<font color=yellow>意义需要补充</font> 默认返回 FiniteElementForGeometry(GeomType) |
| `virtual FiniteElementCollection *GetTraceCollection() const` | （虚函数）<font color=yellow>意义需要补充</font> |
| `virtual ~FiniteElementCollection()` | （虚函数）析构函数 |
| `static FiniteElementCollection *New(const char *name)` | 返回一个 FiniteElementCollection 指针，其指向名为 name 的 FiniteElementCollection 对象 |
| `void SubDofOrder(Geometry::Type Geom, int SDim, int Info, Array<int> &dofs) const` | 意义不明？有时间再研究下代码，或者等遇到再说 |

| FEC Name | Space | Order | BasisType | FiniteElement::MapT | Notes |
| :------: | :---: | :---: | :-------: | :-----: | :---: |
| H1_[DIM]_[ORDER] | H1 | * | 1 | VALUE | H1 nodal elements |
| H1@[BTYPE]_[DIM]_[ORDER] | H1 | * | * | VALUE | H1 nodal elements |
| H1Pos_[DIM]_[ORDER] | H1 | * | 1 | VALUE | H1 nodal elements |
| H1Pos_Trace_[DIM]_[ORDER] | H^{1/2} | * | 2 | VALUE | H^{1/2}-conforming trace elements for H1 defined on the interface between mesh elements (faces,edges,vertices) |
| H1_Trace_[DIM]_[ORDER] | H^{1/2} | * | 1 | VALUE | H^{1/2}-conforming trace elements for H1 defined on the interface between mesh elements (faces,edges,vertices) |
| H1_Trace@[BTYPE]_[DIM]_[ORDER] | H^{1/2} | * | 1 | VALUE | H^{1/2}-conforming trace elements for H1 defined on the interface between mesh elements (faces,edges,vertices) |
| ND_[DIM]_[ORDER] | H(curl) | * | 1 / 0 | H_CURL | Nedelec vector elements |
| ND@[CBTYPE][OBTYPE]_[DIM]_[ORDER] | H(curl) | * | * / * | H_CURL | Nedelec vector elements |
| ND_Trace_[DIM]_[ORDER] | H^{1/2} | * | 1 / 0  | H_CURL | H^{1/2}-conforming trace elements for H(curl) defined on the interface between mesh elements (faces) |
| ND_Trace@[CBTYPE][OBTYPE]_[DIM]_[ORDER] | H^{1/2} | * | 1 / 0 | H_CURL | H^{1/2}-conforming trace elements for H(curl) defined on the interface between mesh elements (faces) |
| RT_[DIM]_[ORDER] | H(div) | * | 1 / 0 | H_DIV | Raviart-Thomas vector elements |
| RT@[CBTYPE][OBTYPE]_[DIM]_[ORDER] | H(div) | * | * / * | H_DIV | Raviart-Thomas vector elements |
| RT_Trace_[DIM]_[ORDER] | H^{1/2} | * | 1 / 0 | INTEGRAL | H^{1/2}-conforming trace elements for H(div) defined on the interface between mesh elements (faces) |
| RT_ValTrace_[DIM]_[ORDER] | H^{1/2} | * | 1 / 0 | VALUE | H^{1/2}-conforming trace elements for H(div) defined on the interface between mesh elements (faces) |
| RT_Trace@[BTYPE]_[DIM]_[ORDER] | H^{1/2} | * | 1 / 0 | INTEGRAL | H^{1/2}-conforming trace elements for H(div) defined on the interface between mesh elements (faces) |
| RT_ValTrace@[BTYPE]_[DIM]_[ORDER] |  H^{1/2} | * | 1 / 0 | VALUE | H^{1/2}-conforming trace elements for H(div) defined on the interface between mesh elements (faces) |
| L2_[DIM]_[ORDER] | L2 | * | 0 | VALUE | Discontinous L2 elements |
| L2_T[BTYPE]_[DIM]_[ORDER] | L2 | * | 0 | VALUE | Discontinous L2 elements |
| L2Int_[DIM]_[ORDER] | L2 | * | 0 | INTEGRAL | Discontinous L2 elements |
| L2Int_T[BTYPE]_[DIM]_[ORDER] | L2 | * | 0 | INTEGRAL | Discontinous L2 elements |
| DG_Iface_[DIM]_[ORDER] | - | * | 0 | VALUE | Discontinuous elements on the interface between mesh elements (faces) |
| DG_Iface@[BTYPE]_[DIM]_[ORDER] | - | * | 0 | VALUE | Discontinuous elements on the interface between mesh elements (faces) |
| DG_IntIface_[DIM]_[ORDER] | - | * | 0 | INTEGRAL | Discontinuous elements on the interface between mesh elements (faces) |
| DG_IntIface@[BTYPE]_[DIM]_[ORDER] | - | * | 0 | INTEGRAL | Discontinuous elements on the interface between mesh elements (faces) |
| NURBS[ORDER] | - | * | - | VALUE | Non-Uniform Rational B-Splines (NURBS) elements |
| LinearNonConf3D | - | 1 | 1 | VALUE | Piecewise-linear nonconforming finite elements in 3D |
| CrouzeixRaviart | - | - | - | - | Crouzeix-Raviart nonconforming elements in 2D |
| Local_[FENAME] | - | - | - | - | Special collection that builds a local version out of the FENAME collection |
|-|-|-|-|-|-|
| Linear | H1 | 1 | 1 | VALUE | Left in for backward compatibility, consider using H1_ |
| Quadratic | H1 | 2 | 1 | VALUE | Left in for backward compatibility, consider using H1_ |
| QuadraticPos | H1 | 2 | 2 | VALUE | Left in for backward compatibility, consider using H1_ |
| Cubic | H1 | 2 | 1 | VALUE | Left in for backward compatibility, consider using H1_ |
| Const2D | L2 | 0 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| Const3D | L2 | 0 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| LinearDiscont2D | L2 | 1 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| GaussLinearDiscont2D | L2 | 1 | 0 | VALUE | Left in for backward compatibility, consider using L2_ |
| P1OnQuad | H1 | 1 | 1 | VALUE | Linear P1 element with 3 nodes on a square |
| QuadraticDiscont2D | L2 | 2 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| QuadraticPosDiscont2D | L2 | 2 | 2 | VALUE | Left in for backward compatibility, consider using L2_ |
| GaussQuadraticDiscont2D | L2 | 2 | 0 | VALUE | Left in for backward compatibility, consider using L2_ |
| CubicDiscont2D | L2 | 3 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| LinearDiscont3D | L2 | 1 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| QuadraticDiscont3D | L2 | 2 | 1 | VALUE | Left in for backward compatibility, consider using L2_ |
| ND1_3D | H(Curl) | 1 | 1 / 0 | H_CURL | Left in for backward compatibility, consider using ND_ |
| RT0_2D | H(Div) | 1 | 1 / 0 | H_DIV | Left in for backward compatibility, consider using RT_ |
| RT1_2D | H(Div) | 2 | 1 / 0 | H_DIV | Left in for backward compatibility, consider using RT_ |
| RT2_2D | H(Div) | 3 | 1 / 0 | H_DIV | Left in for backward compatibility, consider using RT_ |
| RT0_3D | H(Div) | 1 | 1 / 0 | H_DIV | Left in for backward compatibility, consider using RT_ |
| RT1_3D | H(Div) | 2 | 1 / 0 | H_DIV | Left in for backward compatibility, consider using RT_ |

| Tag | Description |
| :------: | :--------: |
| [DIM]    | Dimension of the elements (1D, 2D, 3D) |
| [ORDER]  | Approximation order of the elements (P0, P1, P2, ...) |
| [BTYPE]  | BasisType of the element (0-GaussLegendre, 1 - GaussLobatto, 2-Bernstein, 3-OpenUniform, 4-CloseUniform, 5-OpenHalfUniform) |
| [OBTYPE] | Open BasisType of the element for elements which have both types |
| [CBTYPE] | Closed BasisType of the element for elements which have both types |

## 类 H1_FECollection
- `class H1_FECollection : public FiniteElementCollection`

### protected 成员变量
| 变量 | 解释 | 
| ---- | ---- |
| `int b_type` | 基类型，见类 `BasisType` |
| `char h1_name[32]` | 当前 H1 协调元的名称 |
| `FiniteElement *H1_Elements[Geometry::NumGeom]` | 长为 Geometry::NumGeom 的 FiniteElement* 型的数组，储存各个几何对象上（对应于阶数和基类型）的有限元  |
| `int H1_dof[Geometry::NumGeom]` | 长为 Geometry::NumGeom 的 int 型数组，储存各个几何对象上自由度个数 |
| `int *SegDofOrd[2], *TriDofOrd[6], *QuadDofOrd[8], *TetDofOrd[24]` | 分别用于表示线段、三角形、矩形、四面体上自由度的不同排列（朝向）信息（结合方法 `mesh::GetXXXOrientation`） |

### public 成员函数
#### 构造函数和析构函数
| 方法 | 解释 | 
| ---- | ---- |
| `explicit H1_FECollection(const int p, const int dim = 3, const int btype = BasisType::GaussLobatto)` | 构造函数，根据输入的阶数 p ，维数 dim 和基类型 btype，为成员变量 b_type ， h1_name ， H1_Elements ， H1_dof ， XXXDofOrd 赋值，在 btype 为 BasisType::Serendipity 时需要特殊的处理 |
| `virtual ~H1_FECollection()` | （虚函数实例）析构函数，释放 H1_Elements ，SegDofOrd ， TriDofOrd ， QuadDofOrd ， TetDofOrd |
#### 信息获取
| 方法 | 解释 | 
| ---- | ---- |
| `virtual const FiniteElement *FiniteElementForGeometry( Geometry::Type GeomType) const` | （虚函数实例）返回 H1_Elements[GeomType] |
| `virtual int DofForGeometry(Geometry::Type GeomType) const` | （虚函数实例）返回 H1_dof[GeomType] |
| `virtual const int *DofOrderForOrientation(Geometry::Type GeomType, int Or) const` | （虚函数实例）根据 GeomType 返回 XXXDofOrd[Or%Num] 其中 Num 为所有朝向的数目，即 XXXDofOrd 的长度 |
| `virtual const char *Name() const` | （虚函数实例）返回 h1_name |
| `virtual int GetContType() const` | （虚函数实例）返回 CONTINUOUS |
| `int GetBasisType() const` | 返回 btype |
| `const int *GetDofMap(Geometry::Type GeomType) const` |  |
| `FiniteElementCollection *GetTraceCollection() const` | <font color = yellow>见类 H1_Trace_FECollection</font> |


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
