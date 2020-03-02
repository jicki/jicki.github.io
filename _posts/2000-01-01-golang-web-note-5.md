---
layout: post
title: GORM MYSQL
categories: [golang,Go]
description: GORM MYSQL
keywords: golang,Go
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
---

# Go Web 编程

## GORM

* ORM（Object Relation Mapping）, 对象关系映射, 实际上就是对数据库的操作进行封装, 对上层开发人员屏蔽数据操作的细节, 开发人员看到的就是一个个对象, 大大简化了开发工作, 提高了生产效率

* O = Object 对象 (程序中的对象/实例 如: Struct 结构体)

* R = Relational 关系 (关系型数据库 如: Mysql)

* M = Mapping 映射

  * 结构体 --> 数据表
  * 结构体实例 --> 数据库行
  * 结构体字段 --> 数据库字段



* ORM 优缺点

  * 优点 - 提高开发效率, 不需要使用SQL语句。

  * 缺点 - 执行性能较差、灵活性较差、弱化了SQL能力。


### GORM 概览

1. 全功能 ORM (无限接近)

2. 关联 (Has One, Has Many, Belongs To, Many To Many, 多态)

3. 钩子 (在创建/保存/更新/删除/查找之前或之后)

4. 预加载

5. 事务

6. 复合主键

7. SQL 生成器

8. 数据库自动迁移

9. 自定义日志

10. 可扩展性, 可基于 GORM 回调编写插件

11. 所有功能都被测试覆盖

12. 开发者友好


### gorm 模块用法

```shell
go get -u github.com/jinzhu/gorm

```

* gorm to mysql


```go

import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
）

var db *gorm.DB

func init() {
    var err error
    db, err = gorm.Open("mysql", "<user>:<password>/<database>?charset=utf8&parseTime=True&loc=Local")
    if err != nil {
        panic(err)
    }
    // 设置连接池
    db.DB().SetMaxIdleConns(10)
    db.DB().SetMaxOpenConns(100)
}

```



### GORM Model

* 在使用ORM工具时，通常我们需要在代码中定义模型（Models）与数据库中的数据表进行映射，在GORM中模型（Models）通常是正常定义的结构体、基本的go类型或它们的指针。 同时也支持`sql.Scanner`及`driver.Valuer`、接口`interfaces`。



* GORM 内置了一个`gorm.Model`结构体。`gorm.Model`是一个包含了`ID` 、 `CreatedAt` 、`UpdatedAt` 、 `DeletedAt`四个字段的Golang结构体。

```go
// gorm.Model 定义
type Model struct {
  ID        uint `gorm:"primary_key"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt *time.Time
}
```


* 嵌入到自己的结构体模型中

```go
// 定义 数据模型
type UserInfo struct {
	gorm.Model   // 内嵌 Model 模型
	Name         string
	Age          sql.NullInt64 //零值类型
	Birthday     *time.Time
	Email        string  `gorm:"type:varchar(100);unique_index"`
	Role         string  `gorm:"size:255"`        // 限制大小为最多255
	MemberNumber *string `gorm:"unique;not null"` // 设置会员号唯一,且不能为空。
	Num          int     `gorm:"AUTO_INCREMENT"`  // 设置字段 num 自增类型。
	Address      string  `gorm:"index:addr"`      // 给address字段创建名为addr的索引
	IgnoreMe     int     `gorm:"-"`               // 忽略本字段
}

```



* GORM 结构体标识 (struct tag)

|结构体标识(Tga)|描述|
|-|-|
|Column |指定列名|
|Type |指定列数据类型|
|Size |指定列大小, 默认值255|
|PRIMARY_KEY |将列指定为主键|
|UNIQUE |将列指定为唯一|
|DEFAULT |指定列默认值|
|PRECISION |指定列精度|
|NOT NULL |将列指定为非 NULL|
|AUTO_INCREMENT |指定列是否为自增类型|
|INDEX |创建具有或不带名称的索引, 如果多个索引同名则创建复合索引|
|UNIQUE_INDEX |和 INDEX 类似，只不过创建的是唯一索引|
|EMBEDDED |将结构设置为嵌入|
|EMBEDDED_PREFIX |设置嵌入结构的前缀|
|- |忽略此字段|



* GORM 创建表时 set 的标识

|创建表的标识|描述|
|-|-|
|MANY2MANY |指定连接表|
|FOREIGNKEY |设置外键|
|ASSOCIATION_FOREIGNKEY |设置关联外键|
|POLYMORPHIC |指定多态类型|
|POLYMORPHIC_VALUE |指定多态值|
|JOINTABLE_FOREIGNKEY |指定连接表的外键|
|ASSOCIATION_JOINTABLE_FOREIGNKEY |指定连接表的关联外键|
|SAVE_ASSOCIATIONS |是否自动完成 save 的相关操作|
|ASSOCIATION_AUTOUPDATE |是否自动完成 update 的相关操作|
|ASSOCIATION_AUTOCREATE |是否自动完成 create 的相关操作|
|ASSOCIATION_SAVE_REFERENCE |是否自动完成引用的 save 的相关操作|
|PRELOAD |是否自动完成预加载的相关操作|



* 主键、表名、列名的约定
 
  * 主键 (Primary Key) , GORM 默认会使用名为`ID` 这个字段名作为表的主键。

  * 修改默认的 主键名为 其他, 需要使用结构体 tag `grom:"primary_key"` 来约定。

```go

type User struct {
  ID   string // 名为`ID`的字段会默认作为表的主键
  Name string
}

// 使用`AnimalID`作为主键
type Animal struct {
  AnimalID int64 `gorm:"primary_key"`
  Name     string
  Age      int64
}

```

  
* 表名 (Table Name) 

  * GROM 创建数据表, 表名默认就是结构体名称的复数, 如: `User = users` , `UserInfo = user_infos`

  * 修改默认的 数据表名称。

```go

// 将 User 的表名设置为 `profiles`
func (User) TableName() string {
  return "profiles"
}

// 多个条件判断
func (u User) TableName() string {
  if u.Role == "admin" {
    return "admin_users"
  } else {
    return "users"
  }
}

// 禁用默认表名的复数形式，如果置为 `true，则 `User` 的默认表名是 `user`
db.SingularTable(true)

```


```go
// table 可以指定表名

// 使用User结构体创建名为`deleted_users`的表
db.Table("deleted_users").CreateTable(&User{})

```


```go
// 修改默认表的规则 (格式为job_表明)
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
  return "job_" + defaultTableName;
}
```



* 数据列 名称 (Column Name)

  * 列名 是由 结构体字段的名称组合,一个字段多个单词组成 会使用 `_` 分割开。如: `MemberNumber` = `member_number`

  * 修改默认的 数据列名称, 可使用结构体`tag` 中的 `column` 指定列名。

```go
type Animal struct {
  AnimalId    int64     `gorm:"column:beast_id"`        
  Birthday    time.Time `gorm:"column:beast_day"` 
  Age         int64     `gorm:"column:beast_age"`
}

```


* 时间类型 (At)

  * `CreatedAt`字段, 该字段的值将会是初次创建记录的时间。

  * `UpdatedAt`字段，该字段的值将会是每次更新记录的时间。 

  * `DeletedAt`字段，调用Delete方法删除该记录时，将会设置`DeletedAt`字段为当前时间，而不是直接将记录从数据库中删除。



### GORM CRUD

* CRUD 增、删、改、查


* 增加(create)


```go

// 定义 数据模型
type User struct {
	ID int64
	// 使用 tag default 设置默认值
	Name string `gorm:"default:'小炒肉'"`
	Age  int64
}

func main() {
	dsn := "jicki:jicki123@tcp(127.0.0.1:3306)/jicki?charset=utf8mb4&parseTime=true"

	db, err := gorm.Open("mysql", dsn)
	if err != nil {
		panic(err)
	}

	defer db.Close()

	// 创建 结构体User 对应的数据表
	db.AutoMigrate(&User{})

	// 创建结构体实例
	u1 := User{
		Name: "小炒肉",
		Age:  20,
	}
	// 判断主键是否存在,可用于判断数据是否成功写入。
	db.NewRecord(&u1)
	// 创建(插入)数据
	db.Create(&u1)
}
```

* 注意: 使用 结构体 tag default 设置默认值的时候, 所有字段的零值, 比如`0`, `""`, `false`或者其它零值，都不会保存到数据库内，但会使用他们的默认值。如需要零值/空值, 可以使用: 


* `Name *string `gorm:"default:'小炒肉'"`` 

```go
// 定义 数据模型
type User struct {
	ID int64
	// 使用 tag default 设置默认值
	Name *string `gorm:"default:'小炒肉'"`
	Age  int64
}

func main(){

	// 创建结构体实例
	u1 := User{
		Name: new(string),
		Age:  20,
	}
	// 判断主键是否存在,可用于判断数据是否成功写入。
	db.NewRecord(&u1)
	// 创建(插入)数据
	db.Create(&u1)
}

```



* `Name sql.NullString `gorm:"default:'小炒肉'"``


```go
type User struct {
        ID int64
        // 使用 tag default 设置默认值
        Name sql.NullString `gorm:"default:'小炒肉'"`
        Age  int64
}

func main(){

        // 创建结构体实例
        u1 := User{
                Name: sql.NullString{"", true},
                Age:  20,
        }
        // 判断主键是否存在,可用于判断数据是否成功写入。
        db.NewRecord(&u1)
        // 创建(插入)数据
        db.Create(&u1)
}

```


* 查询 (First、Take、Last、Find)

```go
func main() {
	// 查询 select
	// 定义一个User结构体类型的变量用于存储查询返回的数据
	var user User

	// 定义一个 User结构体类型的切片,用于存储查询返回的多条数据
	var users []User

	// First 取第一条数据
	db.First(&user)
	fmt.Printf("DB First User: %v \n", user)

	// Take 随机取一条数据
	db.Take(&user)
	fmt.Printf("DB Take User: %v \n", user)

	// Last 根据主键 的顺序查询最后一条数据
	db.Last(&user)
	fmt.Printf("DB Last User: %v \n", user)

	// Find 查询所有的数据
	db.Find(&users)
	for _, user := range users {
		fmt.Printf("DB Find Users: %v \n", user)
	}

	// First 也可以查询指定的数据, 当主键为整型时可用
	db.Debug().First(&user, 2)
	fmt.Printf("DB First 2 User: %v \n", user)
}

```


* 条件过滤 (Where)


```go
func main() {

	// Where 条件查询

	// 过滤查询单条的数据
	db.Where("name = ?", "大炒肉").First(&user)
	fmt.Printf("Where First: %v \n", user)

	// 过滤查询所有的数据
	db.Where("name = ?", "大炒肉").Find(&users)
	fmt.Printf("Where Find: %v \n", users)

	// 过滤查询所有的数据 <> 不等于
	db.Where("name <> ?", "炒肉").Find(&users)
	fmt.Printf("Where Find <> : %v \n", users)

	// 多个条件 IN 查询所有数据
	db.Where("name IN(?)", []string{"小炒肉", "大炒肉"}).Find(&users)
	fmt.Printf("Where Find IN : %v \n", users)

	// 模糊匹配 查询所有的数据
	db.Where("name LIKE ?", "%小%").Find(&users)
	fmt.Printf("Where Find LIKE : %v \n", users)

	// AND 匹配
	db.Where("name =? AND age = ?", "小炒肉", "20").Find(&users)
	fmt.Printf("Where Find AND : %v \n", users)

	// Time 时间匹配
	db.Where("updated_at > ?", "2020-03-02 06:41:44").Find(&users)
	fmt.Printf("Where Find Time : %v \n", users)

	// BETWEEN 区间时间
	db.Where("created_at BETWEEN ? AND ?", "2020-01-01 09:09:01", "2020-03-02 06:41:50").Find(&users)
	fmt.Printf("Where Find BETWEEN : %v \n", users)

	// Where Struct 查询条件进行过滤查询
	db.Where(&User{Name: "小炒肉", Age: 20}).First(&user)
	fmt.Printf("Where Struct : %v \n", user)

	// Where Map 查询条件进行过滤查询
	db.Where(map[string]interface{}{"name": "大炒肉", "age": 30}).Find(&users)
	fmt.Printf("Where Map : %v \n", users)

	// Where 主键切片 查询条件 进行过滤查询
	db.Where([]int64{1, 2}).Find(&users)
	fmt.Printf("Where 切片 : %v \n", users)
}
```






### 实际操作实例


```go

package main

import (
	"fmt"

	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)

/*
数据库 student 表

CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  `nick` varchar(64) DEFAULT NULL,
  `country` varchar(128) DEFAULT NULL,
  `province` varchar(64) DEFAULT NULL,
  `city` varchar(64) DEFAULT NULL,
  `status` int(11) DEFAULT NULL,
  `create_time` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;

*/

// 创建一个 student 的结构体

type student struct {
	// gorm.Model 嵌入常用数据库字段.
	// gorm.Model
	// gorm 用 tag 的方式来标识 mysql 里面的约束
	Id         int    `db:"id" gorm:"primary_key;type: int(11);NOT NULL;AUTO_INCREMENT"`
	Name       string `db:"name" gorm:"type: varchar(64);NOT NULL"`
	Nick       string `db:"nick" gorm:"type: varchar(64);DEFAULT NULL"`
	Country    string `db:"country" gorm:"type: varchar(128);DEFAULT NULL"`
	Province   string `db:"province" gorm:"type: varchar(64);DEFAULT NULL"`
	City       string `db:"city" gorm:"type: varchar(64);DEFAULT NULL"`
	Status     int    `db:"status" gorm:"type: int(11);DEFAULT NULL"`
	CreateTime string `db:"create_time" gorm:"type: varchar(32);DEFAULT NULL"`
}

func main() {
	// 连接数据库信息
	dsn := "root:root@tcp(127.0.0.1:3306)/jicki?parseTime=true"

	// 连接数据库
	DB, err := gorm.Open("mysql", dsn)
	if err != nil {
		panic(err)
	}

	// 设置连接池
	DB.DB().SetMaxIdleConns(10)
	DB.DB().SetMaxOpenConns(100)

	// 关闭数据库
	defer DB.Close()

	// 检查模型`student`表是否存在
	DB.HasTable(&student{})

	// 检查表 `student` 是否存在
	//DB.HasTable("student")

	// 删除数据库
	DB.DropTableIfExists(&student{})

	// 创建表, 会自动创建 结构体对应 数据表, Migrate是迁移的意思, 自动迁移仅仅会创建表
	DB.Set("gorm:table_options", "ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;").AutoMigrate(&student{})

	// 按照构建 结构体的方式 插入数据
	DB.Create(&student{
		Id:       1,
		Name:     "张大仙",
		Nick:     "大仙",
		Country:  "China",
		Province: "广东",
		City:     "深圳",
		Status:   1,
	})

	DB.Create(&student{
		Id:       2,
		Name:     "黄大仙",
		Nick:     "大仙",
		Country:  "China",
		Province: "广东",
		City:     "深圳",
		Status:   1,
	})

	DB.Create(&student{
		Id:       3,
		Name:     "陈小仙",
		Nick:     "小仙",
		Country:  "China",
		Province: "广东",
		City:     "深圳",
		Status:   1,
	})

	// 查询数据
	//DB.Find 查询整个数据表 DB.First 查询一条记录
	var stu student
	var stus []student

	// 获取第一条记录，按主键排序
	DB.First(&stu)
	fmt.Printf("First 获取记录: %#v\n", stu)

	// 获取所有记录
	DB.Find(&stus)
	for _, v := range stus {
		fmt.Printf("Find 获取所有记录: %#v\n", v)
	}

	// 删除数据 (delete from student where id = 1;)
	DB.Delete(&student{}, "id = ?", 1)

	// 更新数据 (update student set name = '猪八戒' where name = '黄大仙';)
	err = DB.Model(&student{}).Where("Name = ?", "黄大仙").Update("Name", "猪八戒").Error
	if err != nil {
		fmt.Println(err)
		return
	}

	// 重新获取所有记录
	DB.Find(&stus)
	for _, v := range stus {
		fmt.Printf("Delete and Update 后记录: %#v\n", v)
	}
}

```

* 输出:

```shell

First 获取记录: main.student{Id:1, Name:"张大仙", Nick:"大仙", Country:"China", Province:"广东", City:"深圳", Status:1, CreateTime:""}
Find 获取所有记录: main.student{Id:1, Name:"张大仙", Nick:"大仙", Country:"China", Province:"广东", City:"深圳", Status:1, CreateTime:""}
Find 获取所有记录: main.student{Id:2, Name:"黄大仙", Nick:"大仙", Country:"China", Province:"广东", City:"深圳", Status:1, CreateTime:""}
Find 获取所有记录: main.student{Id:3, Name:"陈小仙", Nick:"小仙", Country:"China", Province:"广东", City:"深圳", Status:1, CreateTime:""}
Delete and Update 后记录: main.student{Id:2, Name:"猪八戒", Nick:"大仙", Country:"China", Province:"广东", City:"深圳", Status:1, CreateTime:""}
Delete and Update 后记录: main.student{Id:3, Name:"陈小仙", Nick:"小仙", Country:"China", Province:"广东", City:"深圳", Status:1, CreateTime:""}

```
