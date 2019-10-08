# 用Docker搭建外部可以访问的mysql

## 安装mysql 8.0

```
docker run -p 63306:3306 -e MYSQL_ROOT_PASSWORD=zhaooleemysql --name zhaooleemysqldb -d mysql:8.0
```

- `p 53306:3306` 将docker容器的3306端口映射到宿主机的63306端口
- `-e MYSQL_ROOT_PASSWORD=zhaooleemysql` root用户登录密码为 zhaooleemysql
- `--name zhaooleemysqldb` 新建容器的名称为zhaooleemysqldb
- `mysql:8.0` 使用的mysql数据库版本为8.0

## 进入容器

```
docker exec -it zhaooleemysqldb bash
```

> ![img](用Docker搭建外部可以访问的mysql.assets/16cf9bc55c48931e)

## 登录数据库

```
mysql -uroot -p
zhaooleemysql
```

## 新建用户(8.0更为严格,用root用户远程登录比较麻烦, 我们选择新建一个用户)

```
create user 'zhaoolee' identified with mysql_native_password by 'eelooahzpw';
```

`zhaoolee` 新用户名为zhaoolee `mysql_native_password` 密码加密方式为`mysql_native_password``eelooahzpw` 新用户的密码为eelooahzpw

## 为新用户zhaoolee增加权限

```
grant all privileges on *.* to 'zhaoolee';
```

## 刷新权限

```
flush privileges;
```

> ![img](用Docker搭建外部可以访问的mysql.assets/16cf9bc55d91499c)

## 新建数据库

```
create database v2fy charset=utf8;
```

- 新数据库的名称为 `v2fy`

## 退出数据库

```
exit
```

## 退出docker容器

```
control +  p  + q
```

> ![img](用Docker搭建外部可以访问的mysql.assets/16cf9bc55dae36e0)

## 尝试用Navicat连接

> ![img](用Docker搭建外部可以访问的mysql.assets/16cf9bc55dbe7ca0)

> ![img](用Docker搭建外部可以访问的mysql.assets/16cf9bc59cd94f6e)