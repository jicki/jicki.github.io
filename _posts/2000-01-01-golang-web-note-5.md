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


### gorm 用法

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




### 实际操作实例


```go

package main

import (
	"fmt"

	_ "github.com/go-sql-driver/mysql"

	"github.com/jinzhu/gorm"
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
