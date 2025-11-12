  注意，对于带CSF属性的HASH分区来说，由于HASH分区添加时会导致数据重分布（数据重分布式时会做CSF属性的约束校验),故带CSF属性的HAHS分区添加时可能会报错，报错与否取决于是否满足CSF属性约束。
   
   - COMPRESS
   添加一个压缩分区。需要保此压缩分区所处的表空间内具有压缩属性文件，否则插入数据时会报错。
- drop_paerition_clause
  - DROP PARTITION partition_name
    删除分区，partition_name 为分区名
  - DROP SUBPARTITION subparition_name
    删除二级分区的子分区，subpartiion_name为子分区的名字。
- truncate_partition_clause
清空分区子句，partition_name为分区名称，subparition_name为子分区名。
  - DROP STORAGE
  清除存储空间.
  - REUSE STORAGE
  重用空间。
  - PUGRGE
  清空回收站。
- coalesce_partition_clause
  - COALESCE PARTITION
  将最后一个分区的数据插入到前边的某个分区里，再把最后一个分区删除。
    - 仅限HASH分区才能执行COALESCE_PARTITION语句，无需指定分区名。
    - 如果只剩一个分区，执行COALESCE_PARTITION语句会报错。
    - 如果被收缩的分区与其将要迁移至的分区的CSF属性不一致时会报错。
  - MODIFY_PARTITION partition_name COALESCE SUBPARTITION
  将某个父分区最后一个子分区的数据插入到前面某个分区中，再把最后一个子分区删除。
    - 只有当子分区类型为hash分区时才能执行COALESCE SUBPARTITION语句，需要指定需要执行收缩操作的父分区的名字，子分区名字无需指定。
    - 如果父分区下面只剩一个子分区，执行COALESCE SUBPARTITION语句会报错。

- split_partition_clause

  分裂分区，将指定的(子)分区分裂为两个(子)分区，原始分区的数据重分布到新的分区中。当前仅RANGE分区支持split操作。对于带CSF的分区来说，分类后的分区CSF属性于原始分区一致。
  - (sub)partition_name
  需要分裂的（子）分区名称。
  - (sub)part_name1 (sub)part_name2
  分裂后新的（子）分区名称，两个（子）分区名称不能重复。一级分区分裂后，只有最后一个新分区仍然可以使用原分区名称。
  - range_value
  分裂的边界值。
  - UPADATE GLOBAL INDEXES
      - 若指定update global indexes,则数据重分布会自动重建全局索引（如果有的话）。
      - 若不指定，则全局索引处于invalid状态。

- modify_partition_clause
修改分区的属性。
  - partition_name
     需要修改的分区名称。
  -INITRANS interger 
  修改分区的初始数据页面上事务槽的个数，取值范围时[1,255]。
    - 修改后的新值只对新分配的页面有效，对已经分配的老页面无效。
    - 修改分区的INITRANS属性，会同步修改该分区的所有子分区的INITRANS。
  - storage_alter_clause
  表的存储空间的最大值
    - UNLINITED
    表示不限制表存储空间的最大值。
    - interger[K|M|G|T]
    设置表的存储空间最大值，取值范围[1M,1T]。

- exchange_partiion_clause
  交换分区。

@说明：分区交换使用约束：
1.不支持涉及外键约束的子句，例如cascade。如果交换的两张表的任意表上有外键关系，则交换分区会报错。
2.支持windows validation 子句，默认会进行分区数据校验，加上子句不会校验数据[业务保证]，但是还是会检验表定义，但是带来的查询分区数据丢失等后果需要业务自己处理。
3.不支持RCRd的表和索引进行交换。
4.交换双方存在自增列时不允许交换。
5.交换双方的是否压缩、是否csf、是否nologging Insert 属性不相同不允许交换。
6.交换双方的表定义、索引定义、列定义完全相同时，才允许交换。其中列定义，如果普通表（分区表）删除或添加剂，对应的分区表（普通表）经过同样的步骤来删除或添加同样的列，才能保证双方列定义一致。

- WITH TABLE

  设置需要交换的普通表的表名。

- INCLUDING | EXCLUDING INDEXES 
   - INCLUDING INDEXES
   表示要交换索引 
   - EXLUDINGINDEXES
   表示不需要交换索引
- WITH|WITHOUT VALIDATION
   - WITH VALIDATION
   表示需要校验数据。
   - WITHOUTVALIDATION
   表示u不需要交换数据

- partition_clause
需要交换的分区的信息

   - partition_name
   需要交换的分区的信息
   - FOR（part_key value)
   分区名不易获取的情况下，可以设置该分区键值用于指示需要交换的分区。

- enable_disable_clause
启用或禁用约束，并指定启用或禁用约束时是否确保已有记录符合约束。
- ENABLE：启动约束
- DISABLE:禁用约束
- VALIDATE:启用或禁用约束时以及约束被启用或禁用后，确保已有数据是否符合约束。
- NOCALIDATE：启用或禁用约束时以及约束被启用或禁用后，不考虑已有数据是否符合约束。

使用VALIDATE或NOVALIDATE,对新增记录和所更新的记录没有影响。也就是说，无论是使用VALIDATE还是使用NOCVALIDATE,在使用ENABLE启用约束后都会检查新增记录和所更新的记录是否符合约束，但是在使用DISABLE禁用约束后则都不再检查新增记录和所更新的记录是否符合约束。

使用VALIDATE或NOVALIDATE,对新已有记录产生影响。也就是说，在启用或禁用约束时以及约束被启用或禁用后，如果使用VALIDATE,则会检查已有记录是否符合约束，如果不符合，则启用或禁用约束失败或新增修改记录时返回错误信息；使用NOVALIDATE，在启用或禁用约束时以及约束被启用或禁用后，则不会检查已有记录是否符合约束。

使用ENABLE启用约束时，如果不指定使用NOVALIDATE,则默认使用VALIDATE,效果等同于DISABLE NOVALIDATE，禁用约束，删除约束上的索引，且允许修改被约束的记录；指定VALIDATE,则禁用约束，删除约束字段上的索引，且不允许修改任何被约束的记录。

**表 5-20** 关键字组合说明

<a name="zh-cn_topic_0283136979"></a>
<table><thead align="left"><tr id="zh-cn_topic_0283136979"><th class="cellrowborder" valign="top" width="15.701570157015702%" id="mcps1.2.4.1.1"><p id="zh-cn_topic_0283136979"><a name="zh-cn_topic_0283136979"></a><a name="zh-cn_topic_0283136979"></a>关键字组合</p>
</th>
 <th class="cellrowborder" valign="top" width="32.753275327532755%" id="mcps1.2.4.1.2"><p id="zh-cn_topic_02831369798"><a name="zh-cn_topic_0283136979"></a><a name="zh-cn_topic_0283136979"></a>是否检查哟有记录符合约束</p>
</th>
<th class="cellrowborder" valign="top" width="51.54515451545154%" id="mcps1.2.4.1.3"><p id="zh-cn_topic_0283136979"><a name="zh-cn_topic_0283136979"></a><a name="zh-cn_topic_0283136979"></a>是否检查新增或修改记录符合约束</p>
</th>
</tr>
</thead>
<tbody>

<tr id="zh-cn_topic_0283136979">
<td class="cellrowborder" valign="top" width="30%" headers="mcps1.2.4.1.1 "><p id="zh-cn_topic_0283136979"><a name="zh-cn_topic_0283136979"></a><a name="zh-cn_topic_0283136979"></a>ENABLE VALIDATE</p></td>

<td class="cellrowborder" valign="top" width="32%" headers="mcps1.2.4.1.2 "><a name="zh-cn_topic_03"></a><a name="zh-cn_topic_03"></a><a id="zh-cn_topic_03"></a>yes</p></td>
<td class="cellrowborder" valign="top" width="51%" headers="mcps1.2.4.1.3 "><p id="zh-cn_topic_01"><a name="zh-cn_topic_01"></a><a name="zh-cn_topic_01"></a>yes</p></td>
</tr>

<tr>
    <td rowspan="1">ENABLE NOVALIDATE</td>
    <td>no</td>
    <td>yes</td>
</tr>

<tr>
    <td rowspan="1">DISABLE VALIDATE</td>
    <td>yes</td>
    <td>no</td>
</tr>

<tr>
    <td rowspan="1">DISABLE NOVALIDATE</td>
    <td>no</td>
    <td>no</td>
</tr>

</tbody>
</table>

- [PARITION | SUBPARTITION] NOLOGGING
  - NOLOGGING 
  启用或禁用分区上的NOLOGGING INSERT属性。
  - PARTITION NOLOGGING
  启用或禁用分区上的NOLOGGING INSERT属性。
  - SUBPARTITION NOLOGGING
  启用或禁用子分区上的NOLOGGING INSERT属性。
  - NOLOGGING INSERT 操作是为了提高大量数据的入库性能，在表或者分区上执行Nologging insert操作不要与其他正常业务并发，避免因为并发读取undo导致正常业务报错，因为Nologging insert不记录undo日志。
  - Nologging insert完成数据入库之后建议及时提交事务并手动触发数据刷盘，之后再开启其他业务，以保证数据的安全，避免因为掉电导致需要重新执行数据入库操作。
  - Nologging insert不记录undo数据，在完成数据导入后强烈建议及时提交之后在执行其他业务，避免其他业务因为访问不到历史数据而出错；同时，由于没有undo,因此如果一个事务内执行过Nologging insert，那么再执行rollback操作的时候，虽然操作能够成功，但是数据库内部不做任何数据修改动作。因此，执行过Nologging insert的事务不能通过rollback回退到历史版本。
  - 开启逻辑复制或者在主备环境下不允许执行Nologging insert。
  - 如果数据库中有表或者分区对象存在Nologging insert，则不允许动态添加备机。
  - 由于没有redo/undo，所以一旦发生任何异常，数据库不能继续保证数据一致性，因此需要删除表数据，重新执行导入操作。
  - 临时表不支持Nologging insert属性的设置，nologging表本身就带有不记录redo的属性，所以它不支持再设置nologging属性。
  - session级别的Nologging insert由内部工具使用，客户业务不能使用此语法，客户业务应该使用表分区级的nologging inseret替代session级的nologging insert。
  - 表上或者分区上原本有数据的情况下，不允许开启nologging insert。- 支持表分区级Nologging insert特性的版本不允许向不支持表分区级Nologging insert特性的版本降级，但可以升级；如果两个版本都支持Nologging insert特性，则可正常进行升级或降级操作（不考虑其他特性的影响）。升级或降级前需要用户手动确认数据库系统中是否存在Nologging对象，如果这些对象不再需要Nologging属性，建议关闭Nologging属性后再进行升级或降级操作。
  - 如果没有关闭对象的Nologging insert属性进行备份恢复操作，会造成nologging属性扩散到恢复的新环境上，用户需确保这个属性扩散是否需要，如不需要在备份前关闭对象的nologging属性。

表上的NOLOGGING INSERT属性与分区上的相互独立，某个分区是否具有 NOLOGGING INSERT属性与表上的此属性无关，只取决于自身的NOLOGGING INSERT属性；父子分区间的NOLOGGING属性具有关联性，如果父分区打开  NOLOGGING INSERT属性，那么他的所有子分区也打开该属性，但是反过来打开子分区的NOLOGGING INSERT 属性是否打开不影响父分区的此属性。

NOLOGGING INSERT 属性是一种特殊的操作，它是指在INSERT数据时不记录redo和undo日志，以提升INSERT的性能。由于此特性不记录redo和undo，请谨慎使用。该特性使用时，需要注意以下约束：

- set_interval_caluse
设置间隔分区，仅对分区表有效。
    - SET INTERVAL():将间隔分区表修改为范围分区表。
    - SET INTERVAL(interval_value):修改间隔分区表的间隔值。

- rename_cloumn_clause
  修改表名。只能修改自己schama下的表名，不能修改系统表空间下的表名。

- logic_replication_caluses
打开表逻辑复制开关或者关闭逻辑复制开关，支持表级和表分区级逻辑复制开关。
    - (sub)partition_name[,...]
   在表名之后括号内添加表（子）分区名即为设置表分区级逻辑复制开关，支持多次补充未打开表分区开关，不支持表级和表分区级直接切换。
   SYS用户不会加载数据字典，因此SYS用户不支持多次补充打开表分区逻辑复制开关。
    - ADD LOGICAL LOG(UNIQUE index_name)
   根据唯一索引打开逻辑复制开关。
      - index_name唯一索引名称。
    - ADD LOGICAL LOG(PRIMARY KEY)
    根据主键打开表逻辑复制开关。
    - DROP LOGICAL LOG
    关闭表级和分区级逻辑复制开关。

示例：

- 添加列
```
--删除表training
DROP TABLE IF EXISTS training;
--创建表training
CREATE TABLE training(staff_id INT NOT NULL, course_name VARCHAR（50)，course_start_date DATETIME, course_end_date DATETIME, score INT);
--添加列full_masks
ALTER TABLE training ADD full_score INT;
```
- 删除列
```
ALTER TABLE training DROP score;
```

- 修改列的数据类型
```
ALTER TABLE training MODIFY course_name VARCHAR(20);
```

- 添加约束
```
ALTER TABLE training ADD CONSTRANT ck_training CHECK(staff_id > 0);
ALTER TABLE training ADD CONSTRANT uk_training UNIQUE(course_name);
```
- 重命名约束
```
ALTER TABLE training RENAME CONSTRANT ck_training to ck_new_training;
ALTER TABLE training RENAME CONSTRANT uk_training to uk_new_training;
```    

- 删除约束
```
ALTER TABLE training DROP CONSTRANT uk_training;
```

- 重命名表
```
ALTER TABLE training RENAME TO training_2018;
```

- 删除分区training3和training4
```
--删除表training
DROP TABLE IF EXISTS training;
--创建分区表training
CREATE TABLE training(
  staff_id INT NOT NULL,
  course_name CHAR(20),
  course_period DATETIME,
  exam_date DATETIME,
  socre INT)
PARTITION BY RANGE(staff_id)
(
PARTITION training1 VALUES LESS THAN(100);
PARTITION training2 VALUES LESS THAN(200);
PARTITION training3 VALUES LESS THAN(300);
PARTITION training4 VALUES LESS THAN(400);
);
--删除分区training3
ALTER TABLE training DROP PARTITION training3;
--清空分区training4
ALTER TABLE training TRUNCATE PARTITION training4;
```

- 添加分区training5和training6
```
ALTER TABLE training ADD PARTITION training5 VALUES LESS THAN(450);
ALTER TABLE training ADD PARTITION training6 VALUES LESS THAN(MAXVALUE);
```

- 分裂分区
```
ALTER TABLE training SPILIT PARTITION training5 AT(420) INTO (PARITION p1, PARTITION p2);
```

- 添加联合主键约束
```
--删除表
DROP TABLE IF EXISTS training_base;
--创建表
CREATE TABLE training_base(
  id int;
  class varchar(8);
  name varchar2(8);
  gender boolean,
  score number(10,5)
  );
--添加联合主键约束
ALTER TABLE training_base ADD CONSTRANT pk_tbl_base PRIMARY KEY (id,name);
```

- 打开或关闭表级逻辑复制开关
```
--删除表
DROP TABLE IF EXISTS logic_test;
--创建表
CREATE TABLE logic_test(
  staff_id INT PRIMARY KEY,
  course_name VARCHAR(50),
  course_period DATETIME,
  exam_date DATETIME,
  socre INT)
PARTITION BY RANGE(staff_id)(
PARTITION p1 VALUES LESS THAN(10);
PARTITION p2 VALUES LESS THAN(20);
PARTITION p3 VALUES LESS THAN(30);
PARTITION p4 VALUES LESS THAN(400;
);
-- 打开表级逻辑复制开关
ALTER TABLE logic_test ADD LOGICAL LOG(PRIMARY KEY);
-- 关闭表级逻辑复制开关
ALTER TABLE logic_test DROP LOGICAL LOG;
```

- 打开或关闭表分区级逻辑复制开关
```
--删除表
DROP TABLE IF EXISTS logic_test;
--创建表
CREATE TABLE logic_test(
  staff_id INT PRIMARY KEY,
  course_name VARCHAR(50),
  course_period DATETIME,
  exam_date DATETIME,
  socre INT)
PARTITION BY RANGE(staff_id)(
PARTITION p1 VALUES LESS THAN(10);
PARTITION p2 VALUES LESS THAN(20);
PARTITION p3 VALUES LESS THAN(30);
PARTITION p4 VALUES LESS THAN(400;
);
-- 打开表分区级逻辑复制开关
ALTER TABLE logic_test(p1,p2) ADD LOGICAL LOG(PRIMARY KEY);
ALTER TABLE logic_test(p3,p4) ADD LOGICAL LOG(PRIMARY KEY);
-- 删除表分区的同时会删除表分区级逻辑复制开关
ALTER TABLE logic_test DROP PARITION p3;
-- 关闭表级逻辑复制开关
ALTER TABLE logic_test DROP LOGICAL LOG;
```

- 修改表的分区的MAXSIZE值
```
--删除表
DROP TABLE IF EXISTS storage_test;
--创建表
CREATE TABLE storage_test(c_id INT, c_char VARCHAR2(5000)) STORAGE (MAXSIZE 3M INITIAL 1M)
PARTITION BY RANGE(c_id)(
PARTITION p1 VALUES LESS THAN(2) STORAGE (MAXSIZE 3M INITIAL 2M);
PARTITION p2 VALUES LESS THAN(3);
);
--修改表的存储空间的MAXSIZE值
ALTER TABLE storage_test STORAGE (MAXSIZE 3M)；
--添加分区，初始大小是2M最小值是3M
ALTER TABLE storage_test ADD PARTITION p3 VALUES LESS THAN(34) STORAGE (MAXSIZE 3M INITIAL 2M);
--修改分区p1的最大值为2M
ALTER TABLE storage_test MODIFY PARTITION p1 STORAGE (MAXSIZE 2M);
```

- 修改表及分区的INITRANS值
```
--删除表
DROP TABLE IF EXISTS initrans_test;
--创建表
CREATE TABLE initrans_test(c_id INT, c_char VARCHAR2(5000)) INITRANS 2
PARTITION BY RANGE(c_id)(
PARTITION p1 VALUES LESS THAN(2) INITRANS 5;
PARTITION p2 VALUES LESS THAN(3);
);

--修改表的INITRANS值
ALTER TABLE initrans_test INITRANS 10;
--修改表的分区的INITRANS值
ALTER TABLE initrans_test MODIFY PARTITION p1 INITRANS 20;
```





