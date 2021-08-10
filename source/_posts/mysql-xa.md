---
title: mysql-xa
date: 2020-06-15 17:31:42
tags: [mysql binlog xa]
---

MySQL XA





```java
 				boolean logXaCommands = true;
        // 获得资源管理器操作接口实例 RM1
        Connection conn1 = DriverManager.getConnection("jdbc:mysql://localhost:33060/d1", "root", "pass");
        XAConnection xaConn1 = new MysqlXAConnection((JdbcConnection)conn1, logXaCommands);
        XAResource rm1 = xaConn1.getXAResource();

        // 获得资源管理器操作接口实例 RM2
        Connection conn2 = DriverManager.getConnection("jdbc:mysql://localhost:33060/d2", "root", "pass");
        XAConnection xaConn2 = new MysqlXAConnection((JdbcConnection)conn2, logXaCommands);
        XAResource rm2 = xaConn2.getXAResource();
        // AP请求TM执行一个分布式事务，TM生成全局事务id
        byte[] gtrid = "g12345".getBytes();
        int formatId = 1;
        try {
            // ==============分别执行RM1和RM2上的事务分支====================
            // TM生成rm1上的事务分支id
            byte[] bqual1 = "b00001".getBytes();
            Xid xid1 = new MysqlXid(gtrid, bqual1, formatId);
            // 执行rm1上的事务分支 One of TMNOFLAGS, TMJOIN, or TMRESUME.
            rm1.start(xid1, XAResource.TMNOFLAGS);
            // 业务1：插入user表
            PreparedStatement ps1 = conn1.prepareStatement("insert into t1 values (10,'ab')");
            ps1.execute();
            rm1.end(xid1, XAResource.TMSUCCESS);

            // TM生成rm2上的事务分支id
            byte[] bqual2 = "b00002".getBytes();
            Xid xid2 = new MysqlXid(gtrid, bqual2, formatId);
            // 执行rm2上的事务分支
            rm2.start(xid2, XAResource.TMNOFLAGS);
            // 业务2：插入user_msg表
            PreparedStatement ps2 = conn2.prepareStatement("insert into t2 values (20,'cd')");
            ps2.execute();
            rm2.end(xid2, XAResource.TMSUCCESS);

            // ===================两阶段提交================================
            // phase1：询问所有的RM 准备提交事务分支
            int rm1Prepare = rm1.prepare(xid1);
            int rm2Prepare = rm2.prepare(xid2);
            // phase2：提交所有事务分支
            boolean onePhase = false;
            //TM判断有2个事务分支，所以不能优化为一阶段提交
            if (rm1Prepare == XAResource.XA_OK && rm2Prepare == XAResource.XA_OK) {
                //所有事务分支都prepare成功，提交所有事务分支
                rm1.commit(xid1, onePhase);
                rm2.commit(xid2, onePhase);
            } else {
                //如果有事务分支没有成功，则回滚
                rm1.rollback(xid1);
                rm1.rollback(xid2);
            }
        } catch (XAException e) {
            // 如果出现异常，也要进行回滚
            e.printStackTrace();
        }
```

## 事务落在同一个MySQL

```reStructuredText
+---------------+------+----------------+-----------+-------------+----------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                         |
+---------------+------+----------------+-----------+-------------+----------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         1 |         125 | Server ver: 8.0.20, Binlog ver: 4            |
| binlog.000001 |  125 | Previous_gtids |         1 |         156 |                                              |
| binlog.000001 |  156 | Anonymous_Gtid |         1 |         235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 |  235 | Query          |         1 |         345 | XA START X'673132333435',X'623030303031',1   |
| binlog.000001 |  345 | Table_map      |         1 |         399 | table_id: 94 (d1.t1)                         |
| binlog.000001 |  399 | Write_rows     |         1 |         442 | table_id: 94 flags: STMT_END_F               |
| binlog.000001 |  442 | Query          |         1 |         550 | XA END X'673132333435',X'623030303031',1     |
| binlog.000001 |  550 | XA_prepare     |         1 |         598 | XA PREPARE X'673132333435',X'623030303031',1 |
| binlog.000001 |  598 | Anonymous_Gtid |         1 |         677 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 |  677 | Query          |         1 |         787 | XA START X'673132333435',X'623030303032',1   |
| binlog.000001 |  787 | Table_map      |         1 |         841 | table_id: 96 (d2.t2)                         |
| binlog.000001 |  841 | Write_rows     |         1 |         884 | table_id: 96 flags: STMT_END_F               |
| binlog.000001 |  884 | Query          |         1 |         992 | XA END X'673132333435',X'623030303032',1     |
| binlog.000001 |  992 | XA_prepare     |         1 |        1040 | XA PREPARE X'673132333435',X'623030303032',1 |
| binlog.000001 | 1040 | Anonymous_Gtid |         1 |        1117 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 | 1117 | Query          |         1 |        1228 | XA COMMIT X'673132333435',X'623030303031',1  |
| binlog.000001 | 1228 | Anonymous_Gtid |         1 |        1305 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 | 1305 | Query          |         1 |        1416 | XA COMMIT X'673132333435',X'623030303032',1  |
+---------------+------+----------------+-----------+-------------+----------------------------------------------+
```

![](https://youuwd.github.io/images/xa.png)





## 事务落在2个不同MySQL

### MySQL 1

```reStructuredText
+---------------+-----+----------------+-----------+-------------+----------------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                         |
+---------------+-----+----------------+-----------+-------------+----------------------------------------------+
| binlog.000001 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.20, Binlog ver: 4            |
| binlog.000001 | 125 | Previous_gtids |         1 |         156 |                                              |
| binlog.000001 | 156 | Anonymous_Gtid |         1 |         235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 | 235 | Query          |         1 |         345 | XA START X'673132333435',X'623030303031',1   |
| binlog.000001 | 345 | Table_map      |         1 |         399 | table_id: 101 (d1.t1)                        |
| binlog.000001 | 399 | Write_rows     |         1 |         442 | table_id: 101 flags: STMT_END_F              |
| binlog.000001 | 442 | Query          |         1 |         550 | XA END X'673132333435',X'623030303031',1     |
| binlog.000001 | 550 | XA_prepare     |         1 |         598 | XA PREPARE X'673132333435',X'623030303031',1 |
| binlog.000001 | 598 | Anonymous_Gtid |         1 |         675 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 | 675 | Query          |         1 |         786 | XA COMMIT X'673132333435',X'623030303031',1  |
+---------------+-----+----------------+-----------+-------------+----------------------------------------------+
```



### MySQL 2

```reStructuredText
+---------------+-----+----------------+-----------+-------------+----------------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                         |
+---------------+-----+----------------+-----------+-------------+----------------------------------------------+
| binlog.000001 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.20, Binlog ver: 4            |
| binlog.000001 | 125 | Previous_gtids |         1 |         156 |                                              |
| binlog.000001 | 156 | Anonymous_Gtid |         1 |         235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 | 235 | Query          |         1 |         345 | XA START X'673132333435',X'623030303032',1   |
| binlog.000001 | 345 | Table_map      |         1 |         399 | table_id: 106 (d2.t2)                        |
| binlog.000001 | 399 | Write_rows     |         1 |         442 | table_id: 106 flags: STMT_END_F              |
| binlog.000001 | 442 | Query          |         1 |         550 | XA END X'673132333435',X'623030303032',1     |
| binlog.000001 | 550 | XA_prepare     |         1 |         598 | XA PREPARE X'673132333435',X'623030303032',1 |
| binlog.000001 | 598 | Anonymous_Gtid |         1 |         675 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'         |
| binlog.000001 | 675 | Query          |         1 |         786 | XA COMMIT X'673132333435',X'623030303032',1  |
+---------------+-----+----------------+-----------+-------------+----------------------------------------------+
```





#### 