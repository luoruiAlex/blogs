## 作用域
- 局部变量在使用前一定要先初始化(可用任意类型相符的表达式)，全局变量不初始化则是0(只能用常量表达式初始化)
- C99后允许用任意常量表达式来初始化局部变量
##　结构体
- 定义和使用
```
struct pos { double x, y;};
struct pos { double x, y;} p1, p2;
struct pos p1, p2;
```
- 初始化
```
struct pos p1 = {3.0, 4.0};
struct pos p1 ＝｛3.0, y}; //y为表达式，可用于初始化局部变量
struct pos p1 = {3.0};
struct pos p1 = {0};
struct pos p1 = {.y=4.0};　//C99新特性
struct pos p1;p1 = {3.0, 4.0};// {}不是表达式，这里报错
```
