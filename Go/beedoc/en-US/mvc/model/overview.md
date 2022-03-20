---
name: Overview
sort: 1
---

# Models － Beego ORM

[![Build Status](https://drone.io/github.com/beego/beego/v2/status.png)](https://drone.io/github.com/beego/beego/v2/latest) [![Go Walker](http://gowalker.org/api/v1/badge)](http://gowalker.org/github.com/beego/beego/v2/client/orm)

Beego ORM is a powerful ORM framework written in Go. It is inspired by Django ORM and SQLAlchemy.

This framework is still under development so compatibility is not guaranteed.

**Supported Database:**

* MySQL：[github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
* PostgreSQL：[github.com/lib/pq](https://github.com/lib/pq)
* Sqlite3：[github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)

All of the database drivers have passed the tests, but we still need your feedback and bug reports.

**ORM Features:**

* Supports all the types in Go.
* CRUD is easy to use.
* Auto join connection tables.
* Compatible with crossing database queries.
* Supports raw SQL query and mapping.
* Strict and well-covered test cases ensure the ORM's stability.

You can learn more in this documentation.

**Install ORM:**

	go get github.com/beego/beego/v2/client/orm

## Quickstart

### Demo

```go
package main

import (
	"fmt"
	"github.com/beego/beego/v2/client/orm"
	_ "github.com/go-sql-driver/mysql" // import your required driver
)

// Model Struct
type User struct {
	Id   int
	Name string `orm:"size(100)"`
}

func init() {
	// register model
	orm.RegisterModel(new(User))

	// set default database
	orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)
}

func main() {
	o := orm.NewOrm()

	user := User{Name: "slene"}

	// insert
	id, err := o.Insert(&user)
	fmt.Printf("ID: %d, ERR: %v\n", id, err)

	// update
	user.Name = "astaxie"
	num, err := o.Update(&user)
	fmt.Printf("NUM: %d, ERR: %v\n", num, err)

	// read one
	u := User{Id: user.Id}
	err = o.Read(&u)
	fmt.Printf("ERR: %v\n", err)

	// delete
	num, err = o.Delete(&u)
	fmt.Printf("NUM: %d, ERR: %v\n", num, err)
}
```
	
### Relation Query

```go
type Post struct {
	Id    int    `orm:"auto"`
	Title string `orm:"size(100)"`
	User  *User  `orm:"rel(fk)"`
}

var posts []*Post
qs := o.QueryTable("post")
num, err := qs.Filter("User__Name", "slene").All(&posts)
```

### Raw SQL query

You can always use raw SQL to query and mapping.

```go
var maps []Params
num, err := o.Raw("SELECT id FROM user WHERE name = ?", "slene").Values(&maps)
if num > 0 {
	fmt.Println(maps[0]["id"])
}
```

### Transactions

```go
o.Begin()
...
user := User{Name: "slene"}
id, err := o.Insert(&user)
if err == nil {
	o.Commit()
} else {
	o.Rollback()
}
```

### Debugging query log

In development environment, you can enable debug mode by:

```go
func main() {
	orm.Debug = true
...
```

It will output every query statement including execution, preparation and transactions.

For example:

```go
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - 	[INSERT INTO `user` (`name`) VALUES (?)] - `slene`
...
```

Notes: It is not recommended to enable debug mode in a production environment.

## Index

1. [Orm Usage](orm.md)
	- [Set up database](orm.md#set-up-database)
		* [Register Driver](orm.md#registerdatabase)
		* [Variables Config](orm.md#setmaxidleconns)
		* [Timezone Config](orm.md#timezone-config)
	- [Registering Model](orm.md#registering-model)
	- [ORM API Usage](orm.md#orm-api-usage)
	- [Print Out SQL Query in Debugging Mode](orm.md#print-out-sql-query-in-debugging-mode)
2. [CRUD of Object](object.md)
3. [Advanced Queries](query.md)
	- [expr](query.md#expr)
	- [Operators](query.md#operators)
	- [Advanced query API](query.md#advanced-query-api)
	- [Relational Queries](query.md#relational-query)
	- [Load Related Fields](query.md#load-related-field)
	- [Handling ManyToMany Relation](query.md#handling-manytomany-relation)
4. [Use Raw SQL](rawsql.md)
5. [Transactions](transaction.md)
6. [Model Definition](models.md)
	- [Custom Table Names](models.md#custom-table-name)
	- [Custom engine](models.md#custom-engine)
	- [Set Parameters](models.md#set-parameters)
	- [Relationship](models.md#relationship)
	- [Model Fields Mapping with Database Type](models.md#model-fields-mapping-with-database-type)
7. [Command Line](cmd.md)
	- [Table Auto generating](cmd.md#table-auto-generating)
	- [Print SQL Statements](cmd.md#print-sql-statements)
8. [Test ORM](test.md)
9. [Custom Fields](custom_fields.md)
10. [FAQ](faq.md)
