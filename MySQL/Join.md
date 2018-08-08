- 语法 
```
  SELECT <row_list>
    FROM <left_table>
      <inner|left|right> JOIN <right_table>
      ON <join condition>
      WHERE <where condition>
```

- 执行顺序
  - FROM：左右表执行笛卡尔积，产生vv1
  - ON：筛选vt1，结果插入vt2
  - JOIN：添加外部行
    - LEFT JOIN遍历左表每一行，不在vt2的行会插入vt2(剩余字段为NULL)，形成vt3
    - INNER JOIN，vt3=vt2
  - WHERE：筛选vt3，输出vt4
  - SELECT：取出vt4指定字段到vt5
  
  - MySQL不支持FULL JOIN，用LEFT JOIN \+ UNION \+ RIGHT JOIN
  
  - INNER JOIN中ON和WHERE没有区别
