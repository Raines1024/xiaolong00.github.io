---
title: 联表查询并修改sql
date: 2021-01-22 08:15:39
tags: [编程, 过去]
category:
- 200 工作类
- 230 工作技能
- 235 常用sql
---

## 业务需求

退换货原因，需根据车辆编号重新绑定设备号，提供车辆编号对应设备号excel。

## 准备工作

创建临时表t2并导入excel数据。

## sql实现

俩表关联，修改原表设备号为临时表设备号

```sql
UPDATE table_1 t1 left join table_2 t2 on t2.serail_no = t1.serail_no SET t1.dev_no = t2.dev_no
where t1.id>5;
```





































