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

# intrules.hpp & intrules.cpp
## 类IntegrationPoint
- `class IntegrationPoint`
- 用于表示（1、2、3维）空间中带权的点
### public 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `double x, y, z` | 分别储存积分点的 x、y、z 轴坐标|
| `double weight` | 储存积分点对应的权重 |
| `int index` |  |
</br>

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `void Init(int const i)` | 初始化：x = y = z = weight = 0，int = i |
| `void Set(const double *p, const int dim)` | 令 x = p[0], y = p[1](dim > 1), z = p[2](dim = 3) |
| `void Set(const double x1, const double x2, const double x3, const double w)` | x = x1, y = x2, z = x3, weight = w |
| `void Get(double *p, const int dim) const` | 令 p[0] = x, p[1] = y(dim > 1), p[2] = z(dim = 3) |
| `void Set1w(const double *p)` | x = p[0], weight = p[1]，默认维数为1 |
| `void Set1w(const double x1, const double w)` | x = x1, weight = w，默认维数为1 |
| `void Set2(const double *p)` | x = p[0], y = p[1]，默认维数为2 |
| `void Set2(const double x1, const double x2)` | x = x1, y = x2，默认维数为2 |
| `void Set2w(const double *p)` | x = p[0], y = p[1], weight = p[2]，默认维数为2 |
| `void Set2w(const double x1, const double x2, const double w)` | x = x1, y = x2, weight = w，默认维数为2 |
| `void Set3(const double *p)` | x = p[0], y = p[1], z = p[3]，默认维数为3 |
| `void Set3(const double x1, const double x2, const double x3)` | x = x1, y = x2, z = x3，默认维数为3 |
| `void Set3w(const double *p)` | x = p[0], y = p[1], z = p[2], weight = p[3]，默认维数为3 |

---------------------

## 类IntegrationRule
- `class IntegrationRule : public Array<IntegrationPoint>`
- 表示积分规则的类，由一列IntegrationPoint派生得到
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int Order` |  |
| `mutable Array<double> weights` |  |
</br>

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `IntegrationRule()` | 构造一个空的IntegrationRule对象，即相应IntegrationPoint列置为空，Order=0 |
| `explicit IntegrationRule(int NP)` | 构建一个长度为NP的IntegrationPoint列，其中的每个元素（IntegrationPoint对象）调用Init()方法进行初始化，Order=0 |
| `IntegrationRule(IntegrationRule &irx, IntegrationRule &iry)` | 从两个1维的IntegrationRule对象irx和iry出发，构建张量形式的2维的IntegrationRule对象。例如：如果irx为{{x1,w1},{x2,w2}}，iry为{{y1,W1},{y2,W2}}，那么将生成{{(x1,y1),w1\*W1},{(x2,y1),w2\*W1},{(x1,y2),w1\*W2},{(x2,y2),w2\*W2}} |
| `IntegrationRule(IntegrationRule &irx, IntegrationRule &iry, IntegrationRule &irz)` | 从三个1维的IntegrationRule对象irx，iry和irz出发，构建张量形式的3维的IntegrationRule对象 |
| `int GetOrder() const` | 返回Order |
| `void SetOrder(const int order)` | 令Order = order |
| `int GetNPoints() const` | 返回积分点的个数，即IntegrationRule对象作为数列的长度 |
| `IntegrationPoint &IntPoint(int i)` | 返回第i个积分点的引用，(*this)[i] |
| `const IntegrationPoint &IntPoint(int i) const` | 上条的const形式 |
| `const Array<double> &GetWeights() const` | 用weights记录每个积分点对应于权重并返回 |
| `~IntegrationRule()` | 默认析构函数 |

-----------------

## 类QuadratureFunctions1D
- `class QuadratureFunctions1D`
### public成员函数
| 方法 | 解释 |
| ---- | ---- |
| `void GaussLegendre(const int np, IntegrationRule* ir)` | 计算（1维）Gauss-Legendre积分规则ir，np代表积分点的个数 |
| `void GaussLobatto(const int np, IntegrationRule *ir)` | 计算（1维）Gauss-Lobatto积分规则ir，np代表积分点的个数 |
| `void OpenUniform(const int np, IntegrationRule *ir)` | 计算（1维）开一致剖分Gauss-Cotes积分规则ir，即np个（内部）积分点构成了区间[0,1]的等距剖分 |
| `void ClosedUniform(const int np, IntegrationRule *ir)` | 计算（1维）闭一致剖分Gauss-Cotes积分规则ir，即np个积分点（其中第一个和最后一个积分点分别是区间端点0，1）构成了区间[0,1]的等距剖分 |
| `void OpenHalfUniform(const int np, IntegrationRule *ir)` | 计算（1维）开半一致剖分Gauss-Cotes积分规则ir，即np个积分点是区间[0,1]的np份等距剖分的中点 |
| `void ClosedGL(const int np, IntegrationRule *ir)` | 计算（1维）闭Gauss-Legendre积分规则ir，即积分点由2个端点和np-1个Gauss-Legendre点中相邻两个点的中点（共np-2个）所组成 |
| `void GivePolyPoints(const int np, double *pts, const int type)` | 根据type，选择调用前面的哪个方法，type与方法的对应见Quadrature1D中的匿名枚举 |

----------------------

## 类Quadrature1D
- `class Quadrature1D`
### 类中枚举
- ```C++
  enum
   {
      Invalid         = -1,
      GaussLegendre   = 0,
      GaussLobatto    = 1,
      OpenUniform     = 2,  
      ClosedUniform   = 3,  
      OpenHalfUniform = 4,  
      ClosedGL        = 5   
   };
  ```
  - 用于标识计算一维积分规则的方法，见类QuadratureFunctions1D
### public方法
| 方法 | 解释 |
| ---- | ---- |
| `static int CheckClosed(int type)` | 如果Quadrature1D type不是闭的则返回Invalid（-1），否则返回type <font color=yellow>似乎没考虑 type = ClosedGL</font> |
| `static int CheckOpen(int type)` | 如果Quadrature1D type不是开的则返回Invalid（-1），否则返回type（所有方法都可以作为开的）|

----------------

## 类IntegrationRules
- `class IntegrationRules`
### private成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const int quad_type` | 一维积分规则的标志（来自类Quadrature1D）中的匿名枚举，用于决定线段、正方形和正方体上的数值积分类型 |
| `int own_rules, refined` |  |
| `QuadratureFunctions1D quad_func` |  |
| `Array<IntegrationRule *> PointIntRules` |  |
| `Array<IntegrationRule *> SegmentIntRules` |  |
| `Array<IntegrationRule *> TriangleIntRules` |  |
| `Array<IntegrationRule *> SquareIntRules` |  |
| `Array<IntegrationRule *> TetrahedronIntRules` |  |
| `Array<IntegrationRule *> PrismIntRules` |  |
| `Array<IntegrationRule *> CubeIntRules` |  |
</br>

### public成员函数
| 方法 | 解释 |
| ---- | ---- |
| `explicit IntegrationRules(int Ref = 0, int type = Quadrature1D::GaussLegendre)` |  |
| `const IntegrationRule &Get(int GeomType, int Order)` |  |
| `void Set(int GeomType, int Order, IntegrationRule &IntRule)` |  |
| `void SetOwnRules(int o)` |  |
| `~IntegrationRules()` |  |

--------------

## 全局变量
- `extern IntegrationRules IntRules`
- `extern IntegrationRules RefinedIntRules`