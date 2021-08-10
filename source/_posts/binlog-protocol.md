---
title: binlog-protocol
date: 2020-06-02 14:58:21
tags: [binlog protocol]
---

## 生成新的binlog

```mysql
flush logs;
```

## 清空binlog

```mysql
reset master;
```



## 格式

Binlog files start with a [Binlog File Header](https://dev.mysql.com/doc/internals/en/binlog-file-header.html) followed by a series of [Binlog Event](https://dev.mysql.com/doc/internals/en/binlog-event.html)

Header + Event + Event + Event

[Header](https://dev.mysql.com/doc/internals/en/binlog-file-header.html)

A binlog file starts with a `Binlog File Header` `[ fe 'bin' ]`

## Event

EventHeader + Event

#### Binlog Event header

The binlog event header starts each event and is either 13 or 19 bytes long, depending on the [binlog version](https://dev.mysql.com/doc/internals/en/binlog-version.html).

```reStructuredText
4              timestamp
1              event type
4              server-id
4              event-size
   if binlog-version > 1:
4              log pos
2              flags
```

```
+---------+---------+---------+------------+-------------+-------+
|timestamp|type code|server_id|event_length|next_position|flags  |
|4 bytes  |1 byte   |4 bytes  |4 bytes     |4 bytes      |2 bytes|
+---------+---------+---------+------------+-------------+-------+
```



**Fields**

- **timestamp** ([*4*](https://dev.mysql.com/doc/internals/en/describing-packets.html#type-4)) -- seconds since unix epoch
- **event_type** ([*1*](https://dev.mysql.com/doc/internals/en/describing-packets.html#type-1)) -- see [Binlog Event Type](https://dev.mysql.com/doc/internals/en/binlog-event-type.html)
- **server_id** ([*4*](https://dev.mysql.com/doc/internals/en/describing-packets.html#type-4)) -- server-id of the originating mysql-server. Used to filter out events in circular replication.
- **event_size** ([*4*](https://dev.mysql.com/doc/internals/en/describing-packets.html#type-4)) -- size of the event (header, post-header, body)
- **log_pos** ([*4*](https://dev.mysql.com/doc/internals/en/describing-packets.html#type-4)) -- position of the next event
- **flags** ([*2*](https://dev.mysql.com/doc/internals/en/describing-packets.html#type-2)) -- see [Binlog Event Flag](https://dev.mysql.com/doc/internals/en/binlog-event-flag.html)

**Table 14.5 Binlog Versions**

| Binlog version | MySQL Version         |
| -------------- | --------------------- |
| 1              | MySQL 3.23 - < 4.0.0  |
| 2              | MySQL 4.0.0 - 4.0.1   |
| 3              | MySQL 4.0.2 - < 5.0.0 |
| 4              | MySQL 5.0.0+          |

目前市面上都是V4了，故EventHeader的长度可以默认为19

**Binlog::FORMAT_DESCRIPTION_EVENT:**

FORMAT_DESCRIPTION_EVENT 是binlog文件的第一个 Event

```cpp
//https://sourcegraph.com/github.com/mysql/mysql-server/-/blob/libbinlogevents/src/binlog_event.cpp#L84:1
/**
   The method returns the checksum algorithm used to checksum the binary log.
   For MySQL server versions < 5.6, the algorithm is undefined. For the higher
   versions, the type is decoded from the FORMAT_DESCRIPTION_EVENT.

   @param buf buffer holding serialized FD event
   @param len netto (possible checksum is stripped off) length of the event buf

   @return  the version-safe checksum alg descriptor where zero
            designates no checksum, 255 - the orginator is
            checksum-unaware (effectively no checksum) and the actuall
            [1-254] range alg descriptor.
*/
enum_binlog_checksum_alg Log_event_footer::get_checksum_alg(const char *buf,
                                                            unsigned long len) {
  BAPI_ENTER("Log_event_footer::get_checksum_alg(const char*, unsigned long)");
  enum_binlog_checksum_alg ret = BINLOG_CHECKSUM_ALG_UNDEF;
  char version[ST_SERVER_VER_LEN];
  unsigned char version_split[3];
  BAPI_ASSERT(buf[EVENT_TYPE_OFFSET] == FORMAT_DESCRIPTION_EVENT);
  if (len > LOG_EVENT_MINIMAL_HEADER_LEN + ST_COMMON_HEADER_LEN_OFFSET + 1) {
    uint8_t common_header_len =
        buf[LOG_EVENT_MINIMAL_HEADER_LEN + ST_COMMON_HEADER_LEN_OFFSET];
    if (len >=
        static_cast<unsigned long>(common_header_len + ST_SERVER_VER_OFFSET +
                                   ST_SERVER_VER_LEN)) {
      memcpy(version, buf + common_header_len + ST_SERVER_VER_OFFSET,
             ST_SERVER_VER_LEN);
      version[ST_SERVER_VER_LEN - 1] = 0;

      do_server_version_split(version, version_split);
      if (version_product(version_split) < checksum_version_product)
        ret = BINLOG_CHECKSUM_ALG_UNDEF;
      else {
        size_t checksum_alg_offset =
            len - (BINLOG_CHECKSUM_ALG_DESC_LEN + BINLOG_CHECKSUM_LEN);
        ret =
            static_cast<enum_binlog_checksum_alg>(*(buf + checksum_alg_offset));
      }
    }
  }
  BAPI_RETURN(ret);
}

```



> A format description event is the first event of a binlog for binlog-version 4. It describes how the other events are layed out.

**Row Based Replication Events**

In Row Based replication the changed rows are sent to the slave which removes side-effects and makes it more reliable. Now all statements can be sent with RBR though. Most of the time you will see RBR and SBR side by side.

- [TABLE_MAP_EVENT](https://dev.mysql.com/doc/internals/en/table-map-event.html)
- [DELETE_ROWS_EVENTv0](https://dev.mysql.com/doc/internals/en/rows-event.html#delete-rows-eventv0)
- [UPDATE_ROWS_EVENTv0](https://dev.mysql.com/doc/internals/en/rows-event.html#update-rows-eventv0)
- [WRITE_ROWS_EVENTv0](https://dev.mysql.com/doc/internals/en/rows-event.html#write-rows-eventv0)
- [DELETE_ROWS_EVENTv1](https://dev.mysql.com/doc/internals/en/rows-event.html#delete-rows-eventv1)
- [UPDATE_ROWS_EVENTv1](https://dev.mysql.com/doc/internals/en/rows-event.html#update-rows-eventv1)
- [WRITE_ROWS_EVENTv1](https://dev.mysql.com/doc/internals/en/rows-event.html#write-rows-eventv1)
- [DELETE_ROWS_EVENTv2](https://dev.mysql.com/doc/internals/en/rows-event.html#delete-rows-eventv2)
- [UPDATE_ROWS_EVENTv2](https://dev.mysql.com/doc/internals/en/rows-event.html#update-rows-eventv2)
- [WRITE_ROWS_EVENTv2](https://dev.mysql.com/doc/internals/en/rows-event.html#write-rows-eventv2)



```reStructuredText
00000004  87 77 d4 5e|0f|01 00 00  00|79 00 00 00|7d 00 00  |.w.^.....y...}..|
00000014  00|00 00|04 00|38 2e 30  2e 32 30 00 00 00 00 00  |.....8.0.20.....|
00000024  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000034  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000044  00 00 00 00 00 00 00|87  77 d4 5e|13|00 0d 00 08  |........w.^.....|
00000054  00 00 00 00 04 00 04 00  00 00|61|00 04 1a 08 00  |..........a.....|
00000064  00 00 08 08 08 02 00 00  00 0a 0a 0a 2a 2a 00 12  |............**..|
00000074  34 00 0a 28|01|63 c8 ff  8b                       |4..(.c....w.^#..|

```

**Event Header**

> timestamp:	87 77 d4 5e  #1590982535
>
> event type:	0f  # [FORMAT_DESCRIPTION_EVENT](https://dev.mysql.com/doc/internals/en/format-description-event.html)
>
> server-id:	01 00 00  00  #1
>
> event-size:	79 00 00 00     #121
>
> log pos:	7d 00 00 00	#125 position of the next event
>
> flag:	00 00	#0
>
> 

**Event Body** (FORMAT_DESCRIPTION_EVENT)

>binlog-version:	04 00	#4
>
>mysql-server version:	38 2e 30  2e 32 30 00 00 00 00 00 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00	#8.0.20
>
>create timestamp:	87 77 d4 5e	#1590982535
>
>event header length:	13	#19 固定值
>
>event type header lengths：	00 0d 00 08 00 00 00 00 04 00 04 00  00 00	# 长度(15-1)
>
>data length:	61	#97   ==> #check sum block = event-size - event header length - data length = 121-19-97 = 5
>
>skip:	121(eventSize)-19(header)-2(binlogVersion)-50(serverVersion)-4(time)-1(headerLength)-14(typeHeaderLength)-1(dataLength)-5(checkSumBlock) = 25
>
>check sum type:	01 #1 CRC32

例如：[TABLE_MAP_EVENT](https://dev.mysql.com/doc/internals/en/table-map-event.html)

```reStructuredText
post-header:
    if post_header_len == 6 {
  4              table id
    } else {
  6              table id
    }
  2              flags

payload:
  1              schema name length
  string         schema name
  1              [00]
  1              table name length
  string         table name
  1              [00]
  lenenc-int     column-count
  string.var_len [length=$column-count] column-def
  lenenc-str     column-meta-def
  n              NULL-bitmask, length: (column-count + 8) / 7
```



```reStructuredText
+=====================================+
|        | fixed part (post-header)   |
| data   +----------------------------+
|        | variable part (payload)    |
+=====================================+
```

[参考](https://dev.mysql.com/doc/internals/en/binlog-file.html)



