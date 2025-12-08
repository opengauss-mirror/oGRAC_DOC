# COMMENT ON

#### 功能描述

使用comment语句在字典中增加一个关于表、视图或列的注释信息。

通过MY_TAB_COMMENTS、ADM_TAB_COMMENTS或MY_COL_COMMENTS、ADM_COL_COMMENTS系统视图可以查看注释信息。

#### 注意事项

- 为自己的表增加注释时，不需要权限；为任意用户的表增加注释时，需要执行语句的用户拥有COMMENT ANY TABLE权限。
- 支持创建表时，指定列的comment信息。
- 数据库重启回滚期间不支持该操作。

#### 语法格式

COMMENT ON { TABLE [ schema_name. ] { table_name | view_name } 

    | COLUMN [ schema_name. ] { table_name. | view_name. } column_name 

    } IS 'string'

#### 参数说明

- [schema_name. ]

用户名。不指定时默认是当前登录用户。

- { table_name | view_name }

指定注释的表或视图名。

- [schema_name.] {table_name. | view_name. } column_name

指定注释的列名。

- IS

指定注释内容。

- string

注释内容。

最大长度4000字节。

#### 示例

--删除表training。 

DROP TABLE IF EXISTS training;

--创建表training。 

CREATE TABLE training(staff_id INT NOT NULL, course_name VARCHAR(50), course_start_date DATETIME, course_end_date DATETIME, exam_date DATETIME, score INT);

--给表training添加注释。 

COMMENT ON TABLE training IS 'table of training courses';

--给列staff_id添加注释。 

COMMENT ON COLUMN training.staff_id IS 'id of staffs taking training courses';
