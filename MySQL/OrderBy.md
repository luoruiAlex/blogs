## Order By有两种排序实现方式
### 利用有序索引获取有序数据(explain显示Using index)
```
  CREATE TABLE `test` (  
    `id` int(11) NOT NULLAUTO_INCREMENT,  
    `rdate` datetime NOT NULL,  
    `inventid` int(11) NOT NULL,  
    `customerid` int(11) NOT NULL,  
    `staffid` int(11) NOT NULL,  
    `data` varchar(20) NOT NULL,  
    PRIMARY KEY (`id`),  
    UNIQUE KEY `rdate`(`rdate`,`inventid`,`customerid`),  
    KEY `inventid` (`inventid`),  
    KEY `customerid` (`customerid`),  
    KEY `staffid` (`staffid`)  
  ) ENGINE=InnoDB AUTO_INCREMENT=27 DEFAULT CHARSET=latin1  
```
- MySQL在查询时最多只能使用一个索引。因此，如果WHERE条件已经占用了索引，那么在排序中就不使用索引了
- 返回选择的字段，即只包括在有选择的此列上（select后面的字段）
`explain select inventid from test where rdate='2011-12-1400:00:00' order by  inventid;`
- Select选择的列使用索引则order by不使用索引
`explain select * from test where customerid=1  order by  inventid;`
### 文件排序(explain显示Using filesort)
