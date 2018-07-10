## UID GID
- Linux只认识ID，而不是账号，对应关系在/etc/passwd中
- UID为User ID，GID表示Group ID

## /etc/passwd
`adm:x:3:4:adm:/var/adm:/sbin/nologin`
- 账号名称
- 密码 已迁移到/etc/shadow中，只剩余 x
- UID 0表示系统管理员，1~999为系统账号，1000以上的给一般使用者使用
- GID
- 用户信息说明栏
- home目录
- Shell

## ACL(Access Control List)
- 针对user  group mask三个方面来控制权限
