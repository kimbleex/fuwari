---
title: 一些很方便但是容易忘记的SQL语句
category: School
tags: [数据库]
abbrlink: Useful-SQL
published: 2023-02-19 00:00:00
uppublishedd: 2023-02-19 00:00:00
ai: 
  - 本文记录了几条很实用但是容易忘记的SQL语句。
---

有一些方便数据库操作的`SQL`语句，它可以简化一些代码。总是或多或少忘记，记录一下。

我这里拿一个`students`表举例，表结构如下：

|id|class_id|name|gender|score|
|---|---|---|---|---|
|1|1|张三|男|90|
|2|1|李四|男|85|
|3|2|王五|女|95|
|4|2|赵六|女|88|

## 插入或替换

如果我们希望插入一条新记录`INSERT`，但如果记录已经存在，就先删除原记录，再插入新记录。此时，可以使用`REPLACE`语句，这样就不必先查询，再决定是否先删除再插入。

```sql
REPLACE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);
```

若`id=1`的记录不存在，`REPLACE`语句将插入新记录，否则，当前`id=1`的记录将被删除，然后再插入新记录。

## 插入或更新

如果我们希望插入一条新记录`INSERT`，但如果记录已经存在，就啥事也不干直接忽略，此时，可以使用`INSERT IGNORE INTO ...`语句。

```sql
INSERT INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99) ON DUPLICATE KEY UPpublished name='小明', gender='F', score=99;
```

若`id=1`的记录不存在，`INSERT`语句将插入新记录，否则，当前`id=1`的记录将被更新，更新的字段由`UPpublished`指定。

## 插入或忽略

如果我们希望插入一条新记录`INSERT`，但如果记录已经存在，就啥事也不干直接忽略，此时，可以使用`INSERT IGNORE INTO ...`语句.

```sql
INSERT IGNORE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);
```

若`id=1`的记录不存在，`INSERT`语句将插入新记录，否则，不执行任何操作。

## 快照

如果想要对一个表进行快照，即复制一份当前表的数据到一个新表，可以结合`CREATE TABLE`和`SELECT`。

```sql
CREATE TABLE students_of_class1 SELECT * FROM students WHERE class_id=1;
```

## 写入查询结果集

如果查询结果集需要写入到表中，可以结合`INSERT`和`SELECT`，将`SELECT`语句的结果集直接插入到指定表中。

例如，创建一个统计成绩的表`statistics`，记录各班的平均成绩。

```sql
CREATE TABLE statistics (
    id BIGINT NOT NULL AUTO_INCREMENT,
    class_id BIGINT NOT NULL,
    average DOUBLE NOT NULL,
    PRIMARY KEY (id)
);
```

然后，我们就可以用一条语句写入各班的平均成绩。

```sql
INSERT INTO statistics (class_id, average) SELECT class_id, AVG(score) FROM students GROUP BY class_id;
```

确保`INSERT`语句的列和`SELECT`语句的列能一一对应，就可以在`statistics`表中直接保存查询的结果。
