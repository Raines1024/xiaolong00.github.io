---
title: 存储过程介绍
date: 2021-05-14 20:01:39
description: 《MYSQL必知必会》读书笔记单独学习
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 113 数据库



---

## 背景

sql真的很好，不骗你。

## 创建存储过程

```
CREATE PROCEDURE productpricing()
BEGIN
	SELECT AVG(id) AS idavg
	FROM l_biz_task;
END;
```

## 使用存储过程

```
CALL productpricing()
```

## 删除存储过程

```
DROP PROCEDURE IF EXISTS productpricing 
```

## 使用参数

```
CREATE PROCEDURE productpricing(
	OUT pl DECIMAL(8,2),
	OUT ph DECIMAL(8,2),
	OUT pa DECIMAL(8,2)
)
BEGIN
	SELECT MIN(id)
	INTO pl
	FROM l_biz_task;
	SELECT MAX(id)
	INTO ph
	FROM l_biz_task;
	SELECT AVG(id)
	INTO pa
	FROM l_biz_task;
END;
## 使用存储过程
CALL productpricing(@idlow,
										@idhigh,
										@idavg)
SELECT @idlow,@idhigh,@idavg
```

## 存储过程进阶及游标使用


```
## 使用in和out参数，in传入，out从存储过程返回合计。select语句使用这两个参数，where子句使用odel选择正确的行，INTO使用ototal存储计算出来的合计。
DROP PROCEDURE IF EXISTS idtotal;
CREATE PROCEDURE idtotal(
	IN odel INT,
	OUT ototal DECIMAL(8,2)
)
BEGIN
	SELECT SUM(id)
	FROM l_biz_task
	WHERE del_flag = odel
	INTO ototal;
END;

## 调用存储过程
CALL idtotal(1,@total);
## 显示此合计
SELECT @total;
## 调用存储过程
CALL idtotal(0,@total);
## 显示此合计
SELECT @total;


DROP PROCEDURE IF EXISTS idtotal;

CREATE PROCEDURE idtotal(
	IN odel INT,
	IN taxable BOOLEAN,
	OUT ototal DECIMAL(8,2)
) 
BEGIN
	DECLARE total DECIMAL(8,2);
	DECLARE taxrate INT DEFAULT 6;
	
	SELECT SUM(id)
	FROM l_biz_task
	WHERE del_flag = odel
	INTO ototal;
	
	IF taxable THEN
		SELECT total+(total/100*taxrate) INTO total;
		SELECT total INTO ototal;
	END IF;

END

## boolean值1表示真，0表示假。
CALL idtotal(0,0,@total);
SELECT @total;

## 显示用来创建一个存储过程的CREATE语句
SHOW create PROCEDURE idtotal





## FETCH用来检索当前行的id列到一个名为o的局部变量中。
CREATE PROCEDURE processids()
BEGIN
	DECLARE o INT;
	
	DECLARE ids CURSOR
	FOR
	SELECT id FROM l_biz_task;
	OPEN ids;
	FETCH ids INTO o;
	CLOSE ids;
END;
CALL processids();

CREATE PROCEDURE processids()
BEGIN
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	
	DECLARE ids CURSOR
	FOR
	SELECT id FROM l_biz_task;
	
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
	
	OPEN ids;
	
	REPEAT
		FETCH ids INTO o;
	UNTIL done END REPEAT;
	
	CLOSE ids;
END;

DROP PROCEDURE IF EXISTS processids;
CREATE PROCEDURE processids()
BEGIN
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	DECLARE t DECIMAL(8,2);
	
	DECLARE ids CURSOR
	FOR
	SELECT id FROM l_biz_task;
	
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
	
	CREATE TABLE IF NOT EXISTS ids(id int,total DECIMAL(8,2));
	
	OPEN ids;
	
	REPEAT
		FETCH ids INTO o;
		CALL idtotal(0,0,t);
		INSERT INTO ids(id,total) VALUES(o,t);
	UNTIL done END REPEAT;
	
	CLOSE ids;
END;

CALL processids();
```











































