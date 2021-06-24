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

# restriction.hpp & restriction.cpp
## 类 ElementRestriction
- `class ElementRestriction : public Operator`
- 用于表示将有限元空间上的L-向量转变为E-向量的算子的类，其中L向量长度为总自由度个数，代表整体自由度，而E向量长度为每个有限元上自由度个数之和，代表局部自由度的拼接
### private 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `static const int MaxNbNbr = 16` |  |

### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const FiniteElementSpace &fes` | 限制算子对应的有限元空间 |
| `const int ne` | 单元的个数 |
| `const int vdim` | 向量维数，即每个自由度对应的未知量个数 |
| `const bool byvdim` | 代表未知量的排列顺序是否是 Ordering::byVDIM 的 |
| `const int ndofs` | 整体自由度的个数 |
| `const int dof` | 每个单元上自由度的个数（假设每个单元上自由度个数相同） |
| `const int nedofs` | 所有单元上局部自由度个数之和 |
| `Array<int> offsets, gatherMap` | offsets 和 gatherMap 分别是长为 ndofs+1 和 nedofs 的 int 型数组，用于表示自由度到局部自由度的映射：对编号为 i 的自由度，那么对于 offsets[i] <= k < offsets[i+1] 的 k ， gatherMap[k] 代表了这个自由度对应的某个局部自由度的编号 |
| `Array<int> indices` | 长为 nedofs 的 int 型数组，用于表示局部自由度到自由度的映射： indices[i] 表示编号为 i 的局部自由度对应的整体自由度的编号 |
| `Array<int> gatherMap` | 长为 nedofs 的int 型数组 |

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `ElementRestriction(const FiniteElementSpace &f, ElementDofOrdering e_ordering)` | 构造函数，令 fes 为 f ，每个单元中的自由度按照 e_ordering 的方式排列。方法中假定每个有限元是相同的。方法的主要工作是为数组 offsets ， indices 和 gatherMap 赋值 |
| `void Mult(const Vector &x, Vector &y) const` | 给定L-向量 x ，计算其对应的E-向量 y 。此方法是带符号的，即可以认为预先将向量 x 的符号根据 gatherMap 中指标的符号进行改变（实际上没有也不能更改 x ） |
| `void MultTranspose(const Vector &x, Vector &y) const` | 给定E-向量 x ，计算其对应的L-向量 y ，其中被多个单元所包含的自由度对应的 y 中的值取为各个局部自由度对应的 x 中的值之和。此方法是带符号的，即可以认为预先将向量 x 的符号根据 gatherMap 中指标的符号进行改变（实际上没有也不能更改 x ） |
| `void MultUnsigned(const Vector &x, Vector &y) const` | 方法`Mult()`的无符号版本 |
| `void MultTransposeUnsigned(const Vector &x, Vector &y) const` | 方法`MultTranspose()`的无符号版本 |
| `void BooleanMask(Vector& y) const` |  |
| `void FillSparseMatrix(const Vector &mat_ea, SparseMatrix &mat) const` |  |
| `int FillI(SparseMatrix &mat) const` |  |
| `void FillJAndData(const Vector &ea_data, SparseMatrix &mat) const` |  |

## 类 L2ElementRestriction
- `class L2ElementRestriction : public Operator`
- 用于表示将 L2 （间断）有限元空间上的L-向量转变为E-向量的算子的类

### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const int ne` | 单元的个数 |
| `const int vdim` | 向量维数，即每个自由度对应的未知量个数 |
| `const bool byvdim` | 代表未知量的排列顺序是否是 Ordering::byVDIM 的 |
| `const int ndof` | 每个单元上自由度的个数（假设每个单元上自由度个数相同） |
| `const int ndofs` | 整体自由度的个数 |

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `L2ElementRestriction(const FiniteElementSpace& fes)` | 构造函数，根据有限元空间 fes 为成员变量赋值，并为 height 和 weight 赋值 |
| `void Mult(const Vector &x, Vector &y) const` | 给定L-向量 x ，计算其对应的E-向量 y |
| `void MultTranspose(const Vector &x, Vector &y) const` | 给定E-向量 x ，计算其对应的L-向量 y |
| `void FillI(SparseMatrix &mat) const` |  |
| `void FillJAndData(const Vector &ea_data, SparseMatrix &mat) const` |  |


## 类 H1FaceRestriction
- `class H1FaceRestriction : public Operator`

### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const FiniteElementSpace &fes` | 限制算子对应的有限元空间 |
| `const int nf` |  |
| `const int vdim` | 向量维数，即每个自由度对应的未知量个数 |
| `const bool byvdim` | 代表未知量的排列顺序是否是 Ordering::byVDIM 的 |
| `const int ndofs` | 整体自由度的个数 |
| `const int dof` | 每个单元上自由度的个数（假设每个单元上自由度个数相同） |
| `const int nfdofs` |  |
| `Array<int> offsets, gather_indices` | offsets 和 gatherMap 分别是长为 ndofs+1 和 nedofs 的 int 型数组，用于表示自由度到局部自由度的映射：对编号为 i 的自由度，那么对于 offsets[i] <= k < offsets[i+1] 的 k ， gatherMap[k] 代表了这个自由度对应的某个局部自由度的编号 |
| `Array<int> scatter_indices` | 长为 nedofs 的 int 型数组，用于表示局部自由度到自由度的映射： indices[i] 表示编号为 i 的局部自由度对应的整体自由度的编号 |

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `H1FaceRestriction(const FiniteElementSpace&, const ElementDofOrdering, const FaceType)` |  |
| `void Mult(const Vector &x, Vector &y) const` |  |
| `void MultTranspose(const Vector &x, Vector &y) const` |  |

## 类 L2FaceRestriction
- `class L2FaceRestriction : public Operator`

### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `const FiniteElementSpace &fes` | 限制算子对应的有限元空间 |
| `const int nf` |  |
| `const int ne` |  |
| `const int vdim` | 向量维数，即每个自由度对应的未知量个数 |
| `const bool byvdim` | 代表未知量的排列顺序是否是 Ordering::byVDIM 的 |
| `const int ndofs` | 整体自由度的个数 |
| `const int dof` | 每个单元上自由度的个数（假设每个单元上自由度个数相同） |
| `const L2FaceValues m` |  |
| `const int elemDofs` |  |
| `const int nfdofs` |  |
| `Array<int> offsets, gather_indices` | offsets 和 gatherMap 分别是长为 ndofs+1 和 nedofs 的 int 型数组，用于表示自由度到局部自由度的映射：对编号为 i 的自由度，那么对于 offsets[i] <= k < offsets[i+1] 的 k ， gatherMap[k] 代表了这个自由度对应的某个局部自由度的编号 |
| `Array<int> scatter_indices1` | 长为 nedofs 的 int 型数组，用于表示局部自由度到自由度的映射： indices[i] 表示编号为 i 的局部自由度对应的整体自由度的编号 |
| `Array<int> scatter_indices2` | 长为 nedofs 的 int 型数组，用于表示局部自由度到自由度的映射： indices[i] 表示编号为 i 的局部自由度对应的整体自由度的编号 |

### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `L2FaceRestriction(const FiniteElementSpace&, const ElementDofOrdering, const FaceType, const L2FaceValues m = L2FaceValues::DoubleValued)` |  |
| `virtual void Mult(const Vector &x, Vector &y) const` |  |
| `void MultTranspose(const Vector &x, Vector &y) const` |  |
| `virtual void FillI(SparseMatrix &mat, SparseMatrix &face_mat) const` |  |
| `virtual void FillJAndData(const Vector &ea_data, SparseMatrix &mat, SparseMatrix &face_mat) const` |  |
| `void AddFaceMatricesToElementMatrices(Vector &fea_data, Vector &ea_data) const` |  |