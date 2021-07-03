# linearform.hpp & linearform.cpp
## 类 LinearForm
- `class LinearForm : public Vector`
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `FiniteElementSpace *fes` | LineraForm 对象所依赖的有限元空间，不被此对象所拥有 |
| `int extern_lfs` |  |
| `Array<LinearFormIntegrator*> dlfi` |  |
| `Array<Array<int>*>           dlfi_marker` |  |
| `Array<DeltaLFIntegrator*> dlfi_delta` |  |
| `Array<LinearFormIntegrator*> blfi` |  |
| `Array<Array<int>*>           blfi_marker` |  |
| `Array<LinearFormIntegrator*> flfi` |  |
| `Array<Array<int>*>           flfi_marker` |  |
| `Array<LinearFormIntegrator*> iflfi` |  |
| `Array<int> dlfi_delta_elem_id` |  |
| `Array<IntegrationPoint> dlfi_delta_ip` |  |
### private 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `LinearForm(const LinearForm &)` | 拷贝构造函数被禁用 |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `LinearForm(FiniteElementSpace *f)` | 构造依赖于有限元空间 *f 的 LinearForm 对象， f 不被此对象所拥有，即令向量长度为 f 中未知量个数，置 fes = f ， extern_lfs = 0 ， 并调用 `UseDevice(true)` |
| `LinearForm(FiniteElementSpace *f, LinearForm *lf)` | 构造依赖于有限元空间 *f 的 LinerForm 对象并使用与 lf 相同的积分子， f 不被此对象所拥有， lf 中的积分子作为指针被拷贝并且不被此对象所拥有 |
| `LinearForm()` | 构造一个空的 LinearForm 对象，即令 fes = NULL ， extern_lfs = 0 ， 并调用 `UseDevice(true)` ，此对象所依赖的有限元空间可以稍后通过调用方法 `Update(FiniteElementSpace *)` 或者 `Update(FiniteElementSpace *, Vector &, int)` 所设置 |
| `LinearForm(FiniteElementSpace *f, double *data)` |  |
| `LinearForm &operator=(const LinearForm &rhs)` |  |
| ` FiniteElementSpace *FESpace()` |  |
| `const FiniteElementSpace *FESpace()` |  |
| `void AddDomainIntegrator(LinearFormIntegrator *lfi)` |  |
| `void AddDomainIntegrator(LinearFormIntegrator *lfi, Array<int> &elem_marker)` |  |
| `void AddBoundaryIntegrator(LinearFormIntegrator *lfi)` |  |
| `void AddBoundaryIntegrator(LinearFormIntegrator *lfi, Array<int> &bdr_attr_marker)` |  |
| `void AddBdrFaceIntegrator(LinearFormIntegrator *lfi)` |  |
| `void AddBdrFaceIntegrator(LinearFormIntegrator *lfi, Array<int> &bdr_attr_marker)` |  |
| `void AddInteriorFaceIntegrator(LinearFormIntegrator *lfi)` |  |
| `Array<LinearFormIntegrator*> *GetDLFI()` |  |
| `Array<DeltaLFIntegrator*> *GetDLFI_Delta()` |  |
| `Array<LinearFormIntegrator*> *GetBLFI()` |  |
| `Array<LinearFormIntegrator*> *GetFLFI()` |  |
| `Array<Array<int>*> *GetFLFI_Marker()` |  |
| `void Assemble()` |  |
| `void AssembleDelta()` |  |
| `void Update()` |  |
| `void Update(FiniteElementSpace *f)` |  |
| `void Update(FiniteElementSpace *f, Vector &v, int v_offset)` |  |
| `virtual void MakeRef(FiniteElementSpace *f, Vector &v, int v_offset)` |  |
| `double operator()(const GridFunction &gf)` |  |
| `LinearForm &operator=(double value)` |  |
| `LinearForm &operator=(const Vector &v)` |  |
| `~LinearForm()` |  |
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