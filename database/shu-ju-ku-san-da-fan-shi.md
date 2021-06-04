---
description: 这里以通俗的语言来理解数据库的三大范式，避免了生涩的专业性词汇。
---

# MySQL三大范式

## 第一范式

　　第一范式：列的原子性，即每一列（每一个属性，字段）都不可分割。 举例：销售成本=成本的单价\*销售的数量，所以这里就不可以以销售成作为字段。

## 第二范式

 　　第二范式：非主属性必须完全依赖于主属性，不能存在只依赖于主属性的一部分属性。 主键是唯一的，用来确定每一行数据的。学生信息表由，学号，姓名，性别，年龄组成，这里不能以姓名作为主键，因为可能存在同名，不是唯一的，学号是唯一的，所以选择作为主键。

　　“张三”的学号对应了自己的姓名，年龄，性别，这里年龄性别，不能存储别人的信息，不能只有姓名，年龄依赖于学号，必须所有的信息都依赖于学号。

## 第三范式

　　第三范式：消除依赖的传递。 这里可以理解为消除冗余，这里以学生信息表（同上），院系表（主键系编号，属性：系名字，系主任），如果把院系表的系编号，系名字，系主任都存储到“张三”的信息中，这里就会发现数据冗余，可能许多学生的院系信息都是相同的，我们只需要将系编号这个字段存储到学生信息表即可。通过系编号即可查询院系表对应信息，第三范式消除了依赖的传递。 
