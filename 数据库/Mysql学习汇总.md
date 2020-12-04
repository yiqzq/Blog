[toc]

## 事务的四大特性ACID

**原子性(Atomicity)：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

**一致性(Consistency)：** 执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；

**隔离性(Isolation)：** 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；

**持久性(Durability)：** 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

## 并发事务产生的问题

- 脏读

- 不可重复读

- 幻读

具体可以参考我的另外一篇[博客](https://blog.csdn.net/yiqzq/article/details/104533487)

## 数据库事物的四种隔离级别

- Read Uncommitted（读未提交）

  在该隔离级别，所有事务都可以看到其他未提交事务的执行结果，会产生脏读的现象。

- Read Committed（读已提交）

  只允许事务读取已经被其他事务提交的变更，虽然避免了脏读，但是会产生不可重复读。

- Repeatable Read（可重读），**这是mysql的默认隔离级别**

  确保事务可以多次从一个字段中读取相同的值，在事务期间，不允许其他事务对这个字段进行修改。可以避免脏读和不可重复读，但是会有幻读问题。

- Serializable（可串行化）

  确保事务可以从一个表中读取相同的行，在事务期间，禁止其他事务对对该表执行删除，修改，添加操作。可以避免所有问题，但是性能底下。

## 存储引擎

### 查看本地mysql的存储引擎

使用指令`show engines;`，下图是我本地mysql的情况	

![image-20200315123646943](https://i.loli.net/2020/03/15/O9UvafMyG6ZJpXw.png)

关于上图几个参数的解释

Engine参数指存储引擎名称；
Support参数说明MySQL是否支持该类引擎，YES表示支持；
Comment参数指对该引擎的评论；
Transactions 参数表示是否支持事务处理，YES表示支持；
XA参数表示是否分布式交易处理XA规范，YES表示支持；
Savepoints参数表示是否支持保存点，以便事务回滚到保存点，YES表示支持

**一般常用的就两种，InnoDB和MyISAM**

**MyISAM** ：默认表类型，它是基于传统的ISAM类型，ISAM是Indexed Sequential Access Method (有索引的顺序访问方法) 的缩写，它是存储记录和文件的标准方法。不是事务安全的，而且不支持外键，如果执行大量的select，insert MyISAM比较适合。

**InnoDB** ：支持事务安全的引擎，支持外键、行锁、事务是他的最大特点。如果有大量的update和insert，建议使用InnoDB，特别是针对多个并发和QPS较高的情况。**注：** 在MySQL 5.5之前的版本中，默认的搜索引擎是MyISAM，从MySQL 5.5之后的版本中，默认的搜索引擎变更为InnoDB。

**MyISAM和InnoDB的区别：**

1. InnoDB**支持事务**，MyISAM不支持。
   对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务；
2. InnoDB支持**外键**，而MyISAM不支持。
3. InnoDB是**聚集索引**，使用B+Tree作为索引结构，数据文件是和（主键）索引绑在一起的（表数据文件本身就是按B+Tree组织的一个索引结构），必须要有主键，通过主键索引效率很高。MyISAM是非聚集索引，也是使用B+Tree作为索引结构，索引和数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
4. InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快。
5. InnoDB不支持全文索引，而MyISAM支持全文索引，查询效率上MyISAM要高；5.7以后的InnoDB支持全文索引了。
6. InnoDB支持**表、行级锁**(默认)，而MyISAM支持表级锁。
7. InnoDB表必须有**主键**（用户没有指定的话会自己找或生产一个主键），而MyISAM可以没有。
8. InnoDB存储文件有frm、ibd，而MyISAM是frm、MYD、MYI。

InnoDB：frm是表定义文件，ibd是数据文件，MyISAM：frm是表定义文件，MYD是数据文件，MYI是索引文件。

## InnoDB记录存储结构

[大神博客](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483670&idx=1&sn=751d84d0ce50d64934d636014abe2023&chksm=979688e4a0e101f2a51d1f06ec75e25c56f8936321ae43badc2fe9fc1257b4dc1c24223699de&scene=21#wechat_redirect)

InnoDB采用的是分页模式，将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB中页的大小一般为 **16KB**。也就是在一般情况下，一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。

我们平时是以记录为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为`行格式`。现在基本上存在4中行格式，分别是`Compact`、`Redundant`、`Dynamic`和`Compressed`，就拿我现在使用的数据库为例，使用`Dynamic`作为行格式。、

![image-20200315154129495](https://i.loli.net/2020/03/15/fD5QEaClPvgji2Y.png)

### Compact行格式

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey37MIoP0YPpYdY7Y0TO0iaZ3a79QB7GiaHyTqJTicNBQ6Nk202h4JECicib3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以看到，我们存进去的每一条记录的信息并不是那么简单，除了存进去的真实数据之外，mysql还会往记录中添加一些额外的信息。

#### 变长字段长度列表

针对`VARCHAR(M)`这类的可变数据类型，我们必须知道所占的长度，不然在记录的真实数据我们怎么知道哪些数据是属于自己的呢。

**注意的是，这里只存储变长且为非null的字段**

下面是大神的例子，创建一个表，并添加数据

![image-20200315155038062](https://i.loli.net/2020/03/15/BKRDW9kHvSLVUts.png)

![image-20200315155109496](https://i.loli.net/2020/03/15/KeZH5C6YyvislDu.png)

那么对可变长数据分析如下

| 列名 | 存储内容 | 内容长度（十进制表示） | 内容长度（十六进制表示） |
| :--: | :------: | :--------------------: | :----------------------: |
| `c1` | `'aaaa'` |          `4`           |          `0x04`          |
| `c2` | `'bbb'`  |          `3`           |          `0x03`          |
| `c4` |  `'d'`   |          `1`           |          `0x01`          |

最后存储的就是内容长度，因为要求必须逆序放置，所以`040301`就要变成`010304 `

另外原文中还有关于是使用1个字节还有2个字节比埃是内容长度的篇幅，可以区阅读。

#### NULL值列表

为了优化空间，所以设立NULL值列表以表示一条记录中哪些字段是null。

**注意的是，这里只统计所有列的值允许为NULL的情况**

还是刚才表的例子，有c1,c3,c4的列是允许为null的，前面添0是因为长度要是以字节为单位，而且还是和前面一样需要**逆序**存储

<img src="https://i.loli.net/2020/03/15/rVbKGMn5I9SY8zW.png" style="zoom:67%;" />

<img src="https://i.loli.net/2020/03/15/VOIGQXYbsynAxk3.png" alt="image-20200315160641451" style="zoom:67%;" />

#### 记录头信息

存储一些记录数据需要的内容，更多细节可以查看原文

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3seeEXfe32gnNpyFDpGADNaia0ytgBf2l35mLFECb1jI4HJmEoFJAOmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 记录的真实数据

除了我们插入的列数据以外，mysql还会插入其他的内容，用于事务的处理

|       列名       | 是否必须 | 占用空间 |                          描述                           |
| :--------------: | :------: | :------: | :-----------------------------------------------------: |
|     `row_id`     |    否    | `6`字节  | 行ID，唯一标识一条记录，<br/>只有当不设置主键时才会创立 |
| `transaction_id` |    是    | `6`字节  |                         事务ID                          |
|  `roll_pointer`  |    是    | `7`字节  |                        回滚指针                         |

**完整的数据**

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3b0m7SgCx0hOqzK7A66ARzntr7vXLCoAzpVEZNdNdvrPG8nOdSEq13w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

还有一点要注意的是非可变长度类型，比如CHAR(10)等等，这个字段如果为null，那么就会出现在NULL值列表中，如果不为空，那么如果是10字节，那么就一定会占10字节空间，空位补空格（0x20）。

### Redundant行格式

**整体结构**

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3sTj2WHp0xqwYqgadiblaPIXIazt08Hia6Ns8qzlhkDN2Tr5HV6gdvEIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同样的两条数据的显示，更多比较请查看原文

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3JcV1lT0uU8AXCIL3CblsJkJMc1NxmVjwCgWQLb0CV2XmD3B6NnkibRA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Dynamic行格式

#### 关于行溢出

我们知道InnoDB按照页存储数据，而页的一般是16KB，也就是16384字节，但是mysql中一条数据存储的内容很可能高达60000+个字节，（比如一个`VARCHAR(M)`类型的列就最多可以存储`65532`个字节），那么就会出现在一页中无法存下一条数据的问题。 那么对于`Compact`和`Reduntant`这两种行格式，**采用的策略类似于链表，就是每一页只存储部分数据，然后指定一个地址，表示接下来的内容到这个地址去取。**

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3ib7Tq6vJ7UupNM8xYcOJ3eibT7ib9RczQ9b7ibtESicfqgA2HvhPvugd2Ww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而Dynamic这种格式，就不会在记录的真实数据处存储字符串的前`768`个字节，而是把所有的字节都存储到其他页面中，只在记录的真实数据处存储其他页面的地址。

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3NnuO9jdPFbrtMLa6lvWDZKQxtLVlv9tmR6jhFGYEhFXce57Yt6EloQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Compressed行格式

`Compressed`行格式和`Dynamic`不同的一点是，`Compressed`行格式会把存储到其他页面的数据采用压缩算法进行压缩，以节省空间。

## InnoDB 的页结构

下图是InnoDB的基础页结构，我们可以看到大致分为了七类。

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FusCptW3JwBoBchTsR6PJ5YoHfS02chsR4bSERrblH0ousvlric9BV3QcGrCClQ7qibWVWWhx5hQaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

|         名称         |       中文名       | 占用空间大小 |       简单描述       |
| :------------------: | :----------------: | :----------: | :------------------: |
|    `File Header`     |       文件头       |   `38`字节   |   一些描述页的信息   |
|    `Page Header`     |        页头        |   `56`字节   |     页的状态信息     |
| `Infimum + Supremum` | 最小记录和最大记录 |   `26`字节   |   两个虚拟的行记录   |
|    `User Records`    |      用户记录      |    不确定    | 实际存储的行记录内容 |
|     `Free Space`     |      空闲空间      |    不确定    |  页中尚未使用的空间  |
|   `Page Directory`   |       页目录       |    不确定    |  页中的记录相对位置  |
|    `File Tailer`     |      文件结尾      |   `8`字节    |    校验页是否完整    |

### Free Space和User Records

Free Space：顾名思义，这个就是空闲空间。
User Records：就是指用户使用的空间，也就是我们上节所说的每一条数据都是存放在这里。

在最开始的时候，User Records里面是没有任何内容的，当要添加一条记录，就会从Free Space中申请一份空间，划到User Records中存放数据。

<img src="https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FusCptW3JwBoBchTsR6PJ5xibbpE0yjbGYUwszT98icQbzOJibia0aufvOZV4FR6xfJ1HF0jBnmr8fGA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:67%;" />



#### 关于请求头

这个是上节介绍行格式所没有提到的，下面详细分析一下

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FVTOrSL5huibzeawNtia5ey3seeEXfe32gnNpyFDpGADNaia0ytgBf2l35mLFECb1jI4HJmEoFJAOmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



|      名称      | 大小（单位：bit） |                             描述                             |
| :------------: | :---------------: | :----------------------------------------------------------: |
|   `预留位1`    |        `1`        |                           没有使用                           |
|   `预留位2`    |        `1`        |                           没有使用                           |
| `delete_mask`  |        `1`        |                     标记该记录是否被删除                     |
| `min_rec_mask` |        `1`        |         标记该记录是否为B+树的非叶子节点中的最小记录         |
|   `n_owned`    |        `4`        |                    表示当前槽管理的记录数                    |
|   `heap_no`    |       `13`        |                表示当前记录在记录堆的位置信息                |
| `record_type`  |        `3`        | 表示当前记录的类型，`0`表示普通记录，`1`表示B+树非叶节点记录，`2`表示最小记录，`3`表示最大记录 |
| `next_record`  |       `16`        |                   表示下一条记录的相对位置                   |

举一个例子，更多参考[这里](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483678&idx=1&sn=913780d42e7a81fd3f9b747da4fba8ec&chksm=979688eca0e101fa0913c3d2e6107dfa3a6c151a075c8d68ab3f44c7c364d9510f9e1179d94d&scene=21#wechat_redirect)

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FusCptW3JwBoBchTsR6PJ5oWkKqKo9H1FGZQHIr1Hv7mqP2MMeAnzctpxhAdmQaMS2sONwoKZKrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### delete_mask

这个属性标记着当前记录是否被删除，占用1个二进制位，值为`0`的时候代表记录并没有被删除，为`1`的时候代表记录被删除掉了。**需要注意的是，标记了并不会在内存中删除这个记录，而是会保留，这条记录的内容将来会被新**
**内容所替换。**

##### min_rec_mask

关于B+树的内容，以后再说

##### n_owned

下面用于分组，加快主键查询

##### heap_no

这个属性表示当前记录在本页中的位置，从图中可以看出来，我们插入的4条记录在本`页`中的位置分别是：`2`、`3`、`4`、`5`,其中`0`和`1`的记录是InnoDB自己插入的，就是七大组成部分中的`Infimum + Supremum`，表示两个虚拟行。

`Infimum + Supremum`表示最小记录和最大记录，其中比较依赖于主键。

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FusCptW3JwBoBchTsR6PJ5Vm1GdicsErque1N7BU3KFiaSfbVz63DOpBRdLZ0TxwWg2jKLnjI0Z2YQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### record_type

这个属性表示当前记录的类型，一共有4种类型的记录，`0`表示普通记录，`1`表示B+树非叶节点记录，`2`表示最小记录，`3`表示最大记录。而我们插入的一般都是0号，表示普通记录。

##### next_record

表示从当前记录的真实数据到下一条记录的真实数据的地址偏移量，可以理解为是链表，每次指向下一条数据的位置。**其中下一条是按照主键排序的，而不是按照插入顺序。**

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FusCptW3JwBoBchTsR6PJ5iacSkLYEDU9kp03KlvuQs8BhqxU7y6G4ebUtm2frHjQ5m7QTfzuIPGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Page Directory

**这个作用主要是加快页内查询速度。**

首先我们知道在同一页中的数据是按照主键大小排序的，这也就一意味着如果我们按照主键搜索，是不需要遍历，可以通过二分搜索来优化查询的速度。

那么InnoDB就按照下面的策略来实施二分搜索加快查询速度

1. 将所有正常的记录（包括最大和最小记录，不包括标记为已删除的记录）划分为几个组。
2. 每个组的最后一条记录的头信息中的`n_owned`属性表示该组内共有几条记录。
3. 将每个组的最后一条记录的地址偏移量按顺序存储起来，每个地址偏移量也被称为一个`槽`（英文名：`Slot`）。页中存储地址偏移量的部分也被称为`Page Directory`。

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55FusCptW3JwBoBchTsR6PJ5QicCquOS6FndV5ISAgfTkjD4AuaayCXsia2cZOIdjtF50ZuFQpGKjaAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

需要注意的是，对于最小记录所在的分组只能有 **1** 条记录，最大记录所在的分组拥有的记录条数只能在 **1~8** 条之间，剩下的分组中记录的条数范围只能在是 **4~8** 条之间。

这个限制由下面的流程实现

- 初始情况下一个数据页里只有最小记录和最大记录两条记录，它们分属于两个分组。
- 之后每插入一跳记录都把这条记录放到最大记录所在的组，直到最大记录所在组中的记录数等于8个。
- 在最大记录所在组中的记录数等于8个的时候再插入一条记录时，将最大记录所在组平均分裂成2个组，然后最大记录所在的组就剩下4条记录了，然后就可以把即将插入的那条记录放到该组中了。

### Page Header

|        名称        | 占用空间大小 |                             描述                             |
| :----------------: | :----------: | :----------------------------------------------------------: |
| `PAGE_N_DIR_SLOTS` |   `2`字节    |                      在页目录中的槽数量                      |
|  `PAGE_HEAP_TOP`   |   `2`字节    |                       第一个记录的地址                       |
|   `PAGE_N_HEAP`    |   `2`字节    | 本页中的记录的数量（包括最小和最大记录以及标记为删除的记录） |
|    `PAGE_FREE`     |   `2`字节    |       指向可重用空间的地址（就是标记为删除的记录地址）       |
|   `PAGE_GARBAGE`   |   `2`字节    |  已删除的字节数，行记录结构中`delete_flag`为1的记录大小总数  |
| `PAGE_LAST_INSERT` |   `2`字节    |                      最后插入记录的位置                      |
|  `PAGE_DIRECTION`  |   `2`字节    |                        最后插入的方向                        |
| `PAGE_N_DIRECTION` |   `2`字节    |                  一个方向连续插入的记录数量                  |
|   `PAGE_N_RECS`    |   `2`字节    | 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录） |
| `PAGE_MAX_TRX_ID`  |   `8`字节    |        修改当前页的最大事务ID，该值仅在二级索引中定义        |
|    `PAGE_LEVEL`    |   `2`字节    |                 当前页在索引树中的位置，高度                 |
|  `PAGE_INDEX_ID`   |   `8`字节    |                索引ID，表示当前页属于哪个索引                |
|     `PAGE_BTR`     |   `10`字节   |     非叶节点所在段的segment header，仅在B+树的Root页定义     |
|    `PAGE_LEVEL`    |   `10`字节   |       B+树所在段的segment header，仅在B+树的Root页定义       |

### File Header

`File Header`主要描述的就是`页`外的各种状态信息

|                名称                | 占用空间大小 |                             描述                             |
| :--------------------------------: | :----------: | :----------------------------------------------------------: |
|     `FIL_PAGE_SPACE_OR_CHKSUM`     |   `4`字节    |                   页的校验和（checksum值）                   |
|         `FIL_PAGE_OFFSET`          |   `4`字节    |                             页号                             |
|          `FIL_PAGE_PREV`           |   `4`字节    |                        上一个页的页号                        |
|          `FIL_PAGE_NEXT`           |   `4`字节    |                        下一个页的页号                        |
|           `FIL_PAGE_LSN`           |   `8`字节    |  最后被修改的日志序列位置（英文名是：Log Sequence Number）   |
|          `FIL_PAGE_TYPE`           |   `2`字节    |                          该页的类型                          |
|     `FIL_PAGE_FILE_FLUSH_LSN`      |   `8`字节    | 仅在系统表空间的一个页中定义，代表文件至少被更新到了该LSN值，独立表空间中都是0 |
| `FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID` |   `4`字节    |                       页属于哪个表空间                       |

### File Tailer

主要是校验页是否完整

- 前四个字节代表页的校验和

  这个部分是和`File Header`中的校验和相对应的。每当一个页面在内存中修改了，在同步之前就要把它的校验和算出来，因为`File Header`在页面的前边，所以校验和会被首先同步到磁盘，当完全写完时，校验和也会被写到页的尾部，如果完全同步成功，则页的首部和尾部的校验和应该是一致的，反之意味着同步中间出了错。

- 后四个字节代表日志序列位置（LSN）

## 索引

参考[博客一](https://juejin.im/post/5b55b842f265da0f9e589e79#comment)

参考[博客二](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483701&idx=1&sn=bd229dd584f51ef4fe545d44ad8cdbf9&chksm=979688c7a0e101d1b5c752094013b78f5bd50ab905257ba82149d85d35ea07aba1a15b0e52b4&mpshare=1&scene=23&srcid=0315TujtzSrYgouQdmoMRtSI&sharer_sharetime=1584270502302&sharer_shareid=c9f7f7e98f2aa90498c7672357e3e632#rd)

### 关于mysql底层为什么使用B+树

一般使用的树有以下类型

1. 二叉查找树
2. 平衡二叉树
3. 红黑树
4. B树
5. B+树

二叉查找树，时间复杂度明显不行，在极端情况下会退化

平衡二叉树，查找性能很好，但是在节点插入和删除的极端数据下，时间复杂度也不理想，会造成log次旋转。

红黑树，优化了插入和删除的性能，稍微降级了一些查询性能，统计性能较好。但是，我们知道mysql数据库的内容是存在硬盘中的，我们需要将内容读到内存中才能方便操作。二叉树的查询时$$log_2$$级别的，换句话说，如果红黑树有x层，那么就必须IO操作x次，这大大限制查询性能，因此也不可取。

B树，本质上是一棵多叉树，且叉数>=2,那么也就是说，一般情况下， B树的高度会远小于红黑树，减少了IO次数，提高了查询速度。

B+树，相较于B树，拥有比B树更低的高度，也就意味着更少的IO次数，更快的查询速度。且B+树拥有范围查询的功能，而这是B树所没有的，同时B+树的查询的稳定性也更高，一般稳定IO次数是B+树的高度。

**B树结构**

<img src="https://i.loli.net/2020/07/17/qpe8R7lS9V3xnur.png" alt="image-20200717103209320" style="zoom:80%;" />

**B+树和B树的比较**

1. B+树的层级更少，因为B+树的非叶节点只会存储主键和对应的页号，而B树的非叶节点会存储具体的内容，这也就是说，每个节点存储的记录个数比B树多很多，那么就能够使得B+树的高度更低。
2. B+更适于范围查询，在B树中进行范围查询时，首先找到要查找的下限，然后对B树进行中序遍历，直到找到查找的上限；而B+树的范围查询，只需要对双向链表进行遍历即可。(**主要原因**)
3. 更稳定的查询效率，B树的查询时间复杂度在1到树高之间(分别对应记录在根节点和叶节点)，而B+树的查询复杂度则稳定为树高，因为所有数据都在叶节点。

放一张B+树的结构图

![img](https://mmbiz.qpic.cn/mmbiz_png/RLmbWWew55F3opxnpoB8oddtgyoia7Yt55ga3Fmqa9Hicn9CQ6Lh2ibT7chylHXd6C6RdrbmZdmFzZJiafyoyjyIjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 索引的几种类型

**聚簇索引**

将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据

每个非叶子节点保存的是主键+页号，而每个叶子节点存放所有的数据

**一个聚簇索引的小问题：**

因为每个InnoDB存储引擎表都有一个聚簇索引，所以在默认有主键的情况下，那么这个聚簇索引就是主键索引，但是在没有主键的情况下，会进行如下操作

1. 首先判断表中是否有非空的唯一索引,如果有,则该列即为主键

2. 如果不符合上述条件,InnoDB存储引擎自动创建一个6字节大小的指针ROW_ID

现在的问题就在于这个row_id，这个row_id的值是一个全局计数器，这就意味着所有创建row_id的表，他们的插入性能就会受到影响，因为这个计数器是要线程安全的，以保证主键唯一，所以必然这些表在插入的时候是不能并发插入的，所以性能会受到影响。

**二级索引**（也叫非聚簇索引）

聚簇索引是按照主键排序的，那么如果不想要按照主键排序，可以使用二级索引，二级索引简单来说就是按照某一个非主键排序。

每个非叶子节点（也就是目录项）存放的是对应的非主键+页号，而叶子节点存放的是主键+对应的非主键。

所以我们如果要利用二级索引获得完整的数据，还必须再拿获得的主键去查询聚簇索引。

**联合索引**

相较于二级索引，排序的关键字不再只有一个，允许可以有多个关键字，就相当于是有主关键字和副关键字一样，如果主关键字不同就按照主关键字排序，如果相同，就以副关键字排序。

**哈希索引**

![img](https://images2015.cnblogs.com/blog/99941/201607/99941-20160706162359874-1132773212.jpg)

哈希索引就是利用哈希算法来创建索引。

那么特点也就很明显，单点查询快，只需要经过一次哈希计算，就能到到对应数据的位置。

缺点也很明显

1. 哈希算法的通病，存在哈希冲突
2. 不能范围查询，因为哈希算法无法保留原有的数据大小关系
3. 不支持B+树索引的最左匹配原则
4. 没有排序功能

**覆盖索引**

覆盖索引是select的数据列只用从索引中就能够取得，不必读取数据行。可以理解为一种思想，就是**查询列表里只包含索引列**。

举个例子，假设我们创建一个表

```sql
CREATE TABLE `tx`.`student` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  `birth` DATE NOT NULL,
  `gender` CHAR(2) NOT NULL,
  `location` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`));
```

那么我们下面的操作可以称之为覆盖索引

1. 先建立name,birth,gender的联合索引
2. `SELECT name,birth,gender from student where  name='aaa'`使用这样的查询语句



### InnoDB和MyISAM使用索引的区别

**InnoDB**一般使用的是聚簇索引+非聚簇索引，既可以通过主键直接查找数据，也可以通过非聚簇索引查找主键，再通过主键查找的方式查询。

**MyISAM**的使用的是非聚簇索引，所以对于MyISAM而言，建立按照主键排序的索引还是辅助键排序的索引在本质上来说都没有区别，因为MyISAM维护了一张按照插入顺序的表，这里面存放了所有的数据，且每一条数据都有对应的行号。那么MyISAM就是通过索引找到对应数据的行号，然后通过行号再去查找所有数据的。

### B+树索引适用的条件

详情参考[这里](https://mp.weixin.qq.com/s?__biz=MzIxNTQ3NDMzMw==&mid=2247483718&idx=1&sn=4681f6ef312774f4a0a5f6bfb06c862a&chksm=979688b4a0e101a2d182cb37861d6b74ccbe61c1df9effba8da68e9c701701d20872d50429aa&mpshare=1&scene=23&srcid=0315rRH1TE8y2D2Lf9ODq2Os&sharer_sharetime=1584270554243&sharer_shareid=c9f7f7e98f2aa90498c7672357e3e632#rd)

- 全值匹配
- 匹配左边的列
- 匹配范围值
- 精确匹配某一列并范围匹配另外一列
- 用于排序
- 用于分组

### 建立索引需要注意的事项

- 只为用于搜索、排序或分组的列创建索引

- 为列的基数大的列创建索引

- 索引列的类型尽量小

- 可以只对字符串值的前缀建立索引

- 只有索引列在比较表达式中单独出现才可以适用索引

- 为了尽可能少的让`聚簇索引`发生页面分裂和记录移位的情况，建议让主键拥有`AUTO_INCREMENT`属性。

  具体可以参考[InnoDB如果没有主键或者随机主键真的很可怕吗？](https://www.javazhiyin.com/29434.html),这篇文章中的数据

- 定位并删除表中的重复和冗余索引

- 尽量适用`覆盖索引`进行查询，避免`回表`带来的性能损耗。

### 一些关于索引的面试问题的个人见解

#### 索引为什么能够提高查询速度？

索引底层使用的是B+树优化，能够尽可能的减少磁盘IO次数，节省时间，而且在查询的时候，由于数据是有序的，因此可以使用二分查找。这样子就能够加快查询速度。如果没有使用索引，则需要遍历双向链表，那么这个操作是非常耗时的。

#### 索引降低增删改的速度

这个很明显，索引底层是B+树，B+树实际上是平衡树，既然是平衡树就需要额外的时间消耗去维护平衡

## 索引最左匹配原则

对于一个联合索引，我们当用它去匹配的时候，总是会匹配最左边的字段，然后相同就会向后在比较一个字段，直到遇到范围查询(>、<、between、like)就停止匹配。

比如`a = 1 and b = 2 and c > 3 and d = 4` 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，只能够以线性的速度查询。但是需要注意的是如果查询语句变成`b = 2 and a = 1  and c > 3 and d = 4 `还是能够用到a,b的索引。

**可以这个样子原因在于mysql内部有查询优化器，可以对你的查询顺序进行内部的修改，以达到最大程度利用索引的效果。**

## 锁

### 锁的分类

- 表锁
  - 开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低
- 行锁
  - 开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高



- **InnoDB行锁和表锁都支持**！
- **MyISAM只支持表锁**！

InnoDB只有通过**索引条件**检索数据**才使用行级锁**，否则，InnoDB将使用**表锁**。也就是说，**InnoDB的行锁是基于索引的**！

### 行锁的两种锁

**共享锁（S锁，读锁）**：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。是**共享**的，多个客户可以**同时读取同一个**资源，但**不允许其他客户修改**。

**排他锁（X锁，写锁)：**允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。是排他的，**会阻塞其他的写锁和读锁**。

另外，**为了允许行锁和表锁共存，实现多粒度锁机制**，InnoDB还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是**表锁**：

- 意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
- 意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁

### MVCC

参考博客：[一文理解Mysql MVCC](https://zhuanlan.zhihu.com/p/66791480)

#### MVCC的产生原因

数据库通常使用锁来实现隔离性。最原生的锁，锁住一个资源后会禁止其他任何线程访问同一个资源。但是很多应用的一个特点都是读多写少的场景，很多数据的读取次数远大于修改的次数，而读取数据间互相排斥显得不是很必要。所以就使用了一种读写锁的方法，读锁和读锁之间不互斥，而写锁和写锁、读锁都互斥。这样就很大提升了系统的并发能力。之后人们发现并发读还是不够，又提出了能不能让读写之间也不冲突的方法，就是读取数据时通过一种类似快照的方式将数据保存下来，这样读锁就和写锁不冲突了,于是就有了MVCC.

#### MVCC的介绍

MVCC指的是多版本并发控制，MVCC是读已提交和可重复读取隔离级别的实现。

**MVCC只在 READ COMMITTED 和 REPEATABLE READ 两个隔离级别下工作。**其他两个隔离级别够和MVCC不兼容, 因为 READ UNCOMMITTED 总是读取最新的数据行, 而不是符合当前事务版本的数据行。而 SERIALIZABLE 则会对所有读取的行都加锁。

**InnoDB中通过undo log实现了数据的多版本，而并发控制通过锁来实现。**

#### 关于redo log，undo log，bin log

**bin log**：是mysql服务层产生的日志，常用来进行数据恢复、数据库复制，常见的mysql主从架构，就是采用slave同步master的binlog实现的, 另外通过解析binlog能够实现mysql到其他数据源（如ElasticSearch)的数据复制。

**redo log：**记录了数据操作在物理层面的修改，mysql中使用了大量缓存，缓存存在于内存中，修改操作时会直接修改内存，而不是立刻修改磁盘，当内存和磁盘的数据不一致时，称内存中的数据为脏页(dirty page)。为了保证数据的安全性，事务进行中时会不断的产生redo log，在事务提交时进行一次flush操作，保存到磁盘中, redo log是按照顺序写入的，磁盘的顺序读写的速度远大于随机读写。当数据库或主机失效重启时，会根据redo log进行数据的恢复，如果redo log中有事务提交，则进行事务提交修改数据。这样实现了事务的原子性、一致性和持久性。

**undo log**： undo log用于数据的撤回操作，它记录了修改的反向操作，比如，插入对应删除，修改对应修改为原来的数据，通过undo log可以实现事务回滚，并且可以根据undo log回溯到某个特定的版本的数据，实现MVCC。

#### RC和RR在使用MVCC的区别

`RC`、`RR` 两种隔离级别的事务在执行普通的读操作时，通过访问版本链的方法，使得事务间的读写操作得以并发执行，从而提升系统性能。`RC`、`RR` 这两个隔离级别的一个很大不同就是生成 `ReadView` 的时间点不同，`RC` 在每一次 `SELECT` 语句前都会生成一个 `ReadView`，事务期间会更新，因此在其他事务提交前后所得到的 `m_ids` 列表可能发生变化，使得先前不可见的版本后续又突然可见了。而 `RR` 只在事务的第一个 `SELECT` 语句时生成一个 `ReadView`，事务操作期间不更新。

#### 使用流程

这里需要用到前面的知识，关于[行格式的](# 记录的真实数据)

这里面有3个mysql添加的字段`row_id`，`transaction_id`，`roll_pointer`，实现MVCC的多版本控制就主要依赖于后面2个参数。

`transaction_id`：用来标识最近一次对本行记录做修改(insert|update)的事务的标识符, 即最后一次修改(insert|update)本行记录的事务id。

`roll_pointer`：指写入回滚段(rollback segment)的 `undo log` record (撤销日志记录记录)，如果一行记录被更新, 则 `undo log` record 包含 '重建该行记录被更新之前内容' 所必须的信息。也就是通过这个可以形成一条版本链。

![image_1d6vfrv111j4guetptcts1qgp40.png-57.1kB](https://user-gold-cdn.xitu.io/2019/3/27/169bf198524e1b34?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

例子可以[参考这里](https://juejin.im/post/5c9b1b7df265da60e21c0b57#heading-6)，下面说一下增删查改的不同。

**SELECT**

读取创建版本小于或等于当前事务版本号，并且删除版本为空或大于当前事务版本号的记录。也就是在这之前创建且没有被删除的记录。

**INSERT**

将当前事务的版本号保存至行的创建版本号，新插入一行。

**UPDATE**

新插入一行，并以当前事务的版本号作为新行的创建版本号，同时将原记录行的删除版本号设置为当前事务版本号

**DELETE**

将当前事务的版本号保存至行的删除版本号

### 乐观锁和悲观锁

基本信息可以参考我的这篇博客[事务隔离性以及乐观锁和悲观锁](https://blog.csdn.net/yiqzq/article/details/104533487)

悲观锁是数据库层面加锁，都会阻塞去等待锁

乐观锁不是数据库层面上的锁，是需要自己手动去加的锁。

所以，针对乐观锁，一般有以下两种是实现形式

1. 版本号机制

2. CAS算法

版本号机制：一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

CAS算法：即**compare and swap（比较与交换）**

CAS算法涉及到三个操作数

- 需要读写的内存值 V
- 进行比较的值 A
- 拟写入的新值 U

CAS具体执行时，当且仅当预期值A符合内存地址V中存储的值时，就用新值U替换掉旧值，并写入到内存地址V中。否则不做更新。

用下面的图可以更好的理解这个算法



![CAS算法图解](https://user-gold-cdn.xitu.io/2019/9/3/16cf74dbfb0b7acf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**CAS算法带来的问题——ABA问题**

如果内存地址V初次读取的值是A，并且在准备赋值的时候检查到它的值仍然为A，那我们就能说它的值没有被其他线程改变过了吗？如果在这段期间它的值曾经被改成了B，后来又被改回为A，那CAS操作就会误认为它从来没有被改变过。这个漏洞称为CAS操作的“ABA”问题。

**解决方法：使用版本号来区分有没有被修改**

从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

同时CAS算法也有循环开销时间大的问题，如果长时间不成功，会给CPU带来非常大的执行开销

### 间隙锁GAP（Next-Key锁）

当我们**用范围条件检索数据**而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给**符合范围条件的已有数据记录的索引项加锁**；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。

值得注意的是：间隙锁只会在`Repeatable read`隔离级别下使用~

例子：假如emp表中只有101条记录，其empid的值分别是1,2,...,100,101

```sql
Select * from  emp where empid > 100 for update;
```

上面是一个范围查询，InnoDB**不仅**会对符合条件的empid值为101的记录加锁，也会对**empid大于101（这些记录并不存在）的“间隙”加锁**。

InnoDB使用间隙锁的目的有两个：

- **为了防止幻读**(上面也说了，`Repeatable read`隔离级别下再通过GAP锁即可避免了幻读)
- 满足恢复和复制的需要
  - MySQL的恢复机制要求：**在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读**

**所以在RR隔离级别下使用间隙锁可以解决幻读问题。**

### 死锁

**死锁概念**：死锁是指两个或多个事务在同一资源上互相占用，并请求加锁时，而导致的恶性循环现象。当多个事务以不同顺序试图加锁同一资源时，就会产生死锁。

**一些预防措施**

1. 以**固定的顺序**访问表和行。比如对两个job批量更新的情形，简单方法是对id列表先排序，后执行，这样就避免了交叉等待锁的情形；将两个事务的sql顺序调整为一致，也能避免死锁。

2. **大事务拆小**。大事务更倾向于死锁，如果业务允许，将大事务拆小。

3. 在同一个事务中，尽可能做到**一次锁定**所需要的所有资源，减少死锁概率。

4. **降低隔离级别**。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。
5. **为表添加合理的索引**。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。

**解决死锁方法**

1.等待事务超时，主动回滚。

2.进行死锁检查，主动回滚某条事务，让别的事务能继续走下去。

## 千万级别的数据的优化

[MySQL 对于千万级的大表要怎么优化？](https://www.zhihu.com/question/19719997)

## 关于count函数

**比较count（1）和count（*）**

> InnoDB handles SELECT COUNT(*) and SELECT COUNT(1) operations in the same way. There is no performance difference.

Mysql官方的解释count（1）和count（\*）是完全一样的，不存在性能上的差异，但是这两者相比较而言，更推荐使用count（\*）,因为count（\*）是SQL92定义的标准统计行数的语法。

**关于count（字段）**

他的查询比较简单粗暴，就是进行全表扫描，然后判断指定字段的值是不是为NULL，不为NULL则累加。

相比`COUNT(*)`，`COUNT(字段)`多了一个步骤就是判断所查询的字段是否为NULL，所以他的性能要比`COUNT(*)`慢。

**mysql对count（*）的优化**

前面提到了`COUNT(*)`是SQL92定义的标准统计行数的语法，所以MySQL数据库对他进行过很多优化。那么，具体都做过哪些事情呢？

这里的介绍要区分不同的执行引擎。MySQL中比较常用的执行引擎就是InnoDB和MyISAM。

MyISAM和InnoDB有很多区别，其中有一个关键的区别和我们接下来要介绍的`COUNT(*)`有关，那就是**MyISAM不支持事务，MyISAM中的锁是表级锁；而InnoDB支持事务，并且支持行级锁。**

因为MyISAM的锁是表级锁，所以同一张表上面的操作需要串行进行，所以，**MyISAM做了一个简单的优化，那就是它可以把表的总行数单独记录下来，如果从一张表中使用COUNT(\*)进行查询的时候，可以直接返回这个记录下来的数值就可以了，当然，前提是不能有where条件。**

MyISAM之所以可以把表中的总行数记录下来供COUNT(*)查询使用，那是因为MyISAM数据库是表级锁，不会有并发的数据库行数修改，所以查询得到的行数是准确的。

但是，对于InnoDB来说，就不能做这种缓存操作了，因为InnoDB支持事务，其中大部分操作都是行级锁，所以可能表的行数可能会被并发修改，那么缓存记录下来的总行数就不准确了。

但是，InnoDB还是针对COUNT(*)语句做了些优化的。

在InnoDB中，使用COUNT(*)查询行数的时候，不可避免的要进行扫表了，那么，就可以在扫表过程中下功夫来优化效率了。

从MySQL 8.0.13开始，针对InnoDB的`SELECT COUNT(*) FROM tbl_name`语句，确实在扫表的过程中做了一些优化。前提是查询语句中不包含WHERE或GROUP BY等条件。

**我们知道，COUNT(\*)的目的只是为了统计总行数，所以，他根本不关心自己查到的具体值，所以，他如果能够在扫表的过程中，选择一个成本较低的索引进行的话，那就可以大大节省时间。**

我们知道，InnoDB中索引分为聚簇索引（主键索引）和非聚簇索引（非主键索引），聚簇索引的叶子节点中保存的是整行记录，而非聚簇索引的叶子节点中保存的是该行记录的主键的值。

所以，相比之下，非聚簇索引要比聚簇索引小很多，所以**MySQL会优先选择最小的非聚簇索引来扫表。所以，当我们建表的时候，除了主键索引以外，创建一个非主键索引还是有必要的。**

至此，我们介绍完了MySQL数据库对于COUNT(*)的优化，这些优化的前提都是查询语句中不包含WHERE以及GROUP BY条件。

参考内容：[不就是SELECT COUNT语句吗，竟然能被面试官虐的体无完肤](https://blog.csdn.net/hollis_chuang/article/details/102657937)abgf