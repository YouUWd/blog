---
title: Mybatis源码解读
date: 2019-09-24 19:54:35
tags: [MyBatis]
categories: Mybatis
---

## 整体代码结构

![Package ibatis](https://youuwd.github.io/images/Package-ibatis.png)

## 几个核心包

### Executor

![Package Executor](https://youuwd.github.io/images/Package-executor.png)

### Session

![Package session](https://youuwd.github.io/images/Package-session.png)

### SqlSession

![sqlSession](https://youuwd.github.io/images/SqlSession.png)



## 数据库连接打开

![transaction](https://youuwd.github.io/images/transaction.png)

![transaction-jdbc](https://youuwd.github.io/images/jdbc.png)

`org.apache.ibatis.transaction.jdbc.JdbcTransaction#openConnection`

## Mapper 生成

```java
T org.apache.ibatis.session.defaults.DefaultSqlSession#getMapper
T org.apache.ibatis.session.Configuration#getMapper
T org.apache.ibatis.binding.MapperRegistry#getMapper
T org.apache.ibatis.binding.MapperProxyFactory#newInstance(org.apache.ibatis.session.SqlSession)
T java.lang.reflect.Proxy#newProxyInstance
```



## SQL流图

![sql](https://youuwd.github.io/images/query.jpg)



## Select 流程

```java
org.apache.ibatis.executor.CachingExecutor#query(ms,parameterObject,rowBounds,resultHandler)
org.apache.ibatis.executor.BaseExecutor#query(ms,parameter,rowBounds,resultHandler,key,boundSql)
org.apache.ibatis.executor.BaseExecutor#queryFromDatabase
org.apache.ibatis.executor.SimpleExecutor#doQuery
org.apache.ibatis.executor.statement.RoutingStatementHandler#parameterize
org.apache.ibatis.executor.statement.PreparedStatementHandler#parameterize
org.apache.ibatis.scripting.defaults.DefaultParameterHandler#setParameters
org.apache.ibatis.executor.statement.RoutingStatementHandler#query
org.apache.ibatis.executor.resultset.DefaultResultSetHandler#handleResultSets
```

<font color="red">Executor ==> Statement ==> Parameter ==> ResultSet</font>

> *绿色* 几个组件都是可以通过mybatis插件改写的，参考  [mybatis plugin](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
>
> - Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
>
> - StatementHandler (prepare, parameterize, batch, update, query)
>
> - ParameterHandler (getParameterObject, setParameters)
>
> - ResultSetHandler (handleResultSets, handleOutputParameters)
>
>   



