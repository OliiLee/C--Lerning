# element.hpp & element.cpp
## （基）类Element
### 主要成员变量
- `int attribute` 记录单元的分划信息，用于对所有单元进行分类（可用于非重叠型区域分解）
- `Geometry::Type base_geom` 记录单元的几何信息（从参考元角度）：点（POINT）、线（SEGMENT）、三角形（TRIANGLE）、正方形（SQUARE）、四面体（TETRAHEDRON）、立方体（CUBE）、三棱柱（PRISM）
- `enum Type { POINT, SEGMENT, TRIANGLE, QUADRILATERAL,
               TETRAHEDRON, HEXAHEDRON, WEDGE
             };`记录单元的类型
### 主要成员函数
- `Type GetType()`：获取单元类型
- `Geometry::Type GetGeometryType()`：获取单元基本几何类型
- `int GetAttribute()`：获取单元的类别
- `void SetAttribute(const int attr)`：设置单元的类别
- `void SetVertices(const int *ind)`：设置单元顶点
- `void GetVertices(Array<int> &v)`\
  `int *GetVertices()`\
  `const int *GetVertices()`：获得单元顶点信息
- `int GetNVertices()`：获得单元顶点个数
- `int GetNEdges()`：获得单元边的个数
- `const int *GetEdgeVertices(int)`：获得单元某条边的顶点信息
- `int GetNFaces()`：获得单元面的个数
- `int GetNFaceVertices(int fi)`：获得单元某个面的顶点个数
- `const int *GetFaceVertices(int fi)`：获得单元某个面的顶点信息
- `void MarkEdge(const DSTable &v_to_v, const int *length)`
- `int NeedRefinement(HashTable<Hashed2> &v_to_v)`
- `void ResetTransform(int tr)`
- `void PushTransform(int tr)`
- `unsigned GetTransform()`
- `Element *Duplicate(Mesh *m)`