# MyBatis 源码解析 - 完整教程

> 深入理解 MyBatis 的核心原理和执行流程
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | MyBatis |
| **当前版本** | 3.5.x |
| **源码地址** | https://github.com/mybatis/mybatis-3 |
| **学习难度** | ⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐⭐ |
| **预计时长** | 30-40 小时 |
| **前置知识** | JDBC、SQL、反射、动态代理 |

## 🎯 学习目标

- [ ] 理解 MyBatis 的整体架构
- [ ] 掌握配置文件的解析流程
- [ ] 深入理解 SQL 执行的完整流程
- [ ] 掌握一级缓存和二级缓存的实现
- [ ] 理解插件机制的原理
- [ ] 掌握结果集映射的实现
- [ ] 能够自定义 TypeHandler 和插件

## 📖 目录

1. [MyBatis 整体架构](#1-mybatis-整体架构)
2. [配置文件解析](#2-配置文件解析)
3. [SQL 执行流程](#3-sql-执行流程)
4. [缓存机制](#4-缓存机制)
5. [插件机制](#5-插件机制)
6. [结果集映射](#6-结果集映射)

---

## 1. MyBatis 整体架构

### 1.1 核心组件

```
mybatis-3/
├── SqlSessionFactoryBuilder  # 构建 SqlSessionFactory
├── SqlSessionFactory          # 创建 SqlSession
├── SqlSession                 # 执行 SQL 的会话
├── Executor                   # SQL 执行器
├── StatementHandler           # JDBC Statement 处理器
├── ParameterHandler           # 参数处理器
├── ResultSetHandler           # 结果集处理器
└── TypeHandler                # 类型处理器
```

### 1.2 四大对象 🔥

```java
// 1. Executor - 执行器
public interface Executor {
    ResultHandler NO_RESULT_HANDLER = null;
    
    int update(MappedStatement ms, Object parameter) throws SQLException;
    <E> List<E> query(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
    void commit(boolean required) throws SQLException;
    void rollback(boolean required) throws SQLException;
}

// 2. StatementHandler - 语句处理器
public interface StatementHandler {
    Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException;
    void parameterize(Statement statement) throws SQLException;
    <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException;
    int update(Statement statement) throws SQLException;
}

// 3. ParameterHandler - 参数处理器
public interface ParameterHandler {
    Object getParameterObject();
    void setParameters(PreparedStatement ps) throws SQLException;
}

// 4. ResultSetHandler - 结果集处理器
public interface ResultSetHandler {
    <E> List<E> handleResultSets(Statement stmt) throws SQLException;
    <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;
    void handleOutputParameters(CallableStatement cs) throws SQLException;
}
```

### 1.3 执行流程概览

```
1. 创建 SqlSessionFactory
   ↓
2. 创建 SqlSession
   ↓
3. 获取 Mapper 代理对象
   ↓
4. 执行 Mapper 方法
   ↓
5. 通过 Executor 执行 SQL
   ↓
6. StatementHandler 处理 JDBC Statement
   ↓
7. ParameterHandler 设置参数
   ↓
8. 执行 SQL
   ↓
9. ResultSetHandler 处理结果集
   ↓
10. 返回结果
```

---

## 2. 配置文件解析

### 2.1 SqlSessionFactory 创建

```java
// 1. 使用 SqlSessionFactoryBuilder 构建
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = 
    new SqlSessionFactoryBuilder().build(inputStream);

// 2. SqlSessionFactoryBuilder.build() 源码
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
        // 创建 XML 配置构建器
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        // 解析配置文件，返回 Configuration 对象
        return build(parser.parse());
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
        ErrorContext.instance().reset();
        try {
            inputStream.close();
        } catch (IOException e) {
            // Intentionally ignore. Prefer previous error.
        }
    }
}

public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
}
```

### 2.2 配置文件解析流程

```java
// XMLConfigBuilder.parse()
public Configuration parse() {
    if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    // 解析 <configuration> 根节点
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
}

// 解析配置
private void parseConfiguration(XNode root) {
    try {
        // 1. 解析 <properties>
        propertiesElement(root.evalNode("properties"));
        
        // 2. 解析 <settings>
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        loadCustomVfs(settings);
        loadCustomLogImpl(settings);
        
        // 3. 解析 <typeAliases>
        typeAliasesElement(root.evalNode("typeAliases"));
        
        // 4. 解析 <plugins>
        pluginElement(root.evalNode("plugins"));
        
        // 5. 解析 <objectFactory>
        objectFactoryElement(root.evalNode("objectFactory"));
        
        // 6. 解析 <objectWrapperFactory>
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        
        // 7. 解析 <reflectorFactory>
        reflectorFactoryElement(root.evalNode("reflectorFactory"));
        
        // 8. 应用 settings
        settingsElement(settings);
        
        // 9. 解析 <environments>
        environmentsElement(root.evalNode("environments"));
        
        // 10. 解析 <databaseIdProvider>
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        
        // 11. 解析 <typeHandlers>
        typeHandlerElement(root.evalNode("typeHandlers"));
        
        // 12. 解析 <mappers> 🔥
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```

### 2.3 Mapper 文件解析 🔥

```java
// XMLMapperBuilder.parse()
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
        // 解析 <mapper> 节点
        configurationElement(parser.evalNode("/mapper"));
        configuration.addLoadedResource(resource);
        // 绑定 Mapper 接口
        bindMapperForNamespace();
    }
    
    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
}

// 解析 <mapper> 节点
private void configurationElement(XNode context) {
    try {
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.isEmpty()) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }
        builderAssistant.setCurrentNamespace(namespace);
        
        // 解析 <cache-ref>
        cacheRefElement(context.evalNode("cache-ref"));
        
        // 解析 <cache>
        cacheElement(context.evalNode("cache"));
        
        // 解析 <parameterMap>（已废弃）
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        
        // 解析 <resultMap>
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        
        // 解析 <sql>
        sqlElement(context.evalNodes("/mapper/sql"));
        
        // 解析 <select|insert|update|delete>
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. The XML location is '" 
            + resource + "'. Cause: " + e, e);
    }
}
```

---

## 3. SQL 执行流程 🔥

### 3.1 获取 Mapper 代理对象

```java
// 1. 从 SqlSession 获取 Mapper
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

// 2. DefaultSqlSession.getMapper() 源码
@Override
public <T> T getMapper(Class<T> type) {
    return configuration.getMapper(type, this);
}

// 3. Configuration.getMapper()
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}

// 4. MapperRegistry.getMapper()
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    // 从缓存获取 MapperProxyFactory
    final MapperProxyFactory<T> mapperProxyFactory = 
        (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
        // 创建代理对象
        return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
}

// 5. MapperProxyFactory.newInstance()
public T newInstance(SqlSession sqlSession) {
    // 创建 MapperProxy（InvocationHandler）
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
}

protected T newInstance(MapperProxy<T> mapperProxy) {
    // 使用 JDK 动态代理创建代理对象
    return (T) Proxy.newProxyInstance(
        mapperInterface.getClassLoader(), 
        new Class[] { mapperInterface }, 
        mapperProxy);
}
```

### 3.2 执行 Mapper 方法

```java
// MapperProxy.invoke() - 代理方法调用
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        // 如果是 Object 的方法，直接执行
        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        } else {
            // 缓存 MapperMethod
            return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
        }
    } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
    }
}

// MapperMethod.execute() - 执行 SQL
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
        case INSERT: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.insert(command.getName(), param));
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(command.getName(), param);
                if (method.returnsOptional() && 
                    (result == null || !method.getReturnType().equals(result.getClass()))) {
                    result = Optional.ofNullable(result);
                }
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    return result;
}
```

### 3.3 Executor 执行 SQL

```java
// DefaultSqlSession.selectList()
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
        // 获取 MappedStatement
        MappedStatement ms = configuration.getMappedStatement(statement);
        // 通过 Executor 执行查询
        return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}

// CachingExecutor.query() - 带缓存的执行器
@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, 
    RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    // 获取 BoundSql（包含完整的 SQL 和参数）
    BoundSql boundSql = ms.getBoundSql(parameterObject);
    // 创建缓存 Key
    CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}

@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, 
    RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
    throws SQLException {
    // 获取二级缓存
    Cache cache = ms.getCache();
    if (cache != null) {
        flushCacheIfRequired(ms);
        if (ms.isUseCache() && resultHandler == null) {
            ensureNoOutParams(ms, boundSql);
            @SuppressWarnings("unchecked")
            List<E> list = (List<E>) tcm.getObject(cache, key);
            if (list == null) {
                // 缓存未命中，查询数据库
                list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                tcm.putObject(cache, key, list);
            }
            return list;
        }
    }
    // 没有二级缓存，直接查询
    return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}

// BaseExecutor.query() - 基础执行器
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, 
    RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) 
    throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
        clearLocalCache();
    }
    List<E> list;
    try {
        queryStack++;
        // 从一级缓存获取
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
            handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
            // 一级缓存未命中，查询数据库
            list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
    } finally {
        queryStack--;
    }
    if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
            deferredLoad.load();
        }
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
            clearLocalCache();
        }
    }
    return list;
}
```


### 3.4 StatementHandler 处理 JDBC

```java
// SimpleExecutor.doQuery()
@Override
public <E> List<E> doQuery(MappedStatement ms, Object parameter, 
    RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
        Configuration configuration = ms.getConfiguration();
        // 创建 StatementHandler
        StatementHandler handler = configuration.newStatementHandler(
            wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
        // 准备 Statement
        stmt = prepareStatement(handler, ms.getStatementLog());
        // 执行查询
        return handler.query(stmt, resultHandler);
    } finally {
        closeStatement(stmt);
    }
}

// 准备 Statement
private Statement prepareStatement(StatementHandler handler, Log statementLog) 
    throws SQLException {
    Statement stmt;
    Connection connection = getConnection(statementLog);
    // 创建 Statement
    stmt = handler.prepare(connection, transaction.getTimeout());
    // 设置参数
    handler.parameterize(stmt);
    return stmt;
}

// PreparedStatementHandler.parameterize()
@Override
public void parameterize(Statement statement) throws SQLException {
    // 使用 ParameterHandler 设置参数
    parameterHandler.setParameters((PreparedStatement) statement);
}

// DefaultParameterHandler.setParameters()
@Override
public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping parameterMapping = parameterMappings.get(i);
            if (parameterMapping.getMode() != ParameterMode.OUT) {
                Object value;
                String propertyName = parameterMapping.getProperty();
                if (boundSql.hasAdditionalParameter(propertyName)) {
                    value = boundSql.getAdditionalParameter(propertyName);
                } else if (parameterObject == null) {
                    value = null;
                } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                    value = parameterObject;
                } else {
                    MetaObject metaObject = configuration.newMetaObject(parameterObject);
                    value = metaObject.getValue(propertyName);
                }
                // 获取 TypeHandler
                TypeHandler typeHandler = parameterMapping.getTypeHandler();
                JdbcType jdbcType = parameterMapping.getJdbcType();
                if (value == null && jdbcType == null) {
                    jdbcType = configuration.getJdbcTypeForNull();
                }
                try {
                    // 设置参数
                    typeHandler.setParameter(ps, i + 1, value, jdbcType);
                } catch (TypeException | SQLException e) {
                    throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                }
            }
        }
    }
}
```

---

## 4. 缓存机制 🔥

### 4.1 一级缓存（SqlSession 级别）

```java
// BaseExecutor 中的一级缓存
public abstract class BaseExecutor implements Executor {
    // 一级缓存，默认开启
    protected PerpetualCache localCache;
    
    protected BaseExecutor(Configuration configuration, Transaction transaction) {
        this.transaction = transaction;
        this.deferredLoads = new ConcurrentLinkedQueue<>();
        this.localCache = new PerpetualCache("LocalCache");
        this.localOutputParameterCache = new PerpetualCache("LocalOutputParameterCache");
        this.closed = false;
        this.configuration = configuration;
        this.wrapper = this;
    }
    
    // 查询时先从缓存获取
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) 
        throws SQLException {
        // 从一级缓存获取
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
            handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
            // 缓存未命中，查询数据库
            list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
        return list;
    }
    
    // 从数据库查询并放入缓存
    private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) 
        throws SQLException {
        List<E> list;
        localCache.putObject(key, EXECUTION_PLACEHOLDER);
        try {
            list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
        } finally {
            localCache.removeObject(key);
        }
        // 放入一级缓存
        localCache.putObject(key, list);
        if (ms.getStatementType() == StatementType.CALLABLE) {
            localOutputParameterCache.putObject(key, parameter);
        }
        return list;
    }
    
    // 更新/插入/删除时清空缓存
    @Override
    public int update(MappedStatement ms, Object parameter) throws SQLException {
        ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
        if (closed) {
            throw new ExecutorException("Executor was closed.");
        }
        clearLocalCache();
        return doUpdate(ms, parameter);
    }
}
```

### 4.2 二级缓存（Mapper 级别）

```java
// 在 Mapper.xml 中配置二级缓存
<cache/>

// 或者自定义配置
<cache
  eviction="LRU"
  flushInterval="60000"
  size="512"
  readOnly="true"/>

// CachingExecutor 实现二级缓存
public class CachingExecutor implements Executor {
    private final Executor delegate;
    private final TransactionalCacheManager tcm = new TransactionalCacheManager();
    
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, 
        RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
        throws SQLException {
        // 获取二级缓存
        Cache cache = ms.getCache();
        if (cache != null) {
            flushCacheIfRequired(ms);
            if (ms.isUseCache() && resultHandler == null) {
                ensureNoOutParams(ms, boundSql);
                @SuppressWarnings("unchecked")
                // 从二级缓存获取
                List<E> list = (List<E>) tcm.getObject(cache, key);
                if (list == null) {
                    // 缓存未命中，查询数据库
                    list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                    // 放入二级缓存
                    tcm.putObject(cache, key, list);
                }
                return list;
            }
        }
        return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
    
    @Override
    public void commit(boolean required) throws SQLException {
        delegate.commit(required);
        // 提交时才真正放入二级缓存
        tcm.commit();
    }
    
    @Override
    public void rollback(boolean required) throws SQLException {
        try {
            delegate.rollback(required);
        } finally {
            if (required) {
                tcm.rollback();
            }
        }
    }
}
```

### 4.3 缓存 Key 的生成

```java
// BaseExecutor.createCacheKey()
@Override
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, 
    RowBounds rowBounds, BoundSql boundSql) {
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    CacheKey cacheKey = new CacheKey();
    // 1. MappedStatement 的 id
    cacheKey.update(ms.getId());
    // 2. 分页参数
    cacheKey.update(rowBounds.getOffset());
    cacheKey.update(rowBounds.getLimit());
    // 3. SQL 语句
    cacheKey.update(boundSql.getSql());
    // 4. 参数值
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();
    for (ParameterMapping parameterMapping : parameterMappings) {
        if (parameterMapping.getMode() != ParameterMode.OUT) {
            Object value;
            String propertyName = parameterMapping.getProperty();
            if (boundSql.hasAdditionalParameter(propertyName)) {
                value = boundSql.getAdditionalParameter(propertyName);
            } else if (parameterObject == null) {
                value = null;
            } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                value = parameterObject;
            } else {
                MetaObject metaObject = configuration.newMetaObject(parameterObject);
                value = metaObject.getValue(propertyName);
            }
            cacheKey.update(value);
        }
    }
    // 5. Environment id
    if (configuration.getEnvironment() != null) {
        cacheKey.update(configuration.getEnvironment().getId());
    }
    return cacheKey;
}
```

---

## 5. 插件机制 🔥

### 5.1 插件原理

```java
// Interceptor 接口
public interface Interceptor {
    // 拦截方法
    Object intercept(Invocation invocation) throws Throwable;
    
    // 生成代理对象
    default Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    
    // 设置属性
    default void setProperties(Properties properties) {
    }
}

// 使用注解指定拦截点
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
    )
})
public class ExamplePlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 前置处理
        System.out.println("Before query");
        
        // 执行原方法
        Object result = invocation.proceed();
        
        // 后置处理
        System.out.println("After query");
        
        return result;
    }
}
```


### 5.2 插件代理实现

```java
// Plugin.wrap() - 创建代理对象
public static Object wrap(Object target, Interceptor interceptor) {
    // 获取拦截器要拦截的接口和方法
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    // 获取目标对象实现的被拦截的接口
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
        // 使用 JDK 动态代理创建代理对象
        return Proxy.newProxyInstance(
            type.getClassLoader(),
            interfaces,
            new Plugin(target, interceptor, signatureMap));
    }
    return target;
}

// Plugin.invoke() - 代理方法调用
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        Set<Method> methods = signatureMap.get(method.getDeclaringClass());
        // 如果是被拦截的方法，执行拦截器
        if (methods != null && methods.contains(method)) {
            return interceptor.intercept(new Invocation(target, method, args));
        }
        // 否则直接执行原方法
        return method.invoke(target, args);
    } catch (Exception e) {
        throw ExceptionUtil.unwrapThrowable(e);
    }
}
```

### 5.3 插件链的构建

```java
// Configuration.newExecutor() - 创建 Executor 时应用插件
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
        executor = new CachingExecutor(executor);
    }
    // 应用插件（责任链模式）
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}

// InterceptorChain.pluginAll()
public Object pluginAll(Object target) {
    // 遍历所有插件，依次创建代理
    for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target);
    }
    return target;
}
```

### 5.4 常见插件示例

```java
/**
 * 分页插件
 * 
 * @author erik.zhou
 */
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
    )
})
public class PageInterceptor implements Interceptor {
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        Object parameter = args[1];
        RowBounds rowBounds = (RowBounds) args[2];
        
        // 如果需要分页
        if (rowBounds != null && rowBounds != RowBounds.DEFAULT) {
            BoundSql boundSql = ms.getBoundSql(parameter);
            String sql = boundSql.getSql();
            
            // 生成分页 SQL
            String pageSql = sql + " LIMIT " + rowBounds.getOffset() + ", " + rowBounds.getLimit();
            
            // 创建新的 BoundSql
            BoundSql newBoundSql = new BoundSql(
                ms.getConfiguration(), pageSql, 
                boundSql.getParameterMappings(), parameter);
            
            // 创建新的 MappedStatement
            MappedStatement newMs = copyFromMappedStatement(ms, new BoundSqlSqlSource(newBoundSql));
            args[0] = newMs;
            args[2] = RowBounds.DEFAULT;
        }
        
        return invocation.proceed();
    }
    
    private MappedStatement copyFromMappedStatement(MappedStatement ms, SqlSource newSqlSource) {
        MappedStatement.Builder builder = new MappedStatement.Builder(
            ms.getConfiguration(), ms.getId(), newSqlSource, ms.getSqlCommandType());
        // 复制其他属性...
        return builder.build();
    }
}

/**
 * SQL 性能监控插件
 * 
 * @author erik.zhou
 */
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "update",
        args = {MappedStatement.class, Object.class}
    ),
    @Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
    )
})
public class SqlPerformanceInterceptor implements Interceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(SqlPerformanceInterceptor.class);
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        try {
            return invocation.proceed();
        } finally {
            long endTime = System.currentTimeMillis();
            long sqlCost = endTime - startTime;
            
            MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
            String sqlId = mappedStatement.getId();
            
            // 记录慢 SQL
            if (sqlCost > 1000) {
                logger.warn("慢 SQL 警告: {} 耗时: {}ms", sqlId, sqlCost);
            } else {
                logger.info("SQL 执行: {} 耗时: {}ms", sqlId, sqlCost);
            }
        }
    }
}
```

---

## 6. 结果集映射 🔥

### 6.1 ResultSetHandler 处理结果集

```java
// DefaultResultSetHandler.handleResultSets()
@Override
public List<Object> handleResultSets(Statement stmt) throws SQLException {
    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());
    
    final List<Object> multipleResults = new ArrayList<>();
    
    int resultSetCount = 0;
    ResultSetWrapper rsw = getFirstResultSet(stmt);
    
    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    int resultMapCount = resultMaps.size();
    validateResultMapsCount(rsw, resultMapCount);
    
    while (rsw != null && resultMapCount > resultSetCount) {
        ResultMap resultMap = resultMaps.get(resultSetCount);
        // 处理结果集
        handleResultSet(rsw, resultMap, multipleResults, null);
        rsw = getNextResultSet(stmt);
        cleanUpAfterHandlingResultSet();
        resultSetCount++;
    }
    
    String[] resultSets = mappedStatement.getResultSets();
    if (resultSets != null) {
        while (rsw != null && resultSetCount < resultSets.length) {
            ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
            if (parentMapping != null) {
                String nestedResultMapId = parentMapping.getNestedResultMapId();
                ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                handleResultSet(rsw, resultMap, null, parentMapping);
            }
            rsw = getNextResultSet(stmt);
            cleanUpAfterHandlingResultSet();
            resultSetCount++;
        }
    }
    
    return collapseSingleResultList(multipleResults);
}

// 处理单个结果集
private void handleResultSet(ResultSetWrapper rsw, ResultMap resultMap, 
    List<Object> multipleResults, ResultMapping parentMapping) throws SQLException {
    try {
        if (parentMapping != null) {
            handleRowValues(rsw, resultMap, null, RowBounds.DEFAULT, parentMapping);
        } else {
            if (resultHandler == null) {
                DefaultResultHandler defaultResultHandler = new DefaultResultHandler(objectFactory);
                handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);
                multipleResults.add(defaultResultHandler.getResultList());
            } else {
                handleRowValues(rsw, resultMap, resultHandler, rowBounds, null);
            }
        }
    } finally {
        closeResultSet(rsw.getResultSet());
    }
}
```

### 6.2 简单映射

```java
// DefaultResultSetHandler.handleRowValues()
public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, 
    ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) 
    throws SQLException {
    if (resultMap.hasNestedResultMaps()) {
        ensureNoRowBounds();
        checkResultHandler();
        // 处理嵌套映射
        handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
    } else {
        // 处理简单映射
        handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
    }
}

// 处理简单映射
private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap, 
    ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) 
    throws SQLException {
    DefaultResultContext<Object> resultContext = new DefaultResultContext<>();
    ResultSet resultSet = rsw.getResultSet();
    // 跳过 offset 行
    skipRows(resultSet, rowBounds);
    // 处理每一行
    while (shouldProcessMoreRows(resultContext, rowBounds) && !resultSet.isClosed() && resultSet.next()) {
        ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(resultSet, resultMap, null);
        // 获取行值
        Object rowValue = getRowValue(rsw, discriminatedResultMap, null);
        // 存储对象
        storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);
    }
}

// 获取行值
private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap, String columnPrefix) 
    throws SQLException {
    final ResultLoaderMap lazyLoader = new ResultLoaderMap();
    // 创建结果对象
    Object rowValue = createResultObject(rsw, resultMap, lazyLoader, columnPrefix);
    if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
        final MetaObject metaObject = configuration.newMetaObject(rowValue);
        boolean foundValues = this.useConstructorMappings;
        if (shouldApplyAutomaticMappings(resultMap, false)) {
            // 自动映射
            foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;
        }
        // 属性映射
        foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, columnPrefix) || foundValues;
        foundValues = lazyLoader.size() > 0 || foundValues;
        rowValue = foundValues || configuration.isReturnInstanceForEmptyRow() ? rowValue : null;
    }
    return rowValue;
}
```

### 6.3 嵌套映射

```java
// 处理嵌套映射
private void handleRowValuesForNestedResultMap(ResultSetWrapper rsw, ResultMap resultMap, 
    ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) 
    throws SQLException {
    final DefaultResultContext<Object> resultContext = new DefaultResultContext<>();
    ResultSet resultSet = rsw.getResultSet();
    skipRows(resultSet, rowBounds);
    Object rowValue = previousRowValue;
    while (shouldProcessMoreRows(resultContext, rowBounds) && !resultSet.isClosed() && resultSet.next()) {
        final ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(resultSet, resultMap, null);
        // 创建缓存 Key
        final CacheKey rowKey = createRowKey(discriminatedResultMap, rsw, null);
        Object partialObject = nestedResultObjects.get(rowKey);
        if (mappedStatement.isResultOrdered()) {
            if (partialObject == null && rowValue != null) {
                nestedResultObjects.clear();
                storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);
            }
            rowValue = getRowValue(rsw, discriminatedResultMap, rowKey, null, partialObject);
        } else {
            rowValue = getRowValue(rsw, discriminatedResultMap, rowKey, null, partialObject);
            if (partialObject == null) {
                storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);
            }
        }
    }
    if (rowValue != null && mappedStatement.isResultOrdered() && shouldProcessMoreRows(resultContext, rowBounds)) {
        storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);
        previousRowValue = null;
    } else if (rowValue != null) {
        previousRowValue = rowValue;
    }
}
```

---

## 7. 动态 SQL 解析

### 7.1 动态 SQL 节点

```java
// SqlNode 接口
public interface SqlNode {
    boolean apply(DynamicContext context);
}

// IfSqlNode - <if> 标签
public class IfSqlNode implements SqlNode {
    private final ExpressionEvaluator evaluator;
    private final String test;
    private final SqlNode contents;
    
    public IfSqlNode(SqlNode contents, String test) {
        this.test = test;
        this.contents = contents;
        this.evaluator = new ExpressionEvaluator();
    }
    
    @Override
    public boolean apply(DynamicContext context) {
        // 判断条件是否成立
        if (evaluator.evaluateBoolean(test, context.getBindings())) {
            // 应用子节点
            contents.apply(context);
            return true;
        }
        return false;
    }
}

// ForEachSqlNode - <foreach> 标签
public class ForEachSqlNode implements SqlNode {
    private final ExpressionEvaluator evaluator;
    private final String collectionExpression;
    private final SqlNode contents;
    private final String open;
    private final String close;
    private final String separator;
    private final String item;
    private final String index;
    
    @Override
    public boolean apply(DynamicContext context) {
        Map<String, Object> bindings = context.getBindings();
        // 获取集合
        final Iterable<?> iterable = evaluator.evaluateIterable(collectionExpression, bindings);
        if (!iterable.iterator().hasNext()) {
            return true;
        }
        boolean first = true;
        // 添加 open
        applyOpen(context);
        int i = 0;
        for (Object o : iterable) {
            DynamicContext oldContext = context;
            if (first || separator == null) {
                context = new PrefixedContext(context, "");
            } else {
                context = new PrefixedContext(context, separator);
            }
            int uniqueNumber = context.getUniqueNumber();
            // 绑定 item 和 index
            if (o instanceof Map.Entry) {
                @SuppressWarnings("unchecked")
                Map.Entry<Object, Object> mapEntry = (Map.Entry<Object, Object>) o;
                applyIndex(context, mapEntry.getKey(), uniqueNumber);
                applyItem(context, mapEntry.getValue(), uniqueNumber);
            } else {
                applyIndex(context, i, uniqueNumber);
                applyItem(context, o, uniqueNumber);
            }
            // 应用子节点
            contents.apply(new FilteredDynamicContext(configuration, context, index, item, uniqueNumber));
            if (first) {
                first = !((PrefixedContext) context).isPrefixApplied();
            }
            context = oldContext;
            i++;
        }
        // 添加 close
        applyClose(context);
        context.getBindings().remove(item);
        context.getBindings().remove(index);
        return true;
    }
}
```


### 7.2 动态 SQL 解析流程

```java
// XMLScriptBuilder.parseScriptNode()
public SqlSource parseScriptNode() {
    // 解析动态标签
    MixedSqlNode rootSqlNode = parseDynamicTags(context);
    SqlSource sqlSource;
    if (isDynamic) {
        // 动态 SQL
        sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
    } else {
        // 静态 SQL
        sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);
    }
    return sqlSource;
}

// 解析动态标签
protected MixedSqlNode parseDynamicTags(XNode node) {
    List<SqlNode> contents = new ArrayList<>();
    NodeList children = node.getNode().getChildNodes();
    for (int i = 0; i < children.getLength(); i++) {
        XNode child = node.newXNode(children.item(i));
        if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE 
            || child.getNode().getNodeType() == Node.TEXT_NODE) {
            String data = child.getStringBody("");
            TextSqlNode textSqlNode = new TextSqlNode(data);
            if (textSqlNode.isDynamic()) {
                contents.add(textSqlNode);
                isDynamic = true;
            } else {
                contents.add(new StaticTextSqlNode(data));
            }
        } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) {
            String nodeName = child.getNode().getNodeName();
            // 获取节点处理器
            NodeHandler handler = nodeHandlerMap.get(nodeName);
            if (handler == null) {
                throw new BuilderException("Unknown element <" + nodeName + "> in SQL statement.");
            }
            // 处理节点
            handler.handleNode(child, contents);
            isDynamic = true;
        }
    }
    return new MixedSqlNode(contents);
}

// DynamicSqlSource.getBoundSql()
@Override
public BoundSql getBoundSql(Object parameterObject) {
    DynamicContext context = new DynamicContext(configuration, parameterObject);
    // 应用所有 SqlNode，生成完整 SQL
    rootSqlNode.apply(context);
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();
    // 解析 #{} 参数
    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    // 添加附加参数
    context.getBindings().forEach(boundSql::setAdditionalParameter);
    return boundSql;
}
```

---

## 8. TypeHandler 类型处理

### 8.1 TypeHandler 接口

```java
public interface TypeHandler<T> {
    // 设置参数
    void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
    
    // 获取结果
    T getResult(ResultSet rs, String columnName) throws SQLException;
    T getResult(ResultSet rs, int columnIndex) throws SQLException;
    T getResult(CallableStatement cs, int columnIndex) throws SQLException;
}

// 示例：StringTypeHandler
public class StringTypeHandler extends BaseTypeHandler<String> {
    
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType)
        throws SQLException {
        ps.setString(i, parameter);
    }
    
    @Override
    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getString(columnName);
    }
    
    @Override
    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getString(columnIndex);
    }
    
    @Override
    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getString(columnIndex);
    }
}
```

### 8.2 自定义 TypeHandler

```java
/**
 * JSON 类型处理器
 * 
 * @author erik.zhou
 */
@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(Object.class)
public class JsonTypeHandler extends BaseTypeHandler<Object> {
    
    private static final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) 
        throws SQLException {
        try {
            ps.setString(i, objectMapper.writeValueAsString(parameter));
        } catch (JsonProcessingException e) {
            throw new SQLException("Error converting object to JSON", e);
        }
    }
    
    @Override
    public Object getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String json = rs.getString(columnName);
        return parseJson(json);
    }
    
    @Override
    public Object getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        String json = rs.getString(columnIndex);
        return parseJson(json);
    }
    
    @Override
    public Object getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        String json = cs.getString(columnIndex);
        return parseJson(json);
    }
    
    private Object parseJson(String json) throws SQLException {
        if (json == null || json.isEmpty()) {
            return null;
        }
        try {
            return objectMapper.readValue(json, Object.class);
        } catch (JsonProcessingException e) {
            throw new SQLException("Error parsing JSON", e);
        }
    }
}

// 注册 TypeHandler
<typeHandlers>
    <typeHandler handler="com.example.handler.JsonTypeHandler"/>
</typeHandlers>

// 或者在字段上指定
<result column="extra_info" property="extraInfo" 
    typeHandler="com.example.handler.JsonTypeHandler"/>
```

---

## 9. 核心设计模式

### 9.1 建造者模式（Builder Pattern）

```java
// SqlSessionFactoryBuilder
public class SqlSessionFactoryBuilder {
    public SqlSessionFactory build(InputStream inputStream) {
        return build(inputStream, null, null);
    }
    
    public SqlSessionFactory build(InputStream inputStream, String environment) {
        return build(inputStream, environment, null);
    }
    
    public SqlSessionFactory build(InputStream inputStream, Properties properties) {
        return build(inputStream, null, properties);
    }
    
    public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
        try {
            XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
            return build(parser.parse());
        } catch (Exception e) {
            throw ExceptionFactory.wrapException("Error building SqlSession.", e);
        }
    }
    
    public SqlSessionFactory build(Configuration config) {
        return new DefaultSqlSessionFactory(config);
    }
}
```

### 9.2 工厂模式（Factory Pattern）

```java
// SqlSessionFactory
public interface SqlSessionFactory {
    SqlSession openSession();
    SqlSession openSession(boolean autoCommit);
    SqlSession openSession(Connection connection);
    SqlSession openSession(TransactionIsolationLevel level);
    SqlSession openSession(ExecutorType execType);
    SqlSession openSession(ExecutorType execType, boolean autoCommit);
    SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level);
    SqlSession openSession(ExecutorType execType, Connection connection);
    Configuration getConfiguration();
}

// DefaultSqlSessionFactory
public class DefaultSqlSessionFactory implements SqlSessionFactory {
    
    @Override
    public SqlSession openSession() {
        return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
    }
    
    private SqlSession openSessionFromDataSource(ExecutorType execType, 
        TransactionIsolationLevel level, boolean autoCommit) {
        Transaction tx = null;
        try {
            final Environment environment = configuration.getEnvironment();
            final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
            tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
            final Executor executor = configuration.newExecutor(tx, execType);
            return new DefaultSqlSession(configuration, executor, autoCommit);
        } catch (Exception e) {
            closeTransaction(tx);
            throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
        } finally {
            ErrorContext.instance().reset();
        }
    }
}
```

### 9.3 代理模式（Proxy Pattern）

```java
// MapperProxy - Mapper 接口的动态代理
public class MapperProxy<T> implements InvocationHandler, Serializable {
    
    private final SqlSession sqlSession;
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethodInvoker> methodCache;
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);
            } else {
                return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
            }
        } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
        }
    }
}

// Plugin - 插件的动态代理
public class Plugin implements InvocationHandler {
    
    private final Object target;
    private final Interceptor interceptor;
    private final Map<Class<?>, Set<Method>> signatureMap;
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            Set<Method> methods = signatureMap.get(method.getDeclaringClass());
            if (methods != null && methods.contains(method)) {
                return interceptor.intercept(new Invocation(target, method, args));
            }
            return method.invoke(target, args);
        } catch (Exception e) {
            throw ExceptionUtil.unwrapThrowable(e);
        }
    }
}
```

### 9.4 模板方法模式（Template Method Pattern）

```java
// BaseExecutor
public abstract class BaseExecutor implements Executor {
    
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) 
        throws SQLException {
        // 模板方法
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
            handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
            list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
        return list;
    }
    
    private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) 
        throws SQLException {
        List<E> list;
        localCache.putObject(key, EXECUTION_PLACEHOLDER);
        try {
            // 调用子类实现
            list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
        } finally {
            localCache.removeObject(key);
        }
        localCache.putObject(key, list);
        return list;
    }
    
    // 抽象方法，由子类实现
    protected abstract <E> List<E> doQuery(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException;
}

// SimpleExecutor
public class SimpleExecutor extends BaseExecutor {
    
    @Override
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        Statement stmt = null;
        try {
            Configuration configuration = ms.getConfiguration();
            StatementHandler handler = configuration.newStatementHandler(
                wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
            stmt = prepareStatement(handler, ms.getStatementLog());
            return handler.query(stmt, resultHandler);
        } finally {
            closeStatement(stmt);
        }
    }
}
```

### 9.5 装饰器模式（Decorator Pattern）

```java
// Cache 装饰器
public class LruCache implements Cache {
    private final Cache delegate;
    private Map<Object, Object> keyMap;
    private Object eldestKey;
    
    public LruCache(Cache delegate) {
        this.delegate = delegate;
        setSize(1024);
    }
    
    @Override
    public void putObject(Object key, Object value) {
        delegate.putObject(key, value);
        cycleKeyList(key);
    }
    
    @Override
    public Object getObject(Object key) {
        keyMap.get(key);
        return delegate.getObject(key);
    }
    
    // 其他方法委托给 delegate
}

// CachingExecutor 装饰 BaseExecutor
public class CachingExecutor implements Executor {
    private final Executor delegate;
    private final TransactionalCacheManager tcm = new TransactionalCacheManager();
    
    public CachingExecutor(Executor delegate) {
        this.delegate = delegate;
        delegate.setExecutorWrapper(this);
    }
    
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, 
        RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
        throws SQLException {
        Cache cache = ms.getCache();
        if (cache != null) {
            // 使用缓存
            // ...
        }
        // 委托给 delegate
        return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
}
```

---

## 10. 最佳实践与优化建议

### 10.1 性能优化

```java
/**
 * 1. 合理使用缓存
 */
// 一级缓存默认开启，注意事务范围
// 二级缓存需要手动开启
<cache eviction="LRU" flushInterval="60000" size="512" readOnly="true"/>

/**
 * 2. 批量操作
 */
SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH);
try {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    for (User user : userList) {
        mapper.insertUser(user);
    }
    sqlSession.commit();
} finally {
    sqlSession.close();
}

/**
 * 3. 延迟加载
 */
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>

<resultMap id="userResultMap" type="User">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <!-- 延迟加载关联对象 -->
    <association property="profile" column="id" 
        select="selectUserProfile" fetchType="lazy"/>
</resultMap>

/**
 * 4. 避免 N+1 查询
 */
// 使用嵌套查询
<resultMap id="userResultMap" type="User">
    <id property="id" column="id"/>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNo" column="order_no"/>
    </collection>
</resultMap>

<select id="selectUserWithOrders" resultMap="userResultMap">
    SELECT u.*, o.id as order_id, o.order_no
    FROM user u
    LEFT JOIN `order` o ON u.id = o.user_id
    WHERE u.id = #{id}
</select>
```

### 10.2 常见问题

```java
/**
 * 1. 一级缓存导致的脏读
 * 
 * @author erik.zhou
 */
// 问题：同一个 SqlSession 中，第一次查询后，其他地方修改了数据，第二次查询仍然返回缓存数据
// 解决：
// 方案1：每次查询使用新的 SqlSession
// 方案2：手动清空缓存
sqlSession.clearCache();
// 方案3：设置 flushCache="true"
<select id="selectUser" flushCache="true">
    SELECT * FROM user WHERE id = #{id}
</select>

/**
 * 2. 二级缓存的线程安全问题
 * 
 * @author erik.zhou
 */
// 问题：二级缓存是跨 SqlSession 的，可能导致并发问题
// 解决：
// 方案1：使用 readOnly="true"（返回缓存对象的引用，不可修改）
<cache readOnly="true"/>
// 方案2：实体类实现 Serializable（返回缓存对象的副本）
public class User implements Serializable {
    // ...
}

/**
 * 3. 动态 SQL 中的空指针
 * 
 * @author erik.zhou
 */
// 问题：参数为 null 时，动态 SQL 判断出错
// 解决：使用 OGNL 表达式的安全导航
<if test="username != null and username != ''">
    AND username = #{username}
</if>

/**
 * 4. 大数据量查询内存溢出
 * 
 * @author erik.zhou
 */
// 问题：一次性加载大量数据到内存
// 解决：使用游标
<select id="selectUsers" resultType="User" fetchSize="1000">
    SELECT * FROM user
</select>

try (Cursor<User> cursor = mapper.selectUsers()) {
    for (User user : cursor) {
        // 处理每条数据
    }
}
```

---

## 11. 总结

MyBatis 源码的核心流程：

1. **初始化阶段**：解析配置文件，构建 Configuration 对象
2. **代理阶段**：通过动态代理创建 Mapper 接口的实现
3. **执行阶段**：Executor → StatementHandler → ParameterHandler → JDBC → ResultSetHandler
4. **缓存机制**：一级缓存（SqlSession 级别）+ 二级缓存（Mapper 级别）
5. **插件机制**：基于责任链模式的拦截器
6. **类型处理**：TypeHandler 实现 Java 类型与 JDBC 类型的转换

掌握这些核心原理，可以帮助我们：
- 更好地使用 MyBatis
- 排查和解决问题
- 进行性能优化
- 开发自定义插件和扩展

---

## 📚 参考资料

- [MyBatis 官方文档](https://mybatis.org/mybatis-3/zh/index.html)
- [MyBatis GitHub](https://github.com/mybatis/mybatis-3)
- 《MyBatis 技术内幕》
- 《MyBatis 从入门到精通》
