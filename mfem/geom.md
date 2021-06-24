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

# geom.hpp & geom.cpp
## 类 Geomtry
- `class Geometry`
### 类内类、枚举和结构体
```C++
// public 枚举，用于表示（参考元）的几何类型
enum Type
{
    INVALID = -1,//无效的几何类型
    POINT = 0, //一个点0
    SEGMENT, //区间[0,1]
    TRIANGLE, //三角形，顶点为(0,0)，(0,1)，(1,0)
    SQUARE, //单位正方形(0,1)x(0,1)
    TETRAHEDRON,// 四面体，顶点为(0,0,0)，(1,0,0)，(0,1,0)，(0,0,1)
    CUBE, //单位正方体(0,1)x(0,1)x(0,1)
    PRISM,//三棱柱，顶点为(0,0,0)，(1,0,0)，(0,1,0)，(0,0,1)，(1,0,1)，(0,1,1)
    NUM_GEOMETRIES//几何的数目
};
```
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `IntegrationRule *GeomVert[NumGeom]` | 储存各种几何形状的顶点 |
| `IntegrationPoint GeomCenter[NumGeom]` | 储存各种几何形状的中点 |
| `DenseMatrix *GeomToPerfGeomJac[NumGeom], *PerfGeomToGeomJac[NumGeom]` | 对于每种几何，其均对应了一个“完美”的几何（比如三角形对应于正三角形），那么这种“不完美”的几何与“完美的”几何之间存在了一个变换，而这个变换可以由一个通过相应几何上的结点型线性元表达的 IsoparametricTransformation 对象所刻画，那么对于 GeomType 型几何，*GeomToPerfGeomJac[GeomType] 保存了这个变换的 Jacobian 矩阵（因为这个变换实际上是仿射变换，所以此 Jacobian 矩阵是常的）（此 Jacobian 矩阵是 dim x dim 维的， dim 是相应几何的维数，对于点几何，相应的指针为 NULL ），对应得， *PerfGeomToGeomJac[GeomType] 保存了 Jacobian 矩阵的逆 |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `Geometry()` | 构造函数，生成 GeomVert 、GeomCenter 、 GeomToPerfGeomJac 和 PerfGeomToGeomJac |
| `~Geometry()` | 析构函数，释放分配的内存 |
| `const IntegrationRule *GetVertices(int GeomType)` | 得到 GeomType 型几何的所有顶点信息，即以 const 形式返回 GeomVert[GermType] |
| `const IntegrationPoint &GetCenter(int GeomType)` | 得到 GeomType 型几何的中点，返回 GeomCenter[GeomType] 的 const 引用 |
| `static void GetRandomPoint(int GeomType, IntegrationPoint &ip)` | （静态方法）生成 GeomType 型几何中的一个随机点，保存在 ip 中 |
| `static bool CheckPoint(int GeomType, const IntegrationPoint &ip)` | （静态方法）检查点 ip 是否在 GeomType 型几何内 |
| `static bool CheckPoint(int GeomType, const IntegrationPoint &ip, double eps)` | （静态方法）检查点 ip 是否在 GeomType 型几何内，eps 是一个模糊阈值，用于规避浮点数造成的误差 |
| `static bool ProjectPoint(int GeomType, const IntegrationPoint &beg, IntegrationPoint &end)` | （静态方法）将一个点 end 投影到 GeomType 型几何内，如果 end 已经在其中，则返回真；否则返回假，并且计算点 beg （必须在几何内）与 end 之间连线与几何边界的交点，用这个点作为投影点，同时保存在 end 中 |
| `static bool ProjectPoint(int GeomType, IntegrationPoint &ip)` | （静态方法）将一个点 ip 投影到 GeomType 型几何内，如果 ip 已经在其中，则返回真；否则返回假，并且计算几何内与点 ip 距离最近的点，用这个点作为投影点，同时保存在 ip 中 |
| `DenseMatrix *GetPerfGeomToGeomJac(int GeomType)` | 返回指向 PerfGeomToGeomJac[GeomType] |
| `const DenseMatrix &GetGeomToPerfGeomJac(int GeomType) const` | 返回 *GeomToPerfGeomJac[GeomType] 的 const 引用 |
| `void GetPerfPointMat(int GeomType, DenseMatrix &pm)` | 得到“完美”的 GeomType 型几何的顶点，保存在矩阵 pm 中，即 pm 的每一列代表一个结点。“完美”的几何指正多边形或者正多面体 |
| `void JacToPerfJac(int GeomType, const DenseMatrix &J, DenseMatrix &PJ) const` | 对于建立在 GeomType 型几何上的映射 $\Phi$，其在点 $x$ 上的 Jacobian 为 $J$ ， 另外假设几何到“完美”几何的映射为 $F$ ，那么 $F$ 诱导的映射 $\hat{\Phi}=\Phi\circ F^{-1}$ 在 $\hat{x}=F(x)$ 上的 Jacobian 为 $PJ=J\cdot J_F^{-1}$ |
| `int NumBdr(int GeomType)` | 返回 GeomType 型几何面的个数 |

---------------

## 类 RefinedGeometry
- `class RefinedGeometry`
- 用于表示几何剖分的类，对于0维几何对象（点），剖分操作是不合法的；对于1维几何对象（线段），借助类 Quadrature1D 生成 Times+1 个剖分点，从而可以生成 Times 个子几何（需要注意这 Times+1 个剖分点中前后两个端点并不一定是线段的两个端点，对应于 1D 的开积分点）；对于2维和3维几何对象，将其每条边（ 1D 的几何对象）按照前述方法剖分成 Times 份，从而可以相继生成面剖分和体剖分，即最终生成对于几何的剖分，参数 Times 称为剖分的层数
- RefinedGeometry 类还提供了边剖分的过程，在剖分层数为 Times 的剖分的基础上，将几何每条边上的 Times 个子边均等得分为 ETimes 份，对应了 ETimes-1 个剖分点，从而这些剖分点可以生成一个对几何的更粗的剖分，我们可以用一系列（细）子边来刻画这个剖分中的（粗）子边（即所有粗子几何的边）， ETimes 称为边剖分的层数，它必须是 Times 的因数
- 用处？
### public 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int Times` | 剖分的层数 |
| `int ETimes` | 边剖分的层数 |
| `IntegrationRule RefPts` | 记录了所有剖分点（子几何顶点）的信息，其长度即为剖分点的个数 |
| `Array<int> RefGeoms` | 一个长为 Np*NGeoms 的数组，记录了子几何的顶点标号信息（即如果标号为 k ，则代表了 RefPts[k] 这个点），其中 Np 代表子几何顶点个数而 NGeoms 代表子几何的个数，数组中的数据每 Np 个一组，代表每个子几何的顶点标号 |
| `Array<int> RefEdges` | 一个长为 2*NEdge 的数组，记录了边剖分过程中用于刻画粗剖分（粗）子边的（细）子边端点编号信息，其中 NEdge 代表这样的（细）子边个数，数组中的数据每2个一组，代表了每条（细）子边的端点编号 |
| `int NumBdrEdges` | RefEdges 中一定保存了位于原几何的边上的（细）子边， NumBdrEdges 为这样的（细）子边的个数，即 RefEdges 中的前 2*NumBdrEdges 个元素表示了这样的（细）子边 |
| `int Type` | 用于生成1维剖分点的 Quadrature1D 类型 |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `RefinedGeometry(int NPts, int NRefG, int NRefE, int NBdrE = 0)` | 构造函数，设置 RefPts 长度为 NPts ， RefGeoms 和 RefEdges 的长度分别为 NRefG 和 NRefE ， NumBdrEdges = NBdrE （默认为0）  |

-------------------

## 类 GeometryRefiner
- `class GeometryRefiner`
- 用于生成和保存剖分的类
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int type` | Quadrature1D 类型，默认为 ClosedUniform |
| `Array<RefinedGeometry *> RGeom[Geometry::NumGeom]` | 和方法 Refine() 匹配；一个长为 `Geometry::NumGeom` 的 `Array<RefinedGeometry *>` 型数组，其中第 Geom 个元素是一个 RefinedGeometry* 型的数组，其中的每个元素对应了对几何对象 Geom 的一个剖分 |
| `Array<IntegrationRule *> IntPts[Geometry::NumGeom]` | 和方法 RefineInterior() 所匹配；一个长为 `Geometry::NumGeom` 的 `Array<IntegrationRule *>` 型数组，其中第 Geom 个元素是一个 IntegrationRule* 型的数组，其中的每个元素对应了对几何对象 Geom 的一个剖分中的内部积分点 |
### public 成员函数
#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `GeometryRefiner()` | 构造函数，设置 type 为 `Quadrature1D::ClosedUniform` |
| `~GeometryRefiner()` | 析构函数，释放为 RGeom 和 IntPts 分配的内存 |
| `void SetType(const int t)` | 设置 type 为 t |
| `int GetType() const` | 返回 type |
| `RefinedGeometry *Refine(Geometry::Type Geom, int Times, int ETimes = 1)` | 给定 Times 和 ETimes ，对几何类型 Geom 进行细化，如果对应于参数 type 、 Geom 、 Times 、 ETimes 的对象已经存在并保存在 RGeom 中，则直接从 RGeom 中提取相应的信息，否则将生成并保存在 RGeom 中 |
| `const IntegrationRule *RefineInterior(Geometry::Type Geom, int Times)` | 对于给定的 Times ，返回几何类型 Geom 细化后位于几何内部的点，注意到此方法总是使用 Quadrature1D::OpenUniform 类型的点，如果相应点的信息已经存在并保存在 IntPts 中，则直接从 IntPts 中提取相应的信息，否则将生成并保存在 IntPts 中 |
| `virtual int GetRefinementLevelFromPoints(Geometry::Type Geom, int Npts)` | （虚函数）根据点的个数 Npts 和几何类型 Geom ，推断细化的层数（即 Times ） |
| `virtual int GetRefinementLevelFromElems(Geometry::Type geom, int Nels)` | （虚函数）根据子几何的个数 Nels 和几何类型 geom ，推断细化的层数（即 Times） |

