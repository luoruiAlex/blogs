- 书写顺序
```
SELECT
DISTINCT <select_list>
FROM <left_table>
<join type> JOIN <right_table> ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```
- 执行顺序
```
FROM <left_table>                   #笛卡尔积
ON <join_condition>                 #主表保留
<join type> JOIN <right_table>      #不符合ON也添加
WHERE <where_condition>             #非聚合、非SELECT别名
GROUP BY <group_by_list>            #改变对表引用
HAVING <having_condition>           #只作用分组后
SELECT
DISTINCT <select_list>
ORDER BY <order_by_condition>       #可使用SELECT别名
LIMIT <limit_number>                #rows offset
```
