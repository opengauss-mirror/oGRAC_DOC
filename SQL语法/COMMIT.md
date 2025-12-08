# COMMIT

#### 功能描述

该语句使当前事务工作单元中的所有操作“永久化”，并结束该事务。

#### 注意事项

Gauss100 OLTP中的数据操作（SELECT、DELETE、UPDATE）提交是默认关闭的，会话退出时，需要显式COMMIT，否则记录将丢失。

#### 语法格式

```
COMMIT [ TRANSACTION | PREPARED XID | FORCE LTID ]
```

#### 参数说明

- TRANSACTION可选关键字，用于增加语句可读性，等同于单独执行COMMIT操作。

- PREPARED*XID*

- 提交处于已完成第一阶段提交的全局事务分支。

- XID事务标示符。**XID**格式：点分字符串 ,其包括以下3部分：

- FORMAT ID：整型值, 有效范围：[0, 9223372036854775807]。

- GLOBAL TRANSACTION ID：BASE16格式，长度小于等于128B。

- BRANCH ID：BASE16格式，长度小于等于128B。

- FORCE*LTID*用于在故障修复时，提交因页面损坏而产生的无法回滚的本地事务。

- *LTID*点分字符串，其中格式为*SEG_ID*.*SLOT*.*XNUM*，字段值可通过查询视图DV_TRANSACTIONS中SEG_ID、SLOT、XNUM字段获取。

#### 示例

创建表training，插入数据并更新数据，提交操作后结束该事务。

```
--删除表training。
DROP TABLE IF EXISTS training;
--创建表training。
CREATE TABLE training(staff_id INT NOT NULL, staff_name VARCHAR(16), course_name CHAR(20), course_start_date DATETIME, course_end_date DATETIME, exam_date DATETIME, score INT);
--向表training中插入记录1。
INSERT INTO training(staff_id,staff_name,course_name,course_start_date,course_end_date,exam_date,score) VALUES(10,'LIPENG','JAVA','2017-06-15 12:00:00','2017-06-20 12:00:00','2017-06-25 12:00:00',90);
--向表training中插入记录2。
INSERT INTO training(staff_id,staff_name,course_name,course_start_date,course_end_date,exam_date,score) VALUES(11,'CAOM','JAVA','2017-06-20 12:00:00','2017-06-25 12:00:00','2017-06-26 12:00:00',95);
--更新记录2中的staff_name字段和course_name字段。
UPDATE training SET staff_name='WANGPAN', course_name='INFORMATION SAFETY' WHERE staff_id=11;
--提交事务。
COMMIT;
```
