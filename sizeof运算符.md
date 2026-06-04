------------------------------
## 一、 sizeof 的底层
1. 以下代码，编译器在编译代码时计算A类如果在内存中实例化，需要占用多少个字节，然后把 sizeof(A)替换为纯数字字面量
2. A类中成员变量a为静态，属于类，不计入对象内存大小中，因此A类实例化后是空类
3. 由于类默认权限是private，所以A类在运行时无法实例化，因为无法在类外调用私有构造函数。

```
#include <iostream>
using namespace std;
class A {
    A() {}
    ~A() {}
    static int a;
};
int main() {
    cout << sizeof(A) << endl; // 输出1
    return 0;
}
```
