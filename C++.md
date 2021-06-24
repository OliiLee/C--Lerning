# 关键词
## const
- 修饰变量类型：`const type name = value`，用于定义一个常量
- 修饰指针
  - `int age = 16; const int* pt = &age;`：意味着pt指向const int（这里是16），也就是说*pt的值不能（通过\*pt）进行修改，但是pt的值是可以更改的
    - 指向const的指针既可以指向non-const变量又可以指向const变量，强调指针本身不能对变量进行修改；然而指向non-const的指针不能指向const变量
    - 指向const的指针可以使用指向non-const的指针赋值，这只在仅有一层间接性时是合法的，如`int age = 16; int* pt = &age; const int* pd = pt;`。在两层间接性时，混用const和non-const是不安全的，用一个指向const的双指针来指向一个指向non-const的指针是不合法的，否则可能利用这个关系修改一个const变量的值（如果合法，代码`const int** pp; int* p; const int n = 0;pp = &p;*pp = &n;*p = 1`实际上修改了n的值）
  - `int age = 16; int* const pt = &age;`：意味着pt本身是const的，也就是说pt的值无法更改而*pt的值可以更改
  - 当指针作为函数形参，尽可能得使用const修饰它，只要函数内不需要对指针指向的实参的内容进行修改，不管实参是否是const的，此方法均是合法的。这个方法在只有一层不直接性时是有效的，在有两层不直接性时，不要这样操作
- 修饰引用
  - `int age = 16; const int& rf = age;`：意味着不能通过rf对age的值进行更改
  - 在引用作为函数的形参时，通过在前面加const关键字，强调函数内部不能对实参进行修改
- 在类中
  - const成员函数：如果声明了一个const对象`const className obj`，那么对一个非const的成员函数`void className::foo()`，`obj.foo()`不合法，因为编译器无法判断`foo()`中是否对成员变量进行修改。从而需要声明`foo()`为const的，即`void className::foo() const`，这可以理解为`foo()`中隐含的形参this用const修饰
## constexpr
- C++11提出，用于表示常量，强调“不可修改”属性；用于修饰返回值非空的函数，使编译器在编译阶段计算出函数的返回值
- constexpr不能直接修饰对于用户自定义数据类型（如类），需要在类内用constexpr修饰构造函数从而实现此功能
## enum
- `enum ENUM` 创建名为ENUM的枚举类型，用于刻画某一个变量的可能取值有限的情况
- `enum class ENUM` 为了解决`enum`存在的潜在的作用域污染的问题，C++11提出了枚举类的概念`enum class`，两个不同的枚举类可以包含相同的关键字，但是需要使用`枚举类名称::关键字`对枚举类对象进行赋值
## explicit
- 修饰构造函数，阻止不应该允许的经过转换构造函数进行的隐式转换的发生。C++中， 一个参数的构造函数(或者除了第一个参数外其余参数都有默认值的多参构造函数)， 承担了两个角色：*构造*；*默认且隐含的类型转换操作符*。
## extern
- 与"C"一起连用时，如: extern "C" void fun(int a, int b);则告诉编译器在编译fun这个函数名时按着C的规则去翻译相应的函数名而不是C++的
- 当extern不与"C"在一起修饰变量或函数时，它的作用就是声明函数或全局变量的作用范围的关键字，其声明的函数和变量可以在本模块或其他模块中使用。它是一个声明不是定义!也就是说B模块(编译单元)要是引用模块(编译单元)A中定义的全局变量或函数时，它只要包含A模块的头文件即可。
## friend
- 友元函数，使一个类外的函数可以访问private和protected的成员
- 友元类，在类A内声明类B为其友元，那么类B的所有成员函数就可以访问类A的private和protected成员
- 友元成员函数，在类A内声明类B的一个成员函数为其友元成员函数，那么此成员函数可以访问类A的private和protected成员
## mutable
- 被mutable修饰的变量，将永远处于可变的状态。即使在被const函数所调用。
    >典型应用场景：在测试某方法的调用个数时，定义const函数`int getAge() const;`函数体中对变量`m_nums`进行+1，此时需要声明`m_nums`为mutable
- mutable不能修饰const 和 static 类型的变量。
## static
- 定义静态全局变量，只能在本文件中可以访存（与extern区别）
- 在类中
  - 静态数据成员（修饰变量），被所有类的实例所共享的变量，是类而不是某个对象所享有的，可以通过类名调用，需要在类声明外初始化
  - 静态成员函数（修饰函数），只能访问静态成员，从而同样是类而不是某个对象所享有的，可以通过类名调用
## typedef
- 类型的别名，`typedef typeName aliasName`，`typeName`可以使一个变量类型，指针类型甚至是一个表达式（`decltype(x)`）
# 方法

# 杂项
## C++中的POD数据格式（C++11）
- 通俗地讲，一个类、结构、共用体对象或非构造类型对象能通过二进制拷贝（如 memcpy()）后还能保持其数据不变正常使用的就是POD类型的对象。严格来讲，一个对象既是普通类型（Trivial Type）又是标准布局类型（Standard-layout Type）那么这个对象就是 POD 类型
## 左值和右值
C++中左值（lvalue）
