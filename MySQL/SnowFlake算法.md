## 作用与特点
- 在分布式系统中生成ID
- 生成的ID整体上按照时间自增排序，且整个系统内无ID碰撞
## SnowFlake算法生成的ID共有64bit
- 第1位，值始终是0，无实际作用
- 时间戳，41bit，精确到了毫秒
- 工作机器ID，10bit，5bit的datacenterId+5bitworkerId
- 序列号，12bit
## 同一毫秒的ID数量 = 1024 * 4094
