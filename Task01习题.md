![](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210401100653725.png)

```sql
create table Addressbook 
	(regist_no integer not null,
     name varchar(128) not null,
     address varchar(256) not null,
     tel_no char(10),
     mail_address char(20),
     primary key(regist_no));
```

- 假设在创建练习1.1中的 Addressbook 表时忘记添加如下一列 postal_code （邮政编码）了，请把此列添加到 Addressbook 表中。

    列名 ： postal_code

    数据类型 ：定长字符串类型（长度为 8）

    约束 ：不能为 NULL

    ```sql
    alter table Addressbook add column pstal_code char(8) not null;
    ```

- 编写 SQL 语句来删除 Addressbook 表。

```sql
drop table Addressbook;
```

- 编写 SQL 语句来恢复删除掉的 Addressbook 表。

```sql
删除的表无法恢复
```

