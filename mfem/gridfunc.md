# gridfunc.hpp & gridfunc.cpp
## 类 GridFunction
- `class GridFunction : public Vector`
### protected 成员变量
| `FiniteElementSpace *fes` |  |
| `FiniteElementCollection *fec` |  |
| `Vector t_vec` |  |
### public 成员函数
| `GridFunction()` | 默认构造函数，置 fes = fec = NULL ； fes_sequence = 0 ；并调用 `UseDevice(true)` |
| `GridFunction(const GridFunction &orig)` | 拷贝构造函数，注意到 t_vec 不进行拷贝，并且置 fec = NULL |
| `GridFunction(FiniteElementSpace *f)` | 构造与有限元空间 *f 相关的 GridFunction 对象，令长度为 *f 中未知量个数，置 fes = f ； fec = NULL ； fes_sequence = f->GetSequence() 并调用 `UseDevice(true)` |
| `GridFunction(FiniteElementSpace *f, double *data)` | 构造与有限元空间 *f 相关的，数据来自于 data 的 GridFunction 对象，要求 data 的长度不少于 *f 中未知量的个数，置 fes = f ； fec = NULL ； fes_sequence = f->GetSequence() 并调用 `UseDevice(true)` |
| `GridFunction(Mesh *m, std::istream &input)` |  |
| `GridFunction(Mesh *m, GridFunction *gf_array[], int num_pieces)` |  |
| `GridFunction &operator=(const GridFunction &rhs)` |  |
| `void MakeOwner(FiniteElementCollection *fec_)` |  |
| `FiniteElementCollection *OwnFEC()` |  |
| `int VectorDim() const` |  |
| `const Vector &GetTrueVector() const` |  |
| `Vector &GetTrueVector()` |  |
| `void GetTrueDofs(Vector &tv) const` |  |
| `void SetTrueVector()` |  |
| `virtual void SetFromTrueDofs(const Vector &tv)` |  |
| `void SetFromTrueVector()` |  |
| `void GetNodalValues(int i, Array<double> &nval, int vdim = 1) const` |  |
| `virtual double GetValue(int i, const IntegrationPoint &ip, int vdim = 1) const` |  |
| `virtual void GetVectorValue(int i, const IntegrationPoint &ip, Vector &val) const` |  |
| `void GetValues(int i, const IntegrationRule &ir, Vector &vals, int vdim = 1) const` |  |
| `void GetValues(int i, const IntegrationRule &ir, Vector &vals, DenseMatrix &tr, int vdim = 1) const` |  |
| `void GetVectorValues(int i, const IntegrationRule &ir, DenseMatrix &vals, DenseMatrix &tr) const` |  |
| `virtual double GetValue(ElementTransformation &T, const IntegrationPoint &ip, int comp = 0, Vector *tr = NULL) const` |  |
| `virtual void GetVectorValue(ElementTransformation &T, const IntegrationPoint &ip, Vector &val, Vector *tr = NULL) const` |  |
| `void GetValues(ElementTransformation &T, const IntegrationRule &ir, Vector &vals, int comp = 0, DenseMatrix *tr = NULL) const` |  |
| `void GetVectorValues(ElementTransformation &T, const IntegrationRule &ir, DenseMatrix &vals, DenseMatrix *tr = NULL) const` |  |
| `int GetFaceValues(int i, int side, const IntegrationRule &ir, Vector &vals, DenseMatrix &tr, int vdim = 1) const` |  |
| `int GetFaceVectorValues(int i, int side, const IntegrationRule &ir, DenseMatrix &vals, DenseMatrix &tr) const` |  |
| `void GetLaplacians(int i, const IntegrationRule &ir, Vector &laps, int vdim = 1) const` |  |
| `void GetLaplacians(int i, const IntegrationRule &ir, Vector &laps, DenseMatrix &tr, int vdim = 1) const` |  |
| `void GetHessians(int i, const IntegrationRule &ir, DenseMatrix &hess, int vdim = 1) const` |  |
| `void GetHessians(int i, const IntegrationRule &ir, DenseMatrix &hess, DenseMatrix &tr, int vdim = 1) const` |  |
| `void GetValuesFrom(const GridFunction &orig_func)` |  |
| `void GetBdrValuesFrom(const GridFunction &orig_func)` |  |
| `void GetVectorFieldValues(int i, const IntegrationRule &ir, DenseMatrix &vals, DenseMatrix &tr, int comp = 0) const` |  |
| `void ReorderByNodes()` |  |
| `void GetNodalValues(Vector &nval, int vdim = 1) const` |  |
| `void GetVectorFieldNodalValues(Vector &val, int comp) const` |  |
| `void ProjectVectorFieldOn(GridFunction &vec_field, int comp = 0)` |  |
| `void GetDerivative(int comp, int der_comp, GridFunction &der)` |  |
| `double GetDivergence(ElementTransformation &tr) const` |  |
| `void GetCurl(ElementTransformation &tr, Vector &curl) const` |  |
| `void GetGradient(ElementTransformation &tr, Vector &grad) const` |  |
| `void GetGradients(ElementTransformation &tr, const IntegrationRule &ir, DenseMatrix &grad) const` |  |
| `void GetGradients(const int elem, const IntegrationRule &ir, DenseMatrix &grad) const` |  |
| `void GetVectorGradient(ElementTransformation &tr, DenseMatrix &grad) const` |  |
| `void GetElementAverages(GridFunction &avgs) const` |  |
| `virtual void GetElementDofValues(int el, Vector &dof_vals) const` |  |
| `void ImposeBounds(int i, const Vector &weights, const Vector &lo_, const Vector &hi_)` |  |
| `void ImposeBounds(int i, const Vector &weights, double min_ = 0.0, double max_ = infinity())` |  |
| `void RestrictConforming()` |  |
| `void ProjectGridFunction(const GridFunction &src)` |  |
| `virtual void ProjectCoefficient(Coefficient &coeff)` |  |
| `void ProjectCoefficient(Coefficient &coeff, Array<int> &dofs, int vd = 0)` |  |
| `void ProjectCoefficient(VectorCoefficient &vcoeff)` |  |
| `void ProjectCoefficient(VectorCoefficient &vcoeff, Array<int> &dofs)` |  |
| `void ProjectCoefficient(VectorCoefficient &vcoeff, int attribute)` |  |
| `void ProjectCoefficient(Coefficient *coeff[])` |  |
| `virtual void ProjectDiscCoefficient(VectorCoefficient &coeff)` |  |
| `virtual void ProjectDiscCoefficient(Coefficient &coeff, AvgType type)` |  |
| `virtual void ProjectDiscCoefficient(VectorCoefficient &coeff, AvgType type)` |  |
| `void ProjectBdrCoefficient(Coefficient &coeff, Array<int> &attr)` |  |
| `virtual void ProjectBdrCoefficient(VectorCoefficient &vcoeff, Array<int> &attr)` |  |
| `virtual void ProjectBdrCoefficient(Coefficient *coeff[], Array<int> &attr)` |  |
| `void ProjectBdrCoefficientNormal(VectorCoefficient &vcoeff, Array<int> &bdr_attr)` |  |
| `virtual double ComputeL2Error(Coefficient &exsol, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeL2Error(Coefficient *exsol[], const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeL2Error(VectorCoefficient &exsol, const IntegrationRule *irs[] = NULL, Array<int> *elems = NULL) const` |  |
| `virtual double ComputeGradError(VectorCoefficient *exgrad, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeCurlError(VectorCoefficient *excurl, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeDivError(Coefficient *exdiv, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeDGFaceJumpError(Coefficient *exsol, Coefficient *ell_coeff, class JumpScaling jump_scaling, const IntegrationRule *irs[] = NULL) const` |  |
| `double ComputeDGFaceJumpError(Coefficient *exsol, Coefficient *ell_coeff, double Nu, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeH1Error(Coefficient *exsol, VectorCoefficient *exgrad, Coefficient *ell_coef, double Nu, int norm_type) const` |  |
| `virtual double ComputeH1Error(Coefficient *exsol, VectorCoefficient *exgrad, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeHDivError(VectorCoefficient *exsol, Coefficient *exdiv, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeHCurlError(VectorCoefficient *exsol, VectorCoefficient *excurl, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeMaxError(Coefficient &exsol, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeMaxError(Coefficient *exsol[], const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeMaxError(VectorCoefficient &exsol, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeL1Error(Coefficient &exsol, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeW11Error(Coefficient *exsol, VectorCoefficient *exgrad, int norm_type, Array<int> *elems = NULL, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeL1Error(VectorCoefficient &exsol, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual double ComputeLpError(const double p, Coefficient &exsol, Coefficient *weight = NULL, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual void ComputeElementLpErrors(const double p, Coefficient &exsol, Vector &error, Coefficient *weight = NULL, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeElementL1Errors(Coefficient &exsol, Vector &error, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeElementL2Errors(Coefficient &exsol, Vector &error, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeElementMaxErrors(Coefficient &exsol, Vector &error, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual double ComputeLpError(const double p, VectorCoefficient &exsol, Coefficient *weight = NULL, VectorCoefficient *v_weight = NULL, const IntegrationRule *irs[] = NULL) const` |  |
| `virtual void ComputeElementLpErrors(const double p, VectorCoefficient &exsol, Vector &error, Coefficient *weight = NULL, VectorCoefficient *v_weight = NULL, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeElementL1Errors(VectorCoefficient &exsol, Vector &error, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeElementL2Errors(VectorCoefficient &exsol, Vector &error, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeElementMaxErrors(VectorCoefficient &exsol, Vector &error, const IntegrationRule *irs[] = NULL ) const` |  |
| `virtual void ComputeFlux(BilinearFormIntegrator &blfi, GridFunction &flux, bool wcoef = true, int subdomain = -1)` |  |
| `GridFunction &operator=(double value)` |  |
| `GridFunction &operator=(const Vector &v)` |  |
| `virtual void Update()` |  |
| `FiniteElementSpace *FESpace()` |  |
| `const FiniteElementSpace *FESpace() const` |  |
| `virtual void SetSpace(FiniteElementSpace *f)` |  |
| `virtual void MakeRef(FiniteElementSpace *f, double *v)` |  |
| `virtual void MakeRef(FiniteElementSpace *f, Vector &v, int v_offset)` |  |
| `void MakeTRef(FiniteElementSpace *f, double *tv)` |  |
| `void MakeTRef(FiniteElementSpace *f, Vector &tv, int tv_offset)` |  |
| `virtual void Save(std::ostream &out) const` |  |
| `virtual void Save(const char *fname, int precision=16) const` |  |
| `void SaveVTK(std::ostream &out, const std::string &field_name, int ref)` |  |
| `void SaveSTL(std::ostream &out, int TimesToRefine = 1)` |  |
| `virtual ~GridFunction()` |  |
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