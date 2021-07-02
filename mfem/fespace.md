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

# fespace.hpp & fespace.cpp
## 类 Ordering
- `class Ordering`
### 类内类、枚举和结构体
-   ```C++
    // public 枚举，用于在向量维数（即每个自由度对应的未知量个数）大于1时，刻画所有未知量的排列顺序
    enum Type
    {
        byNODES, // 以结点（自由度） > 维度的顺序排列，即：XXX...,YYY...,ZZZ... 
        byVDIM   // 以 维度 > 结点（自由度）的顺序排列，即：XYZ,XYZ,XYZ,... 
    };
    ```
### public 成员函数
| 函数 | 解释 |
| ---- | ---- |
| `template <Type Ord> static inline int Map(int ndofs, int vdim, int dof, int vd)` | 对于 dof 自由度的第 vd 维的未知量，返回其在所有未知量中的标号（依照 byNODES 或者 byVDIM 的顺序） |
| `template <Type Ord> static void DofsToVDofs(int ndofs, int vdim, Array<int> &dofs)` | 对于 dofs 所代表的所有自由度对应的未知量，调用方法 Map() 计算它们在所有未知量中的标号，依旧储存在 dofs 中（需要更改 dofs 的长度）（依照 byNODES 或者 byVDIM 的顺序） |

-------------

## 类 FiniteElementSpace
- `class FiniteElementSpace`
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `Mesh *mesh` | FiniteElementSpace 对象所依赖的网格 |
| `const FiniteElementCollection *fec` |  |
| `int vdim` | 向量维数（每个自由度对应的未知量个数） |
| `Ordering::Type ordering` |  |
| `int ndofs` | 自由度的个数，未知量个数 = 自由度个数 * 向量维数 |
| `int nvdofs, nedofs, nfdofs, nbdofs` | 分别记录顶点自由度、边自由度、面自由度和 bubble 自由度的个数 |
| `int *fdofs, *bdofs` | int 型数组，分别记录面自由度和 bubble 自由度的分布信息，以 fdofs 为例，f[0] ~ f[1]-1 代表了第1个面上面自由度编号（即面自由度中的第 f[0] ~ f[1]-1 个自由度）。为什么需要 fdofs 和 bdofs ？实际上，因为网格中各个面和单元的几何可能不同，所以各个面（单元）对应的自由度个数可能是不同的  |
| `mutable Table *elem_dof` | 表 elem_dof 记录了各个单元对应自由的编号 |
| `mutable Table *bdrElem_dof` |  |
| `mutable Table *face_dof` |  |
| `mutable Array<int> face_to_be` |  |
| `Array<int> dof_elem_array, dof_ldof_array` | dof_elem_array 和 dof_ldof_array 用于表示整体-局部自由度（相对于单元）的对应关系，它们是长为 ndofs 的 Array 对象，第 i 个自由度对应了第 dof_elem_array[i] 个单元中的第 dof_ldof_array[i] 个自由度 |
| `NURBSExtension *NURBSext` |  |
| `int own_ext` |  |
| `mutable SparseMatrix *cP` |  |
| `mutable SparseMatrix *cR` |  |
| `mutable bool cP_is_set` |  |
| `OperatorHandle Th` |  |
| `mutable OperatorHandle L2E_nat, L2E_lex` |  |
| `mutable map_L2F L2F` |  |
| `mutable Array<QuadratureInterpolator*> E2Q_array` |  |
| `mutable Array<FaceQuadratureInterpolator*> E2IFQ_array` |  |
| `mutable Array<FaceQuadratureInterpolator*> E2BFQ_array` |  |
| `long sequence` | 应该与 mesh->GetSequence() 一致 |
| `long mesh_sequence` |  |
| `bool orders_changed;` |  |
| `bool relaxed_hp` |  |
### protected 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `void Construct()` | 帮助构造 fespace 对象，方法对 NURBS 空间或变阶数的协调空间是不适用的。设置 elem_dof = bdr_elem_dof = face_dof = NULL ，同时设置 cP = cR = cR_hp = NULL ， cP_is_set = false ；在设置各自由度个数时需要分为以下几种情况： <br> - 对于一般的网格，即每个单元、面、边上分别对应了相同的有限元，相应的自由度个数 nvdofs ， nedofs ， nfdofs 和 nbdofs 即分别是各个几何对象个数乘以相应几何对应的自由度个数的乘积 <br> - 对变阶数网格 <font color = yellow>边，待补充</font> <br> - 对于变阶数网格或者面上的有限元不一致 <font color = yellow>面，待补充</font> <br> - 对于变阶数网格或者单元上的有限元不一致 <font color = yellow>体，待补充</font> <br> 根据 mesh->GetSequence() 设置 mesh_sequence 并将 sequence 增加1，令 orders_changed = false |
| `void Constructor(Mesh *mesh, NURBSExtension *ext, const FiniteElementCollection *fec, int vdim = 1, int ordering = Ordering::byNODES)` | 用于构造 fespace 对象，根据输入的参数 mesh ， fec ， vdim 和 ordering 给 this 对象的相应变量赋值，并设置 sequence = 0 ， orders_changed = relaxed_hp = false ， 设置 Th 的类型为 Operator::ANY_TYPE 。根据是否是 NURBS 网格，有 <br> - 如果是 NURBS 网格<font color=yellow>待补充</font> <br> - 如果不是 NURBS 网格，设置 this->NURBSext = NULL 和 own_ext = 0 ，并调用 `Construct()` 完成构造 <br> 最终，调用 `BuildElementToDofTable()` 建立单元-自由度表 elem_dof ，并设置面-自由度表 face_dof = NULL |
| `void BuildElementToDofTable() const` | 通过调用方法 `GetElementDofs()` 构建单元-自由度表 elem_dof ，如果其已经存在，则直接返回 |
| `` |  |
| `` |  |

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `FiniteElementSpace()` | 默认构造函数，在用方法 Load() 初始化之前，对象是无效的 |
| `FiniteElementSpace(const FiniteElementSpace &orig, Mesh *mesh = NULL, const FiniteElementCollection *fec = NULL)` | 拷贝构造函数，对给定的 FiniteElementSpace 对象 orig 进行深度拷贝，但是对于成员变量 mesh 和 fec ，只有它们为 Null 时，才将其赋值为 orig.mesh 或者 orig.fec 。 mesh 和 fec 所指向的内容应该与 orig 中相应指针指向的内容相一致（或者二者互为拷贝）（调用方法 Constructor()） |
| `FiniteElementSpace(Mesh *mesh, const FiniteElementCollection *fec, int vdim = 1, int ordering = Ordering::byNODES)` | 使用给定的变量 mesh ， fec ， vdim 和 ordering 来构造 FiniteElementSpace 对象，将变量 NURBSext 置为空（调用方法 `Constructor()`） |
| `FiniteElementSpace(Mesh *mesh, NURBSExtension *ext, const FiniteElementCollection *fec, int vdim = 1, int ordering = Ordering::byNODES)` | 使用给定的变量 mesh ， ext ， fec ， vdim 和 ordering 来构造 FiniteElementSpace 对象（调用方法 `Constructor()`） |
| `Mesh *GetMesh() const` | 返回 mesh |
| `NURBSExtension *GetNURBSext()` | 返回 NURBSext |
| `const NURBSExtension *GetNURBSext() const` | 上条的 const 形式 |
| `NURBSExtension *StealNURBSext()` | 将 own_ext 置为0，返回 NURBSext（当 NURBSext 存在且 own_ext 为0时调用非法） |
| `bool Conforming() const` | 判断网格 mesh 是否是一致的 |
| `bool Nonconforming() const` | 判断网格 mesh 是否是非一致的 |
| `const SparseMatrix *GetConformingProlongation() const` |  |
| `const SparseMatrix *GetConformingRestriction() const` |  |
| `virtual const Operator *GetProlongationMatrix() const` |  |
| `virtual const SparseMatrix *GetRestrictionMatrix() const` |  |
| `const Operator *GetElementRestriction(ElementDofOrdering e_ordering) const` | 根据单元内自由度排列顺序 e_ordering 和有限元类型（是否间断）返回 L2E_nat.Ptr() 或者 L2E_lex.Ptr() ，如果它们为 NULL ，那么将调用方法`L2ElementRestriction()`或`ElementRestriction()`生成它们 |
| `virtual const Operator *GetFaceRestriction( ElementDofOrdering e_ordering, FaceType, L2FaceValues mul = L2FaceValues::DoubleValued) const` | <font color=yellow>结合 quadinterpolator.hpp</font> |
| `const QuadratureInterpolator *GetQuadratureInterpolator( const IntegrationRule &ir) const` | <font color=yellow>结合 quadinterpolator.hpp</font> |
| `const QuadratureInterpolator *GetQuadratureInterpolator( const QuadratureSpace &qs) const` | <font color=yellow>结合 quadinterpolator.hpp</font> |
| `const FaceQuadratureInterpolator *GetFaceQuadratureInterpolator( const IntegrationRule &ir, FaceType type) const` | <font color=yellow>结合 quadinterpolator_face.hpp</font> |
| `int GetVDim() const` | 返回 vdim |
| `int GetOrder(int i) const` | 返回第 i 个有限元（单元）的阶数 |
| `int GetFaceOrder(int i) const` | 返回第 i 个面有限元的阶数 |
| `int GetNDofs() const` | 返回 ndofs |
| `int GetVSize() const` | 返回未知量的个数，即 vdim * ndofs |
| `virtual int GetTrueVSize() const` | 返回（真正的）未知量的个数（调用 GetConformingVSize() ） |
| `int GetNConformingDofs() const` | 返回非一致网格上“真实的”自由度（即对应一致网格的自由度）个数（如果有限元空间建立在一个非一致网格上，非一致网格上可能会有悬点） |
| `int GetConformingVSize() const` | 返回“真实”未知量个数（即 vdim * GetNConformingDofs() ） |
| `Ordering::Type GetOrdering() const` | 返回 ordering |
| `const FiniteElementCollection *FEColl() const` | 返回 fec |
| `int GetNVDofs() const` | 返回 nvdofs |
| `int GetNEDofs() const` | 返回 nedofs |
| `int GetNFDofs() const` | 返回 nfdofs |
| `int GetNV() const` | 返回网格 mesh 中顶点的个数 mesh->GetNV() |
| `int GetNE() const` | 返回网格 mesh 中单元的个数 mesh->GetNE() |
| `int GetNF() const` | 返回网格 mesh 中面的个数 mesh->GetNF() |
| `int GetNBE() const` | 返回网格 mesh 中边界单元的个数 mesh->GetNBE() |
| `int GetNFbyType(FaceType type) const` | 返回网格 mesh 中 type 型的面的个数 mesh->GetNFbyType(type) |
| `int GetElementType(int i) const` | 返回网格 mesh 中第 i 个单元的类型 mesh->GetElementType(i) |
| `void GetElementVertices(int i, Array<int> &vertices) const` | 将网格 mesh 中第 i 个单元的顶点标号保存到 vertices 中， mesh->GetElementVertices(i, vertices) |
| `int GetBdrElementType(int i) const` | 返回网格 mesh 中第 i 个边界单元的类型 mesh->GetBdrElementType(i) |
| `ElementTransformation *GetElementTransformation(int i) const` | 返回网格 mesh 中第 i 个单元对应的变换 mesh->GetElementTransformation(i) |
| `void GetElementTransformation(int i, IsoparametricTransformation *ElTr)` | mesh->GetElementTransformation(i, ElTr) |
| `ElementTransformation *GetBdrElementTransformation(int i) const` | mesh->GetBdrElementTransformation(i) |
| `int GetAttribute(int i) const` | 返回网格 mesh 中第 i 个单元的分类信息 mesh->GetAttribute(i) |
| `int GetBdrAttribute(int i) const` | 返回网格 mesh 中第 i 个边界单元的分类信息 mesh->GetBdrAttribute(i) |
| `virtual void GetElementDofs(int i, Array<int> &dofs) const` | 获取第 i 个单元上的所有自由度（包括边界）编号，保存在 dofs 中，如果 elem_dof 存在，那么方法直接从 elem_dof 中获取信息，否则将从现有信息中建立（建立 elem_dof 也需要调用本方法） |
| `virtual void GetBdrElementDofs(int i, Array<int> &dofs) const` | 获取第 i 个边界单元上的所有自由度（包括边界）编号，保存在 dofs 中，如果 bdrElem_dof 存在，那么方法直接从 bdrElem_dof 中获取信息，否则将从现有信息中建立 |
| `virtual void GetFaceDofs(int i, Array<int> &dofs) const` | 获取第 i 个面（可以是 0D 、 1D 、 2D 面）上所有自由度（包括边界）编号，保存在 dofs 中，如果 face_dof 存在，那么方法直接从 face_dof 中获取信息，另外如果我们正在处理一个 NURBS 有限元空间，那么将调用方法 BuildNURBSFaceToDofTable() 建立 face_dof 再从中获取信息，否则将从现有信息中建立 |
| `void GetEdgeDofs(int i, Array<int> &dofs) const` | 获取第 i 条边上所有自由度（包括边界）编号，保存在 dofs 中 |
| `void GetVertexDofs(int i, Array<int> &dofs) const` | 获取第 i 个顶点上所有自由度编号，保存在 dofs 中 |
| `void GetElementInteriorDofs(int i, Array<int> &dofs) const` | 获取第 i 个单元内部自由度（ bubble 自由度）的编号，保存在 dofs 中 |
| `void GetFaceInteriorDofs(int i, Array<int> &dofs) const` | 获取第 i 个面内部自由度的编号，保存在 dofs 中 |
| `int GetNumElementInteriorDofs(int i) const` | 返回第 i 个单元内部自由度（ bubble 自由度）的个数 |
| `void GetEdgeInteriorDofs(int i, Array<int> &dofs) const` | 获取第 i 个边内部自由度的编号，保存在 dofs 中 |
| `void DofsToVDofs(Array<int> &dofs, int ndofs = -1) const` | 计算 dofs 中的自由度对应的所有未知量的编号，仍然保存在 dofs 中（未知量排列顺序由 ordering 决定）。本方法可以用于建立局部或者全局的自由度到未知量的转化，只需改变 ndofs 的大小（默认为全局） |
| `void DofsToVDofs(int vd, Array<int> &dofs, int ndofs = -1) const` | 计算 dofs 中的自由度上第 vd 维对应的未知量，仍然保存在 dofs 中（未知量排列顺序由 ordering 决定）。本方法可以用于建立局部或者全局的自由度到未知量的转化，只需改变 ndofs 的大小（默认为全局） |
| `int DofToVDof(int dof, int vd, int ndofs = -1) const` | 返回 dof 代表的自由度上第 vd 维对应的未知量的编号（未知量排列顺序由 ordering 决定）。本方法可以用于建立局部或者全局的自由度到未知量的转化，只需改变 ndofs 的大小（默认为全局） |
| `int VDofToDof(int vdof) const` | 返回 vdof 代表的未知量对应的自由度编号 |
| `static void AdjustVDofs(Array<int> &vdofs)` | （静态方法）对于 vdofs 中保存的未知量编号，将其中可能的负指标调整为对应的正指标 |
| `void GetElementVDofs(int i, Array<int> &vdofs) const` | 获取第 i 个单元上未知量的编号，保存在 vdofs 中 |
| `void GetBdrElementVDofs(int i, Array<int> &vdofs) const` | 获取第 i 个边界单元上未知量的编号，保存在 vdofs 中 |
| `void GetFaceVDofs(int i, Array<int> &vdofs) const` | 获取第 i 个面上未知量的编号，保存在 vdofs 中 |
| `void GetEdgeVDofs(int i, Array<int> &vdofs) const` | 获取第 i 条边上未知量的编号，保存在 vdofs 中 |
| `void GetVertexVDofs(int i, Array<int> &vdofs) const` | 获取第 i 个顶点上未知量的编号，保存在 vdofs 中 |
| `void GetElementInteriorVDofs(int i, Array<int> &vdofs) const` | 获取第 i 个单元内部（对应于 bubble 自由度）未知量的编号，保存在 vdofs 中 |
| `void GetEdgeInteriorVDofs(int i, Array<int> &vdofs) const` | 获取第 i 条边内部未知量的编号，保存在 vdofs 中 |
| `void RebuildElementToDofTable()` | 重新建立表 elem_dof |
| `void ReorderElementToDofTable()` | 对自由度进行重排，规则可见例子：令{(-1,4,3),(2,4,3,-5)}代表了两个单元上的自由度，那么重排后的自由度为{(-1,2,3),(4,2,3,-5)} |
| `const Table &GetElementToDofTable() const` | 返回 *elem_dof 的 const 引用 |
| `const Table &GetBdrElementToDofTable() const` | 返回 *bdrElem_dof 的 const 引用，如果它不存在（为 NULL ），则调用方法 BuildBdrElementToDofTable() 生成它 |
| `const Table &GetFaceToDofTable() const` | 返回 *face_dof 的 const 引用，如果它不存在（为 NULL ），则调用方法 BuildFaceToDofTable() 生成它 |
| `void BuildDofToArrays()` | 生成 dof_elem_array 和 dof_ldof_array，如果它们已经存在，则直接返回 |
| `int GetElementForDof(int i) const` | 返回 dof_elem_array[i] |
| `int GetLocalDofForDof(int i) const` | 返回 dof_ldof_array[i] |
| `const FiniteElement *GetFE(int i) const` | 返回指向 fec 中对应于第 i 个单元的 FiniteElement 对象的指针 |
| `const FiniteElement *GetBE(int i) const` | 返回指向 fec 中对应于第 i 个边界单元的 FiniteElement 对象的指针 |
| `const FiniteElement *GetFaceElement(int i) const` | 返回指向 fec 中对应于第 i 个面的 FiniteElement 对象的指针 |
| `const FiniteElement *GetEdgeElement(int i) const` | 返回指向 fec 中对应于第 i 条边的 FiniteElement 对象的指针 |
| `const FiniteElement *GetTraceElement(int i, Geometry::Type geom_type) const` |  |
| `virtual void GetEssentialVDofs(const Array<int> &bdr_attr_is_ess, Array<int> &ess_vdofs, int component = -1) const` | （虚函数）标记“重要”的边界单元上的（某些）未知量， bdr_attr_is_ess 是一个长度为边界分类个数的 Boolean 型的向量，用于标记分类为 attr 的边界单元是否是“重要”的（即 bdr_attr_is_ess[attr-1] 是否为零），ess_vdofs 是一个长度为未知量个数的向量，用于刻画每个未知量是否是“重要”的（等于-1意味着是“重要”的，等于0则意味着不“重要”），如果 component < 0 （默认），那么将标记所有“重要”的自由度对应的未知量，否则只标记位于第 component 维的未知量 <font color = yellow>对于非一致网格需要一些特殊的处理，待补充</font> |
| `virtual void GetEssentialTrueDofs(const Array<int> &bdr_attr_is_ess, Array<int> &ess_tdof_list, int component = -1)` | （虚函数）得到“重要”的边界单元上的“真实”未知量标号，与 GetEssentialVDofs() 方法不同的是，向量 ess_tdof_list 中保存的是“重要”的真实未知量的指标（<font color=yellow>对于非一致网格，真实的概念待补充</font>） |
| `void GetBoundaryTrueDofs(Array<int> &boundary_dofs, int component = -1)` | 得到边界上（某些）自由度的编号信息，保存到 boundary_dofs 中，如果 component < 0 （默认），那么将获取所有的自由度，否则只获取位于第 component 维的未知量。如果网格无边界（即边界类别数为0），将释放 boundary_dofs  |
| `static void MarkerToList(const Array<int> &marker, Array<int> &list)` | （静态方法）将 boolean 的标记向量 marker 中带标记（即 marker 中对应的值非零）的指标保存到 list 中 |
| `static void ListToMarker(const Array<int> &list, int marker_size, Array<int> &marker, int mark_val = -1)` | （静态方法）将 list 中储存的指标转化为 boolean 型的标记向量 marker ，其中 marker 的长度（即所有元素的个数）为 marker_size ，而标记值为 mark_val |
| `void ConvertToConformingVDofs(const Array<int> &dofs, Array<int> &cdofs)` |  |
| `void ConvertFromConformingVDofs(const Array<int> &cdofs, Array<int> &dofs)` |  |
| `SparseMatrix *D2C_GlobalRestrictionMatrix(FiniteElementSpace *cfes)` | 建立起同阶间断有限元空间和连续有限元空间之间未知量的全局限制矩阵 R 并返回。方法假定 *this 有限元空间是间断的，对于给定的连续有限元空间 *cfes ，R 是一个 c_nvdofs x d_nvdofs 维的稀疏矩阵， R(i,j) = 1 表示 *cfes 中第 i 个未知量和 *this 中的第 j 个未知量对应了相同的“物理”中的点，并且未知量对应的维度是相同的（对于间断有限元空间，同一个“物理”中的点可能对应了多个自由度），也就是说，R 的第 k 列代表着 *this 中第 k 个未知量在 *cfes 上的限制 |
| `SparseMatrix *D2Const_GlobalRestrictionMatrix(FiniteElementSpace *cfes)` |  |
| `SparseMatrix *H2L_GlobalRestrictionMatrix(FiniteElementSpace *lfes)` |  |
| `void GetTransferOperator(const FiniteElementSpace &coarse_fes, OperatorHandle &T) const` |  |
| `virtual void GetTrueTransferOperator(const FiniteElementSpace &coarse_fes, OperatorHandle &T) const` |  |
| `virtual void Update(bool want_transform = true)` |  |
| `const Operator* GetUpdateOperator()` |  |
| `void GetUpdateOperator(OperatorHandle &T)` |  |
| `void SetUpdateOperatorOwner(bool own)` |  |
| `void SetUpdateOperatorType(Operator::Type tid)` |  |
| `virtual void UpdatesFinished()` |  |
| `long GetSequence() const` | 返回 sequence |
| `bool IsDGSpace() const` | 判断有限元空间是否是连续的 |
| `void Save(std::ostream &out) const` | 将当前 FiniteElementSpace 对象保存到流 out 中 |
| `FiniteElementCollection *Load(Mesh *m, std::istream &input)` | 从流 input 中读取 FiniteElementSpace 对象，返回的 FiniteElementCollection 型指针将被此对象所拥有 |
| `virtual ~FiniteElementSpace()` | （虚函数）析构函数，释放分配的内存 |
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
