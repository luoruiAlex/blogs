## 数据交换
- 以页为单位，默认为每页16KB

## 行格式
- Compact、Redudant、Dynamic、Compressed
- 语句 `CREATE TABLE ... CHARSET=utf8 ROW_FORMAT=COMPACT`
- 行格式 = 记录的额外信息 + 记录的真实数据
### Compact
#### 记录的额外信息
- 变长字段长度列表
  - 数据类型占用的空间包括真正内容和占用的字节数
  - VARCHAR BARBINARY TEXT BLOB等类型
  - 逆序存放长度，有特定规则来计算用几个字节表示长度
  - 只存非NULL的的列内容所占用的长度
- NULL值列表
  - 每个允许NULL的列对应一个二进制位，逆序排列
  - NULL值列表用整数个的字节表示，不够的在高位补0
- 记录头信息
  - 由固定的5个字节组成
  - 包括是否删除标记、B+数算法标记、下一条记录的相对位置等信息
#### 记录的真实数据
- 隐藏列
  - 表没有主键时增加row_id(6字节)
  - transaction_id(6字节)
  - roll_pointer(7字节)
- 不再重复存储NULL
- Char(N)等定长的列不够长的用空格填充
