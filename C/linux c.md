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
- struct的成员名和变量名不在同一个命名空间中
- 可使用数据类型标志使得struct中的同一个成员在不同类型下表示不同值
```
struct pos{
  enum coordinate_type t;
  double a, b;
}
```
- 嵌套struct
```
struct s1 { struct s2 start; struct s2 end;};
struct s1 a = {.start.x=1.0,.end.y=2.0};
struct s1 b = {{..},{...}};
```


## 枚举
- 定义
```
enum coordinate_type {RECTANGULAR = 1, POLAR};
```
- 枚举的成员名和变量名在同一个命名空间中，枚举变量的命名空间中不能出现和成员名相同的变量
