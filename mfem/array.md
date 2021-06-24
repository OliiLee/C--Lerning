# array.hpp & array.cpp
## 类Array
- `template <class T> class Array`：模板类定义
- 抽象数据结构数组，包含类型为T的元素并且可以自动增加长度，T必须是一个POD数据类型。分配的内存长度可能比数组有效数据长度更大，数据可以用符号[]读取
### 成员变量
- `Memory<T> data`：指向数据的指针
- `int size`：数组的长度
### 成员函数
#### 构造函数和析构函数
- `Array()`：创建一个空数组
- `Array(MemoryType mt)`：创建一个空的数组，内存类型为mt
- `Array(int asize)`：创建一个长度为asize的数组
- `Array(T *_data, int asize)`：用一个C-数组来创建数组，原本的C-数组将不会被释放
- `Array(const Array &src)`：拷贝构造函数，此方法支持任意内存类型的src（深度拷贝）
- `template <typename CT> Array(const Array<CT> &src)`：拷贝构造函数（深度拷贝），CT是一个可以转变类型为T的类型，并不支持多样的内存类型
- `~Array()`：释放分配的内存
#### 对数组数据的读取、赋值、拷贝操作
- `Array<T> &operator=(const Array<T> &src)`：赋值函数，从src的深度拷贝，src可以为任意的内存类型
- `template <typename CT> Array &operator=(const Array<CT> &src)`：赋值函数，从src的深度拷贝，CT是一个可以转变类型为T的类型，并不支持多样的内存类型
- `void operator=(const T &a)`：将数组元素赋值为a（data和size需要提前指定，此方法只进行赋值）
- `operator T *()`：返回T*型的data，调用格式为`(T*)obj`
- `operator const T *() const`：上条的const形式
- `T & operator[](int i)`：对数组第i个元素的引用
- `const T &operator[](int i) const`：上条的const形式
- `void Copy(Array &copy) const`：建立数组的副本copy
- `void MakeRef(T *p, int s)`：使数组为长为s的C数组p的引用（此数组将不拥有此指针）
- `void MakeRef(const Array &master)`：使数组为const数组master的引用（此数组将不拥有master的内存）
- `void Assign(const T *p)`：将const指针p的前size项元素拷贝到data中（data和size需要提前指定，此方法只复制数据）
#### 对Memory对象data的读取和操作
- `T *GetData()`：返回T*型的data
- `const T *GetData() const`：上条的const形式
- `Memory<T> &GetMemory()`：返回Memory对象data的引用
- `Memory<T> &GetMemory() const`：上条的const形式
- `bool UseDevice() const`：判断data是否使用了Device
- `bool OwnsData() const`：判断data是否被其拥有（在数组不存在后其内存是否会被释放）
- `void StealData(T **p)`：将data转移到p中（不进行数据的转移），随后重置data
- `void LoseData()`：重置data
- `MakeDataOwner() const`：使数组拥有data
#### 对数组长度、占用内存长度的读取和操作
- `int Size() const`：返回数组的长度（而不是内存长度）
- `void SetSize(int nsize)`：将数组的长度更改为nsize，但不更改数据，如果nsize大于内存长度，那么将重新分配内存并且将原始数据拷贝过去
- `void SetSize(int nsize, const T &initval)`：将数组的长度更改为nsize，如果长度增加，那么增加的数据设置为initval
- `void SetSize(int nsize, MemoryType mt)`：将数组的长度更改为nsize，内存类型更改为mt，只有当内存类型不改变且不需要重新内存时，数据将保留
- `int Capacity() const`：返回数组占有的内存长度
- `void Reserve(int capacity)`：保证数组占有的内存长度至少为capacity，数据将被保存
- `long MemoryUsage() const`：返回分配的内存大小（以字节显示）
#### 数组的拼接和删除
- `int Append(const T & el)`：在数组尾部增加一项el，返回增加后数组的长度
- `int Append(const T *els, int nels)`：在数组尾部增加els的前nels项，返回增加后数组的长度
- `int Append(const Array<T> &els)`：在数组尾部增加另一个数组els，返回增加后数组的长度
- `int Prepend(const T &el)`：在数组头部增加一项el，返回增加后数组的长度
- `T &Last()`：以T&形式返回数组最后一个元素
- `const T &Last() const`：上条的const形式
- `void DeleteLast()`：删除掉数组最后一个元素（数组占用内存长度不变）
- `void DeleteFirst(const T &el)`：删除数组第一个等于el的项（数组占用内存长度不变）
- `void DeleteAll()`：释放data并将其重置
#### 打印、保存和加载
- `void Print(std::ostream &out = mfem::out, int width = 4) const`：打印数组所有元素到流out中，显示宽度为width（即一行最多显示width个元素）
- `void Save(std::ostream &out, int fmt = 0) const`：保存这个数组到流out中，相较于打印，此方法额外保存了size的大小
- `void Load(std::istream &in, int fmt = 0)`：从流in中保存读取数据，fmt=0时先读取size再读取数据，fmt=1时读取size个数据（size需要提前定义）
- `void Load(int new_size, std::istream &in) { SetSize(new_size); Load(in, 1); }`：从流in中读取new_size个数据
#### 其他的实用方法（排序、取大小、查询操作等）
- `int Union(const T & el)`：当el不是数组的元素时，将其增加在数组尾部，返回元素el在数组中的索引
- `void Sort()`：以升序对数组进行排序，需要用到数据类型T的&#60;关系
- `template<class Compare> void Sort(Compare cmp)`：使用比较函数对象cmp对数组进行升序排序
- `int IsSorted()`：如果数组是按照升序排列的，返回1，否则返回0
- `int Find(const T &el) const`：寻找元素el在数组中的指标，如果数组中无此元素，那么将返回-1
- `int FindSorted(const T &el) const`：在一个有序数组中使用二分算法寻找el的索引，如果没有找到将返回-1
- `T Max() const`：得到数组元素的最大值，用到数据类型T的&#60;关系
- `T Min() const`：得到数组元素的最小值，用到数据类型T的&#60;关系
- `void Unique()`：排除掉数组中重复的元素，不改变数组占用内存的长度
- `void PartialSum()`：将数组元素更新为数组的部分和，即${\rm new\_array[i]=\sum_{j=0}^i array[j]}$
- `T Sum()`：得到数组元素之和
- `void GetSubArray(int offset, int sa_size, Array<T> &sa) const`：获得子数组sa，其长度为sa_size，摘取`\*this)[offset]`到`(*this)[offset+sasize-1]`的元素



#### 对Memory对象data的读写操作（见mem_manager.hpp和mem_manager.cpp）
- `const T *Read(bool on_dev = true) const`：
- `const T *HostRead() const`
- `T *Write(bool on_dev = true)`
- `T *HostWrite()`
- `T *ReadWrite(bool on_dev = true)`
- `T *HostReadWrite()`

#### 类STL方法
- `template <typename U> void CopyTo(U *dest)`：将数组内容从头至尾拷贝到dest中
- `template <typename U> void CopyFrom(const U *src)`：从src中拷贝尽可能多的元素（不超过分配的内存长度）到数组中，此方法不更新size的值
- `T* begin()`：返回指向数组第一个元素的指针
- `const T* begin() const`：上条的const形式
- `T* end()`：返回指向数组最后一个元素后面的指针
- `const T* end() const`：上条的const形式

---
## 类Array2D
- `template <class T> class Array2D`：模板类定义
### 成员变量
- `Array<T> array1d`：内部数据是以一维数组的形式定义的（按行）
- `int M, N`：M为列数而N为行数
### 成员函数
#### 构造函数和析构函数
- `Array2D()`：创建一个空的二维数组，即M=N=0
- `Array2D(int m, int n)`：创建一个行数为m列数为n的二维数组，即M=m，N=n，array1d长度为m*n
#### 数组行、列数和内存长度的读取和操作
- `void SetSize(int m, int n)`：将二维数组的行数和列数分别设置为m和n，array1d长度相应进行改变
- `int NumRows() const`：返回二维数组的行数M
- `int NumCols() const`：返回二维数组的列数N
#### 对数组数据的读取、赋值、拷贝操作
- `T &operator()(int i, int j)`：返回二维数组第i行第j列元素的引用
- `const T &operator()(int i, int j) const`：上条的const形式
- `T *operator[](int i)`：返回指向二维数组第i行第一个元素的指针，相当于返回第i行（以C数组形式）
- `const T *operator[](int i) const`：上条的const形式
- `T *operator()(int i)`：等同于[]
- `const T *operator()(int i) const`：上条的const形式
- `T *GetRow(int i)`：等同于[]
- `const T *GetRow(int i) const`：上条的const形式
- `void GetRow(int i, Array<T> &sa) const`：将二维数组的第i行拷贝到数组sa中
- `void Copy(Array2D &copy) const`：创建二维数组的一个拷贝copy
- `void operator=(const T &a)`：将二维数组的每一个元素都赋值为a
- `Array2D& operator=(const Array2D &a)`：默认赋值函数
- `void MakeRef(const Array2D &master)`：使此二维数组是二维数组master的一个引用
- `void DeleteAll()`：重置此二维数组，行数和列数置为0，array1d置为空
#### 数组的打印、保存和加载
- `void Save(std::ostream &out, int fmt = 0) const`：将二维数组保存到流out中，如果fmt=0则保存行列，而fmt=1则只保存元素，元素将以一维数组的形式保存（按行存储）
- `void Load(std::istream &in, int fmt = 0)`：从流in中加载二维数组，如果fmt=0则先依次加载行数和列数再加载数据，否则直接加载数据（二维数组的行数和列数需要预先指定）
- `void Load(const char *filename, int fmt = 0)`：从文件filename中加载二维数组，如果fmt=0则先依次加载行数和列数再加载数据，否则直接加载数据（二维数组的行数和列数需要预先指定）
- `void Load(int new_size0,int new_size1, std::istream &in)`：设置二维数组的行数和列数分别为new_size0和new_size1，并从流in中读取相应多的元素
- `Print(std::ostream &out = mfem::out, int width = 4)`：以宽度width（即一行中最多打印四个元素）打印此二维数组到流out中

---

## 类Array3D
- `template <class T> class Array3D`
### 成员变量
- `Array<T> array1d`
- `int N2, N3`
### 成员函数
- `Array3D()`
- `Array3D(int n1, int n2, int n3)`
- `void SetSize(int n1, int n2, int n3)`
- `T &operator()(int i, int j, int k)`
- `const T &operator()(int i, int j, int k) const`

---

## 类BlockArray
- `template<typename T> class BlockArray`
### 成员类
#### 类iterator_base(protected)
- `template <typename cA, typename cT> class iterator_base`
#### 类iterator
- `class iterator : public iterator_base<BlockArray, T>`
#### 类const_iterator
- `class const_iterator : public iterator_base<const BlockArray, const T>`
### 成员变量

### 成员函数

---

## 类外函数
- `template <class T> bool operator==(const Array<T> &LHS, const Array<T> &RHS)`
- `template <class T> bool operator!=(const Array<T> &LHS, const Array<T> &RHS)`
- `void Swap(Array<T> &, Array<T> &)`