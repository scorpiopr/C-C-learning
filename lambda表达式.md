参考https://blog.csdn.net/m0_60134435/article/details/136151698
# 基本语法
```
[capture list] (parameter list) -> return type { function body }
```
* capture list：外部变量捕获列表
* parameter list：参数列表
* return type：返回值类型
* function body：函数体
# 捕获方式
* 值捕获
  * 值捕获的变量的值在lambda表达式定义时确定
  * 不能在函数体中更改，除非使用mutable关键字
```
int x = 10;
auto f = [x] (int y) -> int { return x + y; }; // 值捕获 x
x = 20; // 修改外部的 x
cout << f(5) << endl; // 输出 15，不受外部 x 的影响
```  
* 引用捕获
  * 引用捕获的变量的值在lambda表达式被调用时确定
  * 可在函数体中更改
```
int x = 10;
auto f = [&x] (int y) -> int { return x + y; }; // 引用捕获 x
x = 20; // 修改外部的 x
cout << f(5) << endl; // 输出 25，受外部 x 的影响
```
* 隐式捕获
  * 在捕获列表中使用=或&，省略变量名，表示按值或引用捕获lambda表达式中使用的所有外部变量
  * 不能和同类型的显示捕获同时使用
```
int x = 10;
int y = 20;
auto f = [=, &y] (int z) -> int { return x + y + z; }; // 隐式按值捕获 x，显式按引用捕获 y
x = 30; // 修改外部的 x
y = 40; // 修改外部的 y
cout << f(5) << endl; // 输出 55，不受外部 x 的影响，受外部 y 的影响
```
* 初始化捕获
  * C++14 引入，允许在捕获列表中使用初始化表达式创建并初始化新变量
  * 值捕获的变量的值在lambda表达式定义时确定
```
int x = 10;
auto f = [z = x + 5] (int y) -> int { return z + y; }; // 初始化捕获 z，相当于值捕获 x + 5
x = 20; // 修改外部的 x
cout << f(5) << endl; // 输出 20，不受外部 x 的影响
```
# lambda表达式和类的关系
本质是函数对象，是重载了\(\)运算符的匿名类的对象
```
int x = 10;
auto f = [x] (int y) -> int { return x + y; };
```
等价于
```
int x = 10;
class __lambda_1
{
public:
    __lambda_1(int x) : __x(x) {} // 构造函数，用于初始化捕获的变量
    int operator() (int y) const // 重载的 operator()，用于调用 Lambda表达式
    {
        return __x + y; // 函数体，与 Lambda表达式的函数体相同
    }
private:
    int __x; // 数据成员，用于存储捕获的变量
};
auto f = __lambda_1(x); // 创建一个匿名类的对象，相当于 Lambda表达式
```
