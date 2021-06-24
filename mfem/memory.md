# mem_manager.hpp & mem_manager.cpp
## 类Memory
- `template <typename T> class Memory`：模板类声明
- `enum class MemoryType`：记录MFEM所支持的内存类型
  - { HOST, HOST_32, HOST_64, HOST_DEBUG, HOST_UMPIRE, MANAGED, DEVICE, DEVICE_DEBUG, DEVICE_UMPIRE }
- `enum class MemoryClass`：记录内存类型的分类
  - HOST: { HOST, HOST_32, HOST_64, HOST_DEBUG, HOST_UMPIRE, MANAGED }
  - HOST_32: { HOST_32, HOST_64, HOST_DEBUG }
  - HOST_64: { HOST_64, HOST_DEBUG }
  - DEVICE: { DEVICE, DEVICE_DEBUG, DEVICE_UMPIRE, MANAGED }
  - MANAGED: { MANAGED }
### 主要成员变量
- `enum FlagMask: unsigned`：记录Memory对象内存状态
  - { REGISTERED, OWNS_HOST, OWNS_DEVICE, OWNS_INTERNAL, VALID_HOST, VALID_DEVICE, USE_DEVICE, ALIAS}
  - 一个unsigned类型的flags可以记录多个状态
- `T *h_ptr`：指向host内存的指针
- `int capacity`：分配的内存的大小
- `MemoryType h_mt`：host内存类型
- `mutable unsigned flags`：记录内存状态的标志
### 主要成员函数
#### 构造函数和析构函数
- `explicit Memory(int size)`：分配长度为size的`MemoryManager::host_mem_type`型的内存
- `Memory(int size, MemoryType mt)`：分配长度为size的mt型的内存
- `explicit Memory(T *ptr, int size, bool own)`：将外部指针ptr包装，内存类型为`MemoryManager::host_mem_type`
- `Memory(T *ptr, int size, MemoryType mt, bool own)`：将将外部指针ptr包装，内存类型为mt
- `Memory(const Memory &base, int offset, int size)`：构造一个指向base对象内部的对象（可以用于构造子内存，也就是说构造一个指针指向base对象h_ptr所占用内存中一小段的对象，offset为内存的偏移量）
- 无参数的构造函数、拷贝构造函数、移动构造函数、拷贝赋值算子、移动赋值算子以及析构函数是**默认**的，注意到默认的析构函数并不会释放当前的内存
#### 对内存标志flags的读写操作
- `bool OwnsHostPtr() const`
- `void SetHostPtrOwner(bool own) const`
- `bool OwnsDevicePtr()`
- `void SetDevicePtrOwner(bool own) const`
- `void ClearOwnerFlags() const`
- `bool UseDevice() const`
- `void UseDevice(bool use_dev) const`
#### 指针的新建、包装和释放
- `void Reset()`：重置函数，将h_ptr重置为空，capacity和flags重置为0，h_mt取为`MemoryManager::host_mem_type`
- `void Reset(MemoryType host_mt)`：重置函数，将h_ptr重置为空，capacity和flags重置为0，h_mt取为host_mt
- `bool Empty() const`：检查h_prt是否为空指针
- `void New(int size)`：分配尺寸为size的，类型为`MemoryManager::host_mem_type`的host指针
- `void New(int size, MemoryType mt)`：分配尺寸为size的，类型为mt（mt为HOST类内存）或mt的对偶型内存（见`MemoryManager::GetDualMemoryType_(mt)`）（mt不为HOST型内存）的host指针
- `void Wrap(T *ptr, int size, bool own)`：将一个外部的T类型的指针ptr包装为h_ptr，类型为`MemoryManager::host_mem_type`，own决定这个外部指针是否被Memory对象所拥有。只有当own为真且h_mt不为HOST时，此外部指针才进行注册（见`MemoryManager::Register_`）
- `void Wrap(T *ptr, int size, MemoryType mt, bool own)`：将一个外部的T类型的指针ptr包装为h_ptr，类型为mt（mt为HOST类内存）或mt的对偶型内存，ptr必须是mt型的。own决定这个外部指针是否被Memory对象所拥有。当own为假或者mt为HOST时，将跳过注册阶段
- `void Wrap(T *h_ptr, T *d_ptr, int size, MemoryType h_mt, bool own)`：包装一对外部的T类型的指针ptr和d_ptr，host指针类型为mt，ptr和d_ptr必须是mt型和其对偶型的。own决定这对指针是否被Memory对象所拥有。
- `void MakeAlias(const Memory &base, int offset, int size)`：构造一个指向base对象内部的对象，对base对象的h_ptr进行了offset的偏移，并且标志flags进行相应的改变
- `void Delete()`：释放被对象拥有的指针，但是对象并不会被重置（reset）
#### 对象成员变量的获取
- `T &operator[](int idx)`：返回h_ptr[idx]
- `const T &operator[](int idx) const`：上条的const形式
- `operator T*()`：返回h_ptr
- `operator const T*() const`：上条的const形式
- `template <typename U> explicit operator U*()`：进行强制类型转换，返回一个U型的h_ptr
- `template <typename U> explicit operator const U*() const`：上式的const形式
- `int Capacity() const`：获得对象的长度
- `MemoryType GetMemoryType() const;`：获得对象的内存类型，当device指针存在时，返回device内存类型，否则返回h_mt（即host内存类型）
#### 4321
- `T *ReadWrite(MemoryClass mc, int size)`
- `const T *Read(MemoryClass mc, int size) const`
- `T *Write(MemoryClass mc, int size)`
- `void Sync(const Memory &other) const`
- `void SyncAlias(const Memory &base, int alias_size) const`
- `MemoryType GetMemoryType() const`
- `void CopyFrom(const Memory &src, int size)`：从src对象中拷贝size项数据到this中
- `void CopyFromHost(const T *src, int size)`：从T型的host指针src中拷贝size项数据到this中
- `void CopyTo(Memory &dest, int size) const`：从this中拷贝size项数据到对象dest中
- `void CopyToHost(T *dest, int size) const`：从this中拷贝size项数据到T型的host指针dest中
- `void PrintFlags() const`：打印对象的状态（由flags所记录的）
- `int CompareHostAndDevice(int size) const`：如果host和device数据都存在时，对比它们的前size项数据