---
title: Mybatis源码解读
date: 2019-09-24 19:54:35
tags: [MyBatis]
categories: Mybatis
---

## 整体代码结构

```shell
└── org
    └── apache
        └── ibatis
            ├── annotations
            ├── binding
            ├── builder
            ├── cache
            ├── cursor
            ├── datasource
            ├── exceptions
            ├── executor
            ├── io
            ├── jdbc
            ├── lang
            ├── logging
            ├── mapping
            ├── package-info.java
            ├── parsing
            ├── plugin
            ├── reflection
            ├── scripting
            ├── session
            ├── transaction
            ├── type
            └── util
```



## 几个核心包

### Executor

```shell
├── BaseExecutor.java
├── BatchExecutor.java
├── BatchExecutorException.java
├── BatchResult.java
├── CachingExecutor.java
├── ErrorContext.java
├── ExecutionPlaceholder.java
├── Executor.java
├── ExecutorException.java
├── ResultExtractor.java
├── ReuseExecutor.java
├── SimpleExecutor.java
├── keygen
│   ├── Jdbc3KeyGenerator.java
│   ├── KeyGenerator.java
│   ├── NoKeyGenerator.java
│   ├── SelectKeyGenerator.java
│   └── package-info.java
├── loader
│   ├── AbstractEnhancedDeserializationProxy.java
│   ├── AbstractSerialStateHolder.java
│   ├── CglibProxyFactory.java
│   ├── JavassistProxyFactory.java
│   ├── ProxyFactory.java
│   ├── ResultLoader.java
│   ├── ResultLoaderMap.java
│   ├── WriteReplaceInterface.java
│   ├── cglib
│   ├── javassist
│   └── package-info.java
├── package-info.java
├── parameter
│   ├── ParameterHandler.java
│   └── package-info.java
├── result
│   ├── DefaultMapResultHandler.java
│   ├── DefaultResultContext.java
│   ├── DefaultResultHandler.java
│   ├── ResultMapException.java
│   └── package-info.java
├── resultset
│   ├── DefaultResultSetHandler.java
│   ├── ResultSetHandler.java
│   ├── ResultSetWrapper.java
│   └── package-info.java
└── statement
    ├── BaseStatementHandler.java
    ├── CallableStatementHandler.java
    ├── PreparedStatementHandler.java
    ├── RoutingStatementHandler.java
    ├── SimpleStatementHandler.java
    ├── StatementHandler.java
    ├── StatementUtil.java
    └── package-info.java
```



### Session

```shell
├── AutoMappingBehavior.java
├── AutoMappingUnknownColumnBehavior.java
├── Configuration.java
├── ExecutorType.java
├── LocalCacheScope.java
├── ResultContext.java
├── ResultHandler.java
├── RowBounds.java
├── SqlSession.java
├── SqlSessionException.java
├── SqlSessionFactory.java
├── SqlSessionFactoryBuilder.java
├── SqlSessionManager.java
├── TransactionIsolationLevel.java
├── defaults
└── package-info.java
```



## 数据库连接打开

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



## Select 流程

### 建连

```
at org.apache.ibatis.transaction.jdbc.JdbcTransaction.getConnection(JdbcTransaction.java:60)
at org.apache.ibatis.executor.BaseExecutor.getConnection(BaseExecutor.java:337)
at org.apache.ibatis.executor.SimpleExecutor.prepareStatement(SimpleExecutor.java:86)
at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:62)
at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:325)
at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:156)
at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:103)
at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:89)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:151)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:145)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:140)
at org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(DefaultSqlSession.java:76)
at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:87)
at org.apache.ibatis.binding.MapperProxy$PlainMethodInvoker.invoke(MapperProxy.java:145)
at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:86)
at com.sun.proxy.$Proxy.select(Unknown Source:-1)
```

### 执行

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



