## 执行
- `#!/bin/bash` `chmod +x hello.sh` `./hello.sh`
- `/bin/sh hello.sh`

## 变量
- 只读变量 `readonly var` `var=xxx`
### 类型
- 局部变量：脚本或命令中定义，仅在当前shell中有效
- 环境变量：所有程序都可以访问环境变量
- Shell变量：由shell程序设置的特殊变量，部分为局部变量，部分为环境变量
### 定义与使用
- 定义时不加`$`
- 使用 `$var` `${var}`
- 删除 `unset var`
### 字符串
  - 单引号中不能有变量和单引号(转义也不行)，双引号可以有变量和转义字符
  - 拼接 `hello, ${string}`
  - 长度 `${#string}`
  - 子串 `${string:1:4}`
  - index `expr index "$string" is`
### 数组
- 定义：`arr=(v1, v2)`或`arr[index]=v`
- 使用:
  - `${arr[index]}`
  - `${#arr[@]}`或`$(#arr[*]}`获取长度
  - @ *可获取数组中的所有元素
    
## 向shell传参
- `./hello.sh 1 2 3`
- `$n`表示第几个参数，`$0`表示执行的文件名hello.sh
- `$*`表示所有参数"1 2 3"，`$@`表示"1" "2" "3"
- `$#`表示参数个数3
- `$$`表示脚本运行的当前进程ID号
- `$!`表示后台运行的最后一个进程的ID号
- `$?`表示最后命令的退出状态，0表示没有错误，其他表示有
- `$-`表示shell使用的当前选项，与set命令功能相同

## 运算符
- var=\`expr 1 + 2\` 注意空格不能省
- 算数运算符`+ - * / % == != ` 注意乘号`*`前必须加反斜杠`\`
- 关系运算符只支持数字或纯数字字符串 `-eq -ne -gt -lt -ge -le` `if test $n1 -eq $n2`
- 布尔运算符 `! -o -a`
- 逻辑预算符 `&& ||`
- 字符串运算符 `= != -z(长度为0) -n(长度不为0) str(字符变量名本身，表示是否为空)`
- 文件测试运算符
  - `[-d $file]`是否为目录
  - `[-f $file]`是否为普通文件 `-b -c`
  - `-e` 文件是否存在
  - `-r -w -x` 是否可读、写、执行
  - `-s` 是否为空(文件大小为0)
  - `-g -k -p -u`

## 流程控制
- if 
  - 一行 `if [ $(ps -ef | grep -c "ssh") -gt 1]; then echo "true"; fi`
  - 多行
  ```
  if condition
  then
    command1
    command2
    ...
  fi
  ```
- for
  - 一行 `for var in item1 item2 ...; do command1; command2.... done;`
  - 多行
  ```
    for var  in item1 item2 ...
    do
      command1
      command2
      ...
    done
  ```
- while
```
while condition
do
  command
  ...
done 
```
- until
```
until condition
do
  command
done
```
- case
```
case 值 in
模式1)
  commands
  ;;
模式2)
  commands
  ;;
esac
```
- break continue可用于所有循环

## 函数
- 定义
```
[function] funName[()]
{
  ...
  [return xx]
}
```
- 调用 `funName arg1 arg2`
- 内部使用`$n`获取参数，外部使用`$?`获取返回值

## 重定向
- `command>file`输出重定向到文件 `command>>file`以追加方式
- `n>file`文件描述符为n的文件重定向到file，文件描述符：0为STDIN，1为STDOUT，2为STDERR
- `n>&m`将输出文件m和n合并
- `<<tag`将开始标记tag和结束标记tag之间的内容作为输入
- 执行命令但不显示输出结果 `command > /dev/null`
- 屏蔽stdout stderr，`command > /dev/null 2>&1` 2>&1表示将stderr重定向到文件描述符为1的文件`/dev/stdout`中

## 文件包含
- `. filename`或者`source filename`
- 被包含的文件不需要可执行权限

## let与expr
- shell中默认是字符串操作，let和expr可执行整数的数学运算
- `let var+=1`运算符间不能有空格
- var=\`expr $var + 1\`保证参数和运算符中间有空格
- 支持浮点预算用bc或者awk
