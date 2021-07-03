# coefficient.hpp & coefficient.cpp
## （基）类 Coefficient
- `class Coefficient`
### protected 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `double time` | 时间 |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `Coefficient()` | 默认构造函数，令 time = 0 |
| `void SetTime(double t)` | 设置 time = t |
| `double GetTime()` | 返回 time |
| `virtual double Eval(ElementTransformation &T, const IntegrationPoint &ip) = 0` | （纯虚函数）获得系数函数在以 T 刻画的单元上位于积分点 ip 上的值（方法需要保证 T 中的积分点与 ip 一致） |
| `double Eval(ElementTransformation &T, const IntegrationPoint &ip, double t)` | 获得在时刻 t 时系数函数在以 T 刻画的单元上位于积分点 ip 上的值（方法需要保证 T 中的积分点与 ip 一致），方法中将设置 time = t |
| `virtual ~Coefficient()` | 空析构函数 |

## 类 ConstantCoefficient
- `class ConstantCoefficient : public Coefficient`
### public 成员变量
| 变量 | 解释 |
| ---- | ---- |
| `double constant` | 代表系数函数的常数 |
### public 成员函数
| 方法 | 解释 |
| ---- | ---- |
| `explicit ConstantCoefficient(double c = 1.0)` | 构造函数，令 constant = c |
| `virtual double Eval(ElementTransformation &T, const IntegrationPoint &ip)` | （虚函数实例）返回 constant |
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