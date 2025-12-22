# COMMIT

#### 功能描述

提交当前事务，使事务中的所有数据操作永久生效并结束该事务。

#### 注意事项

数据操作（`SELECT`、`DELETE`、`UPDATE`）默认不会自动提交。会话退出时，必须显式执行 `COMMIT`，否则所有未提交的更改将会丢失。
关于事务的提交机制，说明如下：
1、对于 DML的事务操作，必须通过显式执行 `COMMIT` 或 `ROLLBACK` 来完成提交或回滚。  **特殊情况**：如果事务中执行了 DDL语句，则该 DDL 语句执行时，会自动提交之前所有未提交的 DML 操作。
此外，若 DML 操作在执行过程中部分失败，不影响后面DML的继续执行和事务提交操作。

2、在执行了一个DDL语句后，不能对之前的DML操作进行回滚。

同时，DDL 操作本身在成功执行后会自动提交，失败则会自动回滚，无需用户显式commit/rollback，且执行成功后不可回滚。

#### 语法格式

```
COMMIT [ TRANSACTION | PREPARED XID | FORCE LTID ]
```

#### 参数说明

| 语法                       | 说明                                                     |
| ------------------------ | ------------------------------------------------------ |
| `COMMIT [ TRANSACTION ]` | 提交当前事务。`TRANSACTION` 为可选的增强可读性关键字，与直接执行 `COMMIT` 效果相同。 |
| `COMMIT PREPARED 'XID'`  | 提交已完成的全局事务分支。                                          |
| `COMMIT FORCE 'LTID'`    | 在故障恢复场景下，强制提交因页面损坏等原因而无法正常回滚的本地事务。                     |

标识符说明：

- `XID`（事务标识符）  
  格式为点分字符串：`FormatID.GlobalTransactionID.BranchID`
  
  - `FormatID`：整数，有效范围 `[0, 9223372036854775807]`
  
  - `GlobalTransactionID`：BASE16 编码字符串，长度 ≤ 128 字节
  
  - `BranchID`：BASE16 编码字符串，长度 ≤ 128 字节

- `LTID`（本地事务标识符）  
  格式为点分字符串：`SEG_ID.SLOT.XNUM`  
  各字段值可通过查询系统视图 `DV_TRANSACTIONS` 中的 `SEG_ID`、`SLOT`、`XNUM` 列获取。

#### 示例

创建表training，插入数据并更新数据，提交操作后结束该事务。

```
--删除表training
DROP TABLE IF EXISTS training;

--创建表training
CREATE TABLE training(staff_id INT NOT NULL, staff_name VARCHAR(16), course_name CHAR(20), course_start_date DATETIME, course_end_date DATETIME, exam_date DATETIME, score INT);

--向表中插入第一条记录
INSERT INTO training(staff_id,staff_name,course_name,course_start_date,course_end_date,exam_date,score) VALUES(10,'zhangsan','cpp','2025-11-23 12:00:00','2025-11-24 12:00:00','2025-11-25 12:00:00',72);

--向表中插入第二条记录
INSERT INTO training(staff_id,staff_name,course_name,course_start_date,course_end_date,exam_date,score) VALUES(11,'lisi','cpp','2025-11-26 12:00:00','2025-11-27 12:00:00','2025-11-28 12:00:00',87);

--更新第二条记录中的staff_name字段和course_name字段
UPDATE training SET staff_name='lisi', course_name='INFORMATION SAFETY' WHERE staff_id=11;

--提交事务。
COMMIT;
```
