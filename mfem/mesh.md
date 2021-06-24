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

# mesh.hpp & mesh.cpp
## 类 Mesh
### 类内结构体、类和枚举
-   ```C++
    // 用于表示一个与一个面相接的单元信息
    struct FaceInfo
    {
        int Elem1No, Elem2No, Elem1Inf, Elem2Inf;//用于刻画与面相接的两个单元（如果存在）的编号和“信息”，相应解释见下
        int NCFace; //如果NCFace=-1，那么该面是通常的一致/边界面
    };
    ```  
    -  一个面最多被两个单元所共有。 Elem1No 和 Elem2No 分别储存这两个面的序号，如果 Elem2No=-1 ，那么这个面只被一个单元拥有，即它是边界面； Elem1Inf 和 Elem2Inf 代表“信息”，储存了这个面在两个单元中的局部标号和顶点方向信息，储存方式如下：以单元1为例，令 `Elem1Inf=64*LFI+FO` ，那么 LFI 是面 F 在单元1中的局部标号，而 FO 是面 F 在单元1中的顶点方向。现在解释一下这两个概念，不妨网格中单元类型是一致的，每个单元对应了相同的参考元和不同的仿射变换，参考元中有一个既定的有序的顶点集合和每个面所对应的有序的顶点集合以及面标号。对于单元1，它对应了一个从参考元到实际单元的仿射变换，在这个仿射变换下，参考元标号为 LFI 的面 LF 变换为了面 F ，同时面 F 有一个相对于单元1的有序的顶点集合， FO 的值能够将面F实际的无序（或者按照顶点标号升序）顶点集合复原为这个有序的顶点集合，实际上只是做一个置换。对于单元1和单元2，这样的置换通常是不同的
-   ```C++
    struct NCFaceInfo
    {
        bool Slave;
        int MasterFace;
        const DenseMatrix* PointMatrix;
        NCFaceInfo() = default;
        NCFaceInfo(bool slave, int master, const DenseMatrix* pm): Slave(slave), MasterFace(master), PointMatrix(pm) {}
    }
    ```

### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `int Dim` | 维数，指网格所在的流形的维数 |
| `int spaceDim` | 空间维数，指网格所处于的欧式空间的维数 |
| `int NumOfVertices, NumOfElements, NumOfBdrElements` | 分别代表网格顶点、单元和边界单元的个数 |
| `int NumOfEdges, NumOfFaces` | 分别代表网格边和面的个数 |
| `mutable int nbInteriorFaces, nbBoundaryFaces` |  |
| `int meshgen` | 一个位标记符，用于表示网格中存在什么类别的单元—— </br> - 第1位： 网格中是否存在单纯形单元（点、线、三角形、四面体） </br> - 第2位：网格中是否存在张量型单元（四边形、六面体） </br> - 第3位：网格中是否存在楔单元（三棱柱） |
| `int mesh_geoms` | 一个位标识符，用于表示网格中存在什么几何对象—— </br> - 第1位：网格中是否存在 POINT </br> - 第2位：网格中是否存在 SEGMENT </br> - 第3位：网格中是否存在 TRIANGLE </br> - 第4位：网格中是否存在 SQUARE </br> - 第5位：网格中是否存在 TETRAHEDRON </br> - 第6位：网格中是否存在 CUBE </br> - 第7位：网格中是否存在 PRISM |
| `long sequence` |  |

#### 网格元素（单元、顶点、边界、边、面）和不同网格元素间联系的表
| 变量 | 解释 |
| ---- | ---- |
| `Array<Element *> elements` | 长为 NumOfElements 的数组，数组中的第 i 个元素是指向第 i 个单元的 Element 型指针 |
| `Array<Vertex> vertices` | 长为 NumOfVertices 的数组，数组中的第 i 个元素表示第 i 个顶点（Vertex型） |
| `Array<Element *> boundary` | 长为 NumOfBdrElements 的数组，数组中第 i 个元素是指向第 i 个边界单元的 Element 型指针。一个边界单元是一个最外层单元的一个面（边、端点），从而可以使用 Element 类刻画 |
| `Array<Element *> faces` | 长为 NumOfFaces 的数组，数组中的第 i 个元素是指向第 i 个面的 Element 型指针。面可以看做是低维的单元，从而可以使用 Element 类刻画 |
| `Array<FaceInfo> faces_info` | 长为 NumOfFaces 的数组，数组中的第 i 个元素对应着第 i 个面的面信息（见结构体 FaceInfo ） |
| `Array<NCFaceInfo> nc_faces_info` |  |
| `Table *el_to_edge` | 指向单元-边表的 Table 型指针，单元-边表储存了每个单元的边的标号信息 |
| `Table *el_to_face` | 指向单元-面表的 Table 型指针，单元-面表储存了每个单元的面的标号信息 |
| `Table *el_to_el` | 指向单元-单元表的 Table 型指针，单元-单元表（ Table 型）中，单元 I 和单元 II 有联系是指它们两个是相邻的 |
| `Array<int> be_to_edge` | 边界单元-边表（2D），边界单元-边表储存了每个边界单元的边的标号信息，二维情况下，一个边界单元只能对应一条边（实际上它本身就是一条边），从而可以用数组来刻画 |
| `Table *bel_to_edge` | 指向边界单元-边表的 Table 型指针（3D），边界单元-边表储存了每个边界单元的边（在网格中）的标号信息（每个边界单元的边是某个网格中的边） |
| `Array<int> be_to_face` | 边界单元-面表，边界单元-面表储存了每个边界单元所对应的面的标号信息，因为一个边界单元只能对应一个面（实际上它本身就是一个面），从而可以用数组来刻画 |
| `mutable Table *face_edge` | 指向面-边表的 Table 型指针，面-边表储存了每个面的边（在网格中）的标号信息 |
| `mutable Table *edge_vertex` | 指向边-顶点表的 Table 型指针，边-顶点表储存了每条边的顶点（在网格中）的标号信息 |
| `IsoparametricTransformation Transformation, Transformation2` |  |
| `IsoparametricTransformation BdrTransformation` |  |
| `IsoparametricTransformation FaceTransformation, EdgeTransformation` |  |
| `FaceElementTransformations FaceElemTr` |  |
| `CoarseFineTransformations CoarseFineTr` |  |
| `GridFunction *Nodes` |  |
| `int own_nodes` |  |
| `static const int vtk_quadratic_tet[10]` |  |
| `static const int vtk_quadratic_wedge[18]` |  |
| `static const int vtk_quadratic_hex[27]` |  |

### public 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `Array<int> attributes` | 按升序排列的数列，用于统计网格中单元所具有的所有的属性值 |
| `Array<int> bdr_attributes` | 按升序排列的数列，用于统计网格中边界单元所具有的所有的属性值 |
| `NURBSExtension *NURBSext` |  |
| `NCMesh *ncmesh` |  |
| `Array<GeometricFactors*> geom_factors` |  |
| `Array<FaceGeometricFactors*> face_geom_factors` |  |
| `Array<Triple<int, int, int> > tmp_vertex_parents` |  |
| `static bool remove_unused_vertices` |  |

### protected 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `void InitTables()` | 将表 el_to_edge ， el_to_face ， el_to_el ， bel_to_edge ， face_edge 和 edge_vertex 置为空 |
| `void DestroyTables()` | 释放 el_to_edge ， el_to_face ， el_to_el ，bel_to_edge（ 3D ） ， face_edge ， edge_vertex ，并调用`DeleteGeometricFactors()`释放几何因子  |
| `void Loader(std::istream &input, int generate_edges = 0, std::string parse_tag = "")` | 用于从流 input 中加载网格，其中  generate_edge 只用于方法 `ReadInlineMesh()` 中，而 parse_tag 为一个标识符，用于 mfem_v12 即以后的网格文件，用于标识流 input 中网格部分的结束。函数 Loader 中，首先通过调用不同的方法，针对不同类型的网格读取信息 Dim ， NumOfElement ， elements ， NumBdrElements ， boundary ， NumOfVertices （并相应地为 vertices 分配内存）， curved ； 接着，调用 FinalizeTopology() 生成相应的拓扑对象；最后，根据流 input 的不同完成 vertices 和 nodes 的构造 |
| `void PrepareNodeReorder(DSTable **old_v_to_v, Table **old_elem_vert)` | 在单元朝向改变之后，用于准备结点编号的重排，根据是否具有内部边结点和单元内部节点个数是否大于1，决定是否生成旧的顶点-顶点表 **old_v_to_v 和单元-顶点表 **old_elem_vert |
| `void DoNodeReorder(DSTable *old_v_to_v, Table *old_elem_vert)` | 进行结点编号的重排，同时，方法中对需要更新的各个拓扑对象均进行了更新 |
| `STable3D *GetElementToFaceTable(int ret_ftbl = 0)` | （仅限 3D ）生成单元-面表 el_to_face 和边界单元-面表 be_to_face ，方法中生成了一个辅助的 STable3D 型指针 faces_tbl ，用于表示每个面对应的顶点信息，如果 ret_ftbl=1 则返回这个指针，否则返回空指针。方法中要求变量 NumOfVertices ， elements ， NumOfElements ，boundary ， NumOfBdrElements （可以是0） 已被赋值，方法为变量 el_to_face ， NumOfFaces ， be_to_face 分配内存（如果需要）和赋值 |
| `void Mesh::GenerateFaces()` | 生成面 faces 和面信息 faces_info ，由于在不同维数时面所代表的意义不同，所以需要对不同维数讨论，在 1D 时，需要 NumOfVertices 已被赋值；在 2D 时，需要 NumOfEdges 和单元-边表 el_to_edge 已被赋值；在 3D 是，需要 NumOfFaces 和单元-面表 el_to_face 已被赋值；在所有维数都需要单元 elements 已被赋值。方法中调用 `AddXXXFaceElement()` 类的方法来最终建立 faces 和 faces_info |
| `void GetVertexToVertexTable(DSTable &v_to_v) const` | 生成 DSTable 对象 v_to_v，用于表示顶点和顶点间的联系（即边） |
| `static void GetElementArrayEdgeTable(const Array<Element*> &elem_array, const DSTable &v_to_v, Table &el_to_edge)` | （静态方法）给定顶点-顶点表 v_to_v （整体的），建立一列单元 elem_array （可以是一部分单元）上的单元-边表 el_to_edge |
| `int GetElementToEdgeTable(Table &e_to_f, Array<int> &be_to_f)` | 建立单元-边表 e_to_f 和边界单元-面表 be_to_f （2D 情况下）， 3D 时，方法另外建立 边界单元-边表 bel_to_edge （与 2D 情况下的边界单元-面表类似），方法要求 NumOfVertices ， elements ， NumOfBdrElements（可以为0） ， boundary 已被赋值 |
| `void SetMeshGen()` | 在单元 elements 已被赋值的基础上，确定 meshgen 和 mesh_geoms |
| `void MarkForRefinement()` | 对于三角形网格和四面体网格，通过置换单元顶点标号（不更改顶点的顺序），使得顶点0和顶点1对应的边是最长边（调用子方法 `MarkTriMeshForRefinement()` 和 `MarkTetMeshForRefinement()`） |
| `` |  |


### public 成员函数
#### 构造函数和析构函数
| 方法 | 解释 |
| ---- | ---- |
| `Mesh() { SetEmpty(); }` | 空构造函数，将网格置为空 |
| `explicit Mesh(const Mesh &mesh, bool copy_nodes = true)` | 拷贝构造函数，不拷贝单元-单元表 el_to_el 、面-边表 face_edge 和细化信息 sequence 、 last_operation ，如果 copy_nodes=1 并且源网格mesh  的结点信息存在，拷贝结点信息 |
| `Mesh(double *vertices, int num_vertices, int *element_indices, Geometry::Type element_type, int *element_attributes, int num_elements, int *boundary_indices, Geometry::Type boundary_type, int *boundary_attributes, int num_boundary_elements, int dimension, int space_dimension= -1)` | 从给定的原始数据出发构造网格，方法中调用了方法`FinalizeTopology()`，在本方法构造完毕或者设置网格结点 Nodes 后可以调用方法`Finalize()` |
| `Mesh(int _Dim, int NVert, int NElem, int NBdrElem = 0, int _spaceDim = -1)` | 网格初始化构造函数，只调用方法`InitMesh()`，确定维数 Dim ，顶点、单元和边界单元个数 NumOfVertices ,  NumOfElements ,  NumOfBdrElements |
| `Mesh(int nx, int ny, int nz, Element::Type type, bool generate_edges = false, double sx = 1.0, double sy = 1.0, double sz = 1.0, bool sfc_ordering = true)` | 构建平行六面体[0,sx]x[0,sy]x[0,sz]上的 type 型（ HEXAHEDRON 或 TETRAHEDRON ）的网格，如果 sfc_ordering=true 并且是六面体网格，单元将按照一条填满空间的曲线的顺序进行标号，否则将按照逐行、列、层排列（指六面体按照这样的顺序排列，四面体网格是六面体网格的细分） |
| `Mesh(int nx, int ny, Element::Type type, bool generate_edges = false, double sx = 1.0, double sy = 1.0, bool sfc_ordering = true)` | 构建平行四边形[0,sx]x[0,sy]上 type 型（ QUADRILATERAL 或 TRIANGLE ）的网格，如果 sfc_ordering=true 并且是四边形网格，单元将按照一条填满空间的曲线的顺序进行标号，否则将按照逐行、列排列（指四边形按照这样的顺序排列，三角形网格是四边形网格的细分） |
| `explicit Mesh(int n, double sx = 1.0)` | 将区间[0,sx]n等分构建一维网格 |
| `explicit Mesh(const char *filename, int generate_edges = 0, int refine = 1, bool fix_orientation = true)` | 从 MFEM ， Netgen 或 VTK 格式文件 filename 中读取网格 |
| `explicit Mesh(std::istream &input, int generate_edges = 0, int refine = 1, bool fix_orientation = true)` | 从 MFEM ， Netgen 或 VTK 格式的数据流中读取网格 |
| `Mesh(Mesh *mesh_array[], int num_pieces)` | 将 num_pieces 片网格组装成一个网格 |
| `Mesh(Mesh *orig_mesh, int ref_factor, int ref_type)` | 将起始网格 orig_mesh 进行细化 |
| `virtual ~Mesh()` | （虚函数）析构函数，释放分配的内存 |


#### 底层网格操作
##### 顶点、单元和边界元的新增
| 方法 | 解释 |
| ---- | ---- |
| `Element *NewElement(int geom)` | 返回指向一个 geom 型单元的指针（调用 `new XXX` 得到） |
| `int AddVertex(double x, double y = 0.0, double z = 0.0)` | 增加一个顶点 (x,y,z) ，返回新增后顶点的个数 |
| `int AddVertex(const double *coords)` | 增加一个顶点，其坐标保存在 *coords 中，返回新增后顶点的个数 |
| `void AddVertexParents(int i, int p1, int p2)` | <font color=yellow>与网格加密有关</font> |
| `int AddXXX(XXX)` | 新增特定类型的单元，返回新增后单元的个数 |
| `int AddBdrXXX(XXX)` | 新增特定类型的边界单元，返回新增后边界单元的个数 |
| `int AddElement(Element *elem)` | 增加单元 elem 到 elements 中，返回新增后单元的个数 |
| `int AddBdrElement(Element *elem);` | 增加 elem 到 boundary 中，返回新增后边界单元的个数 |
| `void GenerateBoundaryElements()` | （重新）生成边界单元 boundary 和 边界单元-边表 be_to_edge （2D）或边界单元-面表 be_to_face （3D），方法需要面 faces 和面信息 faces_info 已经赋值，需要注意在3D情形，边界单元-边表 bel_to_edge 将被重置为空且不更新 |

##### 完成网格的构建
| 方法 | 解释 |
| ---- | ---- |
| `void FinalizeTopology(bool generate_bdr = true)` | 此方法从已有的单元、顶点出发构造拓扑联系，不需要任意的坐标信息。要求 Dim ， NumOfElements ， elements ， NumOfVertices 已被定义。将生成面和边，具体得说：3D时，生成单元-面表 el_to_face 、边界单元 boundary （如果还未构建并且 generate_bdr 为真）、边界元-面表 be_to_face ，并且生成单元-边表 el_to_edge 、（3D）边界元-边表 bel_to_edge ；2D时，生成单元-边表 el_to_edge 、（2D）边界元-边表 be_to_edge 。不管是1D、2D还是3D，都会生成“面” faces 和“面”信息faces_info ，它们在不同维数时含义是不同的。并且将设置 attributes 和 bdr_attributes（调用`SetAttributes()`）<font color=yellow>ncmesh和tmp_vertex_parents待补充</font> |
| `void FinalizeXXXMesh(int generate_edges = 0, int refine = 0, bool fix_orientation = true)` | 针对特定类型，和方法 `Finalize()` 的功能一致 |
| `virtual void Finalize(bool refine = false, bool fix_orientation = false)` | （虚函数）两个功能：</br> - 检查和修正（根据 fix_orientation ）单元、边界元的朝向 </br> - 为细化做准备（根据 refine ，调用方法 `MarkForRefinement()`）</br> 这两个操作可能更改单元顶点的排列，从而需要重新建立拓扑联系（调用 `FinalizeTopology()`），值得注意的是如果 Nodes 存在，那么需要更改节点的顺序（调用 `PrepareNodeReorder` 和 `DoNodeReorder()`） |
| `void FinalizeMesh(int refine = 0, bool fix_orientation = true)` | 完成任意类型的网格的构建，调用了`FinalizeTopology()`和`Finalize()` |
| `virtual void SetAttributes()` | （虚函数）收集所有的单元和边界单元的属性到 attributes 和 bdr_attributes 中 |
| `double GetGeckoElementOrdering(Array<int> &ordering, int iterations = 4, int window = 4, int period = 2, int seed = 0, bool verbose = false, double time_limit = 0)` | Gecko 库接口，用于得到单元的某种填满空间的排序 |
| `void GetHilbertElementOrdering(Array<int> &ordering)` | 获得一种接近于 Hilbert 曲线的单元排序，储存在 ordering 中 |
| `void ReorderElements(const Array<int> &ordering, bool reorder_vertices = true)` | 将单元按照 ordering 的顺序重排，重新构建网格，<font color=yellow>细节待补充</font> |
| `void Load(std::istream &input, int generate_edges = 0, int refine = 1, bool fix_orientation = true)` | 从流 input 中加载网格，现有网格将被清除，功能和同参数的构造函数相同 |
| `void Clear()` | 清除网格内容 |
| `int MeshGenerator()` | 返回 meshgen |
| `int GetNV() const` | 返回顶点数 NumOfVertices |
| `int GetNE() const` | 返回单元数 NumOfElements |
| `int GetNBE() const` | 返回边界单元数 NumOfBdrElements |
| `int GetNEdges() const` | 返回边个数 NumOfEdges |
| `int GetNFaces() const` | 返回面个数 NumOfFaces |
| `int GetNumFaces() const` | 返回面（3D）、边（2D）或顶点（1D）数 |
| `int GetNFbyType(FaceType type) const` | 返回（真实）内部面个数 nbInteriorFaces 或（真实）边界面个数 nbBoundaryFaces ，取决于 type 是 FaceType::Interior 还是 FaceType::Boundary（<font color=yellow>需要进一步理解，连同 nbInteriorFaces 和 nbBoundaryFaces</font>） |
| `virtual long ReduceInt(int value) const` | （虚函数）返回所有处理器上值之和 |
| `long GetGlobalNE() const` | 返回总的单元个数（所有处理器上单元个数之和） |
| `const GeometricFactors* GetGeometricFactors(const IntegrationRule& ir, const int flags)` |  |
| `const FaceGeometricFactors* GetFaceGeometricFactors(const IntegrationRule& ir, const int flags, FaceType type)` |  |
| `void DeleteGeometricFactors()` |  |
| `int EulerNumber() const` | 返回欧拉数 |
| `int EulerNumber2D() const` | 返回2D欧拉数 |
| `int Dimension() const` | 返回维数 Dim |
| `int SpaceDimension() const` | 返回 spaceDim |
| `double *GetVertex(int i)` | 返回指向第 i 个顶点的指针 |
| `const double *GetVertex(int i) const` | 上条的const形式 |
| `void GetElementData(int geom, Array<int> &elem_vtx, Array<int> &attr) const` | 将所有geom型的单元对应的顶点标号放到elem_vtx中（展成一维数组），对应的类别信息放到attr中 |
| `void GetBdrElementData(int geom, Array<int> &bdr_elem_vtx, Array<int> &bdr_attr) const` | 将所有geom型的边界单元对应的顶点标号放到elem_vtx中（展成一维数组），对应的类别信息放到attr中 |
| `void ChangeVertexDataOwnership(double *vertices, int len_vertices, bool zerocopy = false)` | <font color=yellow>用处？待补充</font> |
| `const Element* const *GetElementsArray() const` | 返回指向所有单元的指针数组 |
| `Element *GetElement(int i)` | 返回指向第i个单元的Element*型指针 |
| `const Element *GetElement(int i) const` | 上条的const形式 |
| `Element *GetBdrElement(int i)` | 返回指向第i个边界单元的Element*指针 |
| `const Element *GetBdrElement(int i)` | 上条的const形式 |
| `const Element *GetFace(int i) const` | 返回指向第i个面的Element*型指针 |
| `Geometry::Type GetFaceBaseGeometry(int i) const` | 返回第i个面作为一个Element对象的几何类型 |
| `Geometry::Type GetElementBaseGeometry(int i) const` | 返回第i个单元作为一个Element对象的几何类型 |
| `Geometry::Type GetBdrElementBaseGeometry(int i) const` | 返回第i个边界单元作为一个Element对象的几何类型 |
| `bool HasGeometry(Geometry::Type geom) const` | 判断网格中的几何mesh_geoms是否包含了geom型的几何 |
| `int GetNumGeometries(int dim) const` | 统计网格中维数为dim的几何类型的个数 |
| `void GetGeometries(int dim, Array<Geometry::Type> &el_geoms) const` | 将网格中维数为dim的集合类型记录在el_geoms中 |
| `void GetElementVertices(int i, Array<int> &v) const` | 将第i个单元的顶点编号信息保存在v中 |
| `void GetBdrElementVertices(int i, Array<int> &v) const` | 将第i个边界单元的顶点编号信息保存在v中 |
| `void GetElementEdges(int i, Array<int> &edges, Array<int> &cor) const` | 将第i个单元的边的编号信息和朝向信息分别保存在edges和cor中 |
| `void GetBdrElementEdges(int i, Array<int> &edges, Array<int> &cor) const` | 将第i个边界单元的边的编号信息和朝向信息分别保存在edges和cor中 |
| `void GetFaceEdges(int i, Array<int> &edges, Array<int> &cor) const` | 将第i个面（2D时为边）的边的编号信息和朝向信息分别保存在edges和cor中 |
| `void GetFaceVertices(int i, Array<int> &vert) const` | 将第i个面（1D，2D，3D均可）的顶点信息保存在vert中 |
| `void GetEdgeVertices(int i, Array<int> &vert) const` | 将第i个边的顶点信息保存在vert中 |
| `Table *GetFaceEdgeTable() const` | 返回面-边表face_edge（3D），非3D时返回NULL |
| `Table *GetEdgeVertexTable() const` | 返回边-顶点表edge_vertex，如果不存在，则建立它 |
| `void GetElementFaces(int i, Array<int> &faces, Array<int> &cor) const` | 将第i个单元的面的编号信息和朝向信息分别保存在faces和cor中（1D，2D，3D均可） |
| `void GetBdrElementFace(int i, int *faces, int *cor) const` |  |
| `` | 将第i个边界单元的编号信息和朝向信息分别保存在faces和cor中（仅3D） |
| `int GetBdrElementEdgeIndex(int i) const` | 返回第i个边界单元对应的顶点（1D）/边（2D）/面（3D）的编号 |
| `void GetBdrElementAdjacentElement(int bdr_el, int &el, int &info) const` | 将第bdr_el个边界单元所临接的单元的编号储存在el中，并将此单元信息（即对应的faces_info中的Elm1_Inf）保存在info中 |
| `Element::Type GetElementType(int i) const` | 返回第i个单元的类型 |
| `Element::Type GetBdrElementType(int i) const` | 返回第i个边界单元的类型 |
| `void GetPointMatrix(int i, DenseMatrix &pointmat) const` | 将第i个单元的顶点坐标保存在矩阵pointmat中，pointmat的每一列代表一个顶点 |
| `void GetBdrPointMatrix(int i, DenseMatrix &pointmat) const` | 将第i个边界单元的顶点坐标保存在矩阵pointmat中，pointmat的每一列代表一个顶点 |
| `static FiniteElement *GetTransformationFEforElementType(Element::Type ElemType)` | 给定单元的几何类型 ElemType ，返回指向此类型参考元上的有限元（ PointFE, SegmentFE,  TriangleFE, QuadrilateralFE, TetrahedronFE, HexahedronFE, WedgeFE ） |
| `void GetElementTransformation(int i, IsoparametricTransformation *ElTr)` | 建立第 i 个单元上的等参变换，储存到 *ElTr 中，需要注意的是，如果 Nodes 为空，那么将使用参考元上的默认有限元，否则将从 Nodes 构造有限元（<font color=yellow>构造过程需要对 Nodes 进行深入了解</font>） |
| `ElementTransformation *GetElementTransformation(int i)` | 建立第 i 个单元上的等参变换 Transformation 并返回指向它的指针 |
| `void GetElementTransformation(int i, const Vector &nodes, IsoparametricTransformation *ElTr)` | 建立第 i 个单元上的等参变换，与方法 `void GetElementTransformation(int i, IsoparametricTransformation *ElTr)` 不同的是，此方法使用向量 nodes 中的数据代表单元的顶点\结点 |
| `void GetBdrElementTransformation(int i, IsoparametricTransformation *ElTr)` | 建立第 i 个边界单元上的等参变换，储存到 ElTr 中，需要注意的是，如果 Nodes 为空，那么将使用参考元上的默认有限元，否则将从 Nodes 构造有限元（<font color=yellow>构造过程需要对 Nodes 进行深入了解</font>） |
| `ElementTransformation * GetBdrElementTransformation(int i)` | 建立第 i 个边界单元上的等参变换 BdrTransformation 并返回指向它的指针 |
| `void GetFaceTransformation(int i, IsoparametricTransformation *FTr)` |  |
| `void GetLocalFaceTransformation(int face_type, int elem_type, IsoparametricTransformation &Transf, int info)` |  |
| `ElementTransformation *GetFaceTransformation(int FaceNo)` |  |
| `void GetEdgeTransformation(int i, IsoparametricTransformation *EdTr)` |  |
| `ElementTransformation *GetEdgeTransformation(int EdgeNo)` |  |
| `FaceElementTransformations *GetFaceElementTransformations(int FaceNo, int mask = 31)` |  |
| `FaceElementTransformations *GetInteriorFaceTransformations (int FaceNo)` |  |
| `FaceElementTransformations *GetBdrFaceTransformations (int BdrElemNo)` |  |
| `bool FaceIsInterior(int FaceNo) const` | 判断第 FaceNo 个面是否是内部面（通过 faces_info ） |
| `void GetFaceElements (int Face, int *Elem1, int *Elem2) const` | 将与第 Face 个面相邻接的两个面的编号分别储存在 *Elem1 和 *Elem2  中 |
| `void GetFaceInfos (int Face, int *Inf1, int *Inf2) const` | 将与第 Face 个面相邻接的两个面的面信息（ faces_info.Elem1Inf 和 faces_info.Elem2Inf ）分别储存在 *Inf1 和 *Inf2 中 |
| `Geometry::Type GetFaceGeometryType(int Face) const` | 返回第 Face 个面的几何类型 <font color=yellow>非一致网格待补充</font> |
| `Element::Type GetFaceElementType(int Face) const` | 返回第Face 个面的单元类型 |
| `int CheckElementOrientation(bool fix_it = true)` | 检查网格单元对应顶点排序是否是“正向”的，正向是指从参考元到单元的仿射变换的 Jacobian 行列式为正，返回非正向的单元数；如果 fix_it=1 ，则将非正向的单元修正为正向的（对 WEDGE 元和 HEXAHEDRON 元无效） |
| `int CheckBdrElementOrientation(bool fix_it = true)` |  |
| `int GetAttribute(int i) const` | 返回第 i 个单元的属性 |
| `void SetAttribute(int i, int attr)` | 将第 i 个单元的属性设置为 attr |
| `int GetBdrAttribute(int i) const` | 返回第 i 个边界单元的属性 |
| `const Table &ElementToElementTable()` | 返回单元-单元表 *el_to_el 的 const 引用，如果 el_to_el 不存在则将建立它<font color=yellow>建立过程中对非一致网格的处理待补充</font> |
| `const Table &ElementToFaceTable() const` | 返回单元-面表 *el_to_face 的 const 引用 |
| `const Table &ElementToEdgeTable() const` | 返回单元-边表 *el_to_edge 的 const 引用 |
| `Table *GetVertexToElementTable()` | 返回一个指向顶点-单元表的 Table 型指针，顶点-单元表（ Table ）中一个顶点I与单元II有连接是指顶点I是单元II的顶点，返回的指针最后必须被 delete |
| `Table *GetFaceToElementTable() const` | 返回一个指向面-单元表的 Table 型指针，面-单元表（ Table ）中一个面I与单元II有连接是指面I是单元II的面，返回的指针最后必须被delete |
| `virtual void ReorientTetMesh()` | （虚函数）用于建立四面体网格上的 Nedelec 空间 <font color=yellow>待补充</font> |
| `int *CartesianPartitioning(int nxyz[])` | 将网格单元进行笛卡尔分划，即首先确定网格所在的盒子[xmin,xmax]×[ymin,ymax]×[zmin,zmax]，接着将这个盒子进行剖分，每个方向上的剖分数分别为nxyz[0]，nxyz[1]，nxyz[2]，按照单元中心位于哪个小块儿之中确定单元所在的剖分，剖分的编号按照行、列、层的顺序排列，返回单元的分划信息（一个整型数组指针） |
| `int *GeneratePartitioning(int nparts, int part_method = 1)` | <font color=yellow>调用METIS进行单元分划，待补充</font>
| `void CheckPartitioning(int *partitioning)` | 检查空分划和非连通分划的个数，函数中生成el_to_el，但是最后重置其为NULL |
| `void FindPartitioningComponents(Table &elem_elem, const Array<int> &partitioning, Array<int> &component,Array<int> &num_comp)` | 子方法，以elem_elem所代表的单元间联系为等价关系统计每个分划的等价类个数，记录在partitioning中，而每个单元所在的等价类编号记录在component中 |
| `void CheckDisplacements(const Vector &displacements, double &tmax)` |  |
| `void MoveVertices(const Vector &displacements)` |  |
| `void GetVertices(Vector &vert_coord) const` |  |
| `void SetVertices(const Vector &vert_coord)` |  |
| `void GetNode(int i, double *coord) const` |  |
| `void SetNode(int i, const double *coord)` |  |
| `void MoveNodes(const Vector &displacements)` |  |
| `void GetNodes(Vector &node_coord) const` |  |
| `void SetNodes(const Vector &node_coord)` |  |
| `GridFunction *GetNodes()` |  |
| `const GridFunction *GetNodes() const` |  |
| `bool OwnsNodes() const` |  |
| `void SetNodesOwner(bool nodes_owner)` |  |
| `void NewNodes(GridFunction &nodes, bool make_owner = false)` |  |
| `void SwapNodes(GridFunction *&nodes, int &own_nodes_)` |  |
| `void GetNodes(GridFunction &nodes) const` |  |
| `void SetNodalFESpace(FiniteElementSpace *nfes)` |  |
| `void SetNodalGridFunction(GridFunction *nodes, bool make_owner = false)` |  |
| `const FiniteElementSpace *GetNodalFESpace() const` |  |
| `void EnsureNodes()` |  |
| `virtual void SetCurvature(int order, bool discont = false, int space_dim = -1, int ordering = 1)` |  |
| `void UniformRefinement(int ref_algo = 0)` |  |
| `void GeneralRefinement(const Array<Refinement> &refinements, int nonconforming = -1, int nc_limit = 0)` |  |
| `void GeneralRefinement(const Array<int> &el_to_refine, int nonconforming = -1, int nc_limit = 0)` |  |
| `void RandomRefinement(double prob, bool aniso = false,int nonconforming = -1, int nc_limit = 0)` |  |
| `void RefineAtVertex(const Vertex& vert, double eps = 0.0, int nonconforming = -1)` |  |
| `bool RefineByError(const Array<double> &elem_error, double threshold, int nonconforming = -1, int nc_limit = 0)` |  |
| `bool RefineByError(const Vector &elem_error, double threshold, int nonconforming = -1, int nc_limit = 0)` |  |
| `bool DerefineByError(Array<double> &elem_error, double threshold, int nc_limit = 0, int op = 1)` |  |
| `bool DerefineByError(const Vector &elem_error, double threshold, int nc_limit = 0, int op = 1)` |  |
| `void KnotInsert(Array<KnotVector *> &kv)` |  |
| `void KnotInsert(Array<Vector *> &kv)` |  |
| `void DegreeElevate(int rel_degree, int degree = 16)` |  |
| `void EnsureNCMesh(bool simplices_nonconforming = false)` |  |
| `bool Conforming() const` |  |
| `bool Nonconforming() const` |  |
| `const CoarseFineTransformations &GetRefinementTransforms()` |  |
| `Operation GetLastOperation() const` |  |
| `long GetSequence() const` |  |
| `virtual void PrintXG(std::ostream &out = mfem::out) const` | 将网格以Netgen/Truegrid格式打印到流out中 |
| `virtual void Print(std::ostream &out = mfem::out) const` | 将网格以默认的MFEM格式打印到流out中 |
| `void PrintVTK(std::ostream &out)` | 将网格以VTK格式打印到流out中（仅支持线性和二次网格） |
| `void PrintVTK(std::ostream &out, int ref, int field_data=0)` | <font color=yellow>其它参数的意义待补充</font> |
| `void PrintVTU(std::ostream &out, int ref=1, VTKFormat format=VTKFormat::ASCII, bool high_order_output=false, int compression_level=0, bool bdr_elements=false)` | 将网格以VTU格式打印到流out中<font color=yellow>其它参数的意义待补充</font> |
| `virtual void PrintVTU(std::string fname, VTKFormat format=VTKFormat::ASCII, bool high_order_output=false, int compression_level=0, bool bdr=false)` | 将网格以VTU格式打印到文件fname中<font color=yellow>其它参数的意义待补充</font> |
| `void PrintBdrVTU(std::string fname, VTKFormat format=VTKFormat::ASCII, bool high_order_output=false, int compression_level=0)` | 将网格以VTU格式打印到文件fname中<font color=yellow>其它参数的意义待补充</font> |
| `void GetElementColoring(Array<int> &colors, int el0 = 0)` | <font color=yellow>用在PrintVTK中，效用待补充</font> |
| `void PrintWithPartitioning (int *partitioning,std::ostream &out, int elem_attr = 0) const` |  |
| `void PrintElementsWithPartitioning (int *partitioning, std::ostream &out, int interior_faces = 0)` |  |
| `void PrintSurfaces(const Table &Aface_face, std::ostream &out) const` |  |
| `void ScaleSubdomains (double sf)` | 对子区域做放缩（拥有相同attr的单元视为一个子区域）<font color=yellow>奇怪的算法</font> |
| `void ScaleElements (double sf)` | 对单元做放缩<font color=yellow>奇怪的算法</font> |
| `void Transform(void (*f)(const Vector&, Vector&))` |  |
| `void Transform(VectorCoefficient &deformation)` |  |
| `void RemoveUnusedVertices()` |  |
| `void RemoveInternalBoundaries()` |  |
| `double GetElementSize(int i, int type = 0)` | 计算第i个单元的尺寸，令J为参考元到单元的Jacobi矩阵。type = 0意味着返回\|det(J)\|^(1/dim)；type = 1意味着返回J的最小奇异值；type = 2意味着返回J的最大奇异值 |
| `double GetElementSize(int i, const Vector &dir)` |  |
| `double GetElementVolume(int i)` |  |
| `void GetElementCenter(int i, Vector &center)` |  |
| `void GetBoundingBox(Vector &min, Vector &max, int ref = 2)` |  |
| `void GetCharacteristics(double &h_min, double &h_max,double &kappa_min, double &kappa_max, Vector *Vh = NULL, Vector *Vk = NULL)` |  |
| `static void PrintElementsByGeometry(int dim, const Array<int> &num_elems_by_geom, std::ostream &out)` |  |
| `void PrintCharacteristics(Vector *Vh = NULL, Vector *Vk = NULL, std::ostream &out = mfem::out)` |  |
| `virtual void PrintInfo(std::ostream &out = mfem::out)` |  |
| `void MesquiteSmooth(const int mesquite_option = 0)` |  |
| `virtual int FindPoints(DenseMatrix& point_mat, Array<int>& elem_ids, Array<IntegrationPoint>& ips, bool warn = true, InverseElementTransformation *inv_trans = NULL)` |  |

