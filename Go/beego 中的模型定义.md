

# beego 中的模型定义

## 默认表名

默认的表名规则，使用驼峰转蛇形。例如`AuthUser -> auth_user`，除了开头的大写字母以外，遇到大写会增加 `_`，原名称中的下划线保留。

## 自定义表明

```go
func (u *User) TableName() string {
    return "auth_user"
}
```

## 字段参数

多个设置间使用 `;` 分隔，设置的值如果是多个，使用 `,` 分隔。如`orm:"null;rel(fk)"`。

```go
// 忽略字段
`orm:"-"`
// 自增健
`auto`   // 当 Field 类型为 int, int32, int64, uint, uint32, uint64 时，可以设置字段为自增健
// 当模型定义里没有主键时，符合上述类型且名称为 Id 的 Field 将被视为自增健。

// 主键
`pk` // 适用于自定义其他类型为主键
// 空值
`orm:"null"`  // 数据库表默认为 NOT NULL，设置 null 代表 ALLOW NULL
// 索引
`orm:"index"`  // 多字段索引使用 func (u *User) TableIndex() [][]string{}定义
// 唯一
`orm:"unique"`  // 多字段唯一使用 func (u *User) TableUnique() [][]string {}定义

// 自定义字段名称
`orm:"column(user_name)"`
// 字符字段长度
`orm:"size(60)"`  // string 类型字段默认为 varchar(255),设置 size 以后，db type 将使用 varchar(size)
// 浮点精度
`orm:"digits(12);decimals(4)"`  // 总长度 12 小数点后 4 位 eg: 99999999.9999。设置 float32, float64 类型的浮点精度

// 对于批量的 update 此设置不生效

// 字符字段   默认是varchar型
`orm:"type(text)"`  // 数据库中对应text类型
`orm:"type(char)"`  // 数据库中对应定长char类型

// 时间字段类型
`orm:"auto_now_add;type(date)"`
`orm:"auto_now_add;type(datetime)"`
`orm:"type(datetime);precision(4)"`  // 为datetime字段设置精度值位数，不同DB支持最大精度值位数也不一致。
// 时间自动更新
`orm:"auto_now_add;type(datetime)"` // 第一次保存时才设置时间
`orm:"auto_now;type(datetime)"` 	// 每次 model 保存时都会对时间自动更新

// 默认值
`default:"12"`
`orm:"default(13)"`

// 注释 为字段添加注释
`orm:"default(1);description(这是状态字段)"`
// 注意: 注释中禁止包含引号
```

## 表关系

表的关系通过给字段设置`rel / reverse`。

### 一对一

```go
type User struct {
    ...
    Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
    ...
}
```

对应的反向关系 **RelReverseOne**:

```go
type Profile struct {
    ...
    User *User `orm:"reverse(one)"`
    ...
}
```

### 一对多

```go
type Post struct {
    ...
    User *User `orm:"rel(fk)"` // RelForeignKey relation
    ...
}
```

对应的反向关系 **RelReverseMany**:

```go
type User struct {
    ...
    Posts []*Post `orm:"reverse(many)"` // fk 的反向关系
    ...
}
```
### 多对多
```
type Post struct {
    ...
    Tags []*Tag `orm:"rel(m2m)"` // ManyToMany relation
    ...
}
```

对应的反向关系 **RelReverseMany**:

```
type Tag struct {
    ...
    Posts []*Post `orm:"reverse(many)"`
    ...
}
```

#### rel_table / rel_through

此设置针对 `orm:"rel(m2m)"` 的关系字段

```
rel_table       设置自动生成的 m2m 关系表的名称
rel_through     如果要在 m2m 关系中使用自定义的 m2m 关系表
                通过这个设置其名称，格式为 pkg.path.ModelName
                eg: app.models.PostTagRel
                PostTagRel 表需要有到 Post 和 Tag 的关系
```

当设置 rel_table 时会忽略 rel_through

设置方法：

```
orm:"rel(m2m);rel_table(the_table_name)"
orm:"rel(m2m);rel_through(pkg.path.ModelName)"
```

### 级联删除

设置对应的 rel 关系删除时，如何处理关系字段。

```go
cascade        级联删除(默认值)
set_null       设置为 NULL，需要设置 null = true
set_default    设置为默认值，需要设置 default 值
do_nothing     什么也不做，忽略
```
```go
type User struct {
    ...
    Profile *Profile `orm:"null;rel(one);on_delete(set_null)"`
    ...
}
type Profile struct {
    ...
    User *User `orm:"reverse(one)"`
    ...
}
// 删除 Profile 时将设置 User.Profile 的数据库字段为 NULL
```