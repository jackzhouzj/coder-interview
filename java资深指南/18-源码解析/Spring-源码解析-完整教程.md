# Spring 源码解析 - 完整教程

> 深入理解 Spring 框架的核心原理和设计思想
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | Spring Framework |
| **当前版本** | 6.1.x (Spring Boot 3.2.x) |
| **源码地址** | https://github.com/spring-projects/spring-framework |
| **学习难度** | ⭐⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐⭐ |
| **预计时长** | 40-60 小时 |
| **前置知识** | Java 基础、设计模式、反射、代理 |

## 🎯 学习目标

- [ ] 理解 Spring IoC 容器的启动流程
- [ ] 掌握 Bean 的完整生命周期
- [ ] 深入理解 AOP 的实现原理
- [ ] 掌握 Spring 事务管理机制
- [ ] 理解 Spring MVC 的请求处理流程
- [ ] 学习 Spring 中的设计模式应用
- [ ] 能够解决 Spring 相关的复杂问题

## 📖 目录

1. [Spring 整体架构](#1-spring-整体架构)
2. [IoC 容器启动流程](#2-ioc-容器启动流程)
3. [Bean 生命周期](#3-bean-生命周期)
4. [依赖注入原理](#4-依赖注入原理)
5. [AOP 实现原理](#5-aop-实现原理)
6. [事务管理机制](#6-事务管理机制)
7. [Spring MVC 原理](#7-spring-mvc-原理)
8. [设计模式应用](#8-设计模式应用)

---

## 1. Spring 整体架构

### 1.1 核心模块

```
spring-framework/
├── spring-core          # 核心工具类
├── spring-beans         # Bean 定义和管理
├── spring-context       # 应用上下文
├── spring-aop           # AOP 实现
├── spring-aspects       # AspectJ 集成
├── spring-tx            # 事务管理
├── spring-web           # Web 基础
├── spring-webmvc        # Spring MVC
└── spring-jdbc          # JDBC 支持
```

### 1.2 核心概念

**IoC (Inversion of Control) - 控制反转**
- 对象的创建和管理由 Spring 容器负责
- 降低代码耦合度

**DI (Dependency Injection) - 依赖注入**
- 通过构造器、Setter、字段注入依赖
- IoC 的具体实现方式

**AOP (Aspect Oriented Programming) - 面向切面编程**
- 横切关注点的模块化
- 通过动态代理实现

**Bean**
- Spring 管理的对象
- 由 IoC 容器创建、配置和管理


### 1.3 核心接口

```java
// BeanFactory - Bean 工厂接口
public interface BeanFactory {
    Object getBean(String name) throws BeansException;
    <T> T getBean(String name, Class<T> requiredType) throws BeansException;
    boolean containsBean(String name);
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
}

// ApplicationContext - 应用上下文（BeanFactory 的子接口）
public interface ApplicationContext extends 
    EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
    MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    // 提供更多企业级功能
}

// BeanDefinition - Bean 定义
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    String getBeanClassName();
    String getScope();
    boolean isSingleton();
    boolean isPrototype();
}
```

**BeanFactory vs ApplicationContext**

| 特性 | BeanFactory | ApplicationContext |
|------|-------------|-------------------|
| Bean 实例化 | 延迟加载 | 立即加载 |
| 国际化 | 不支持 | 支持 |
| 事件发布 | 不支持 | 支持 |
| AOP | 需手动配置 | 自动支持 |
| 使用场景 | 资源受限环境 | 企业应用（推荐） |

---

## 2. IoC 容器启动流程 🔥

### 2.1 启动流程概览

```java
// Spring Boot 启动入口
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 底层调用 AbstractApplicationContext.refresh()
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 1. 准备刷新上下文
        prepareRefresh();
        
        // 2. 获取 BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // 3. 准备 BeanFactory
        prepareBeanFactory(beanFactory);
        
        try {
            // 4. 后置处理 BeanFactory
            postProcessBeanFactory(beanFactory);
            
            // 5. 调用 BeanFactory 后置处理器
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // 6. 注册 Bean 后置处理器
            registerBeanPostProcessors(beanFactory);
            
            // 7. 初始化消息源
            initMessageSource();
            
            // 8. 初始化事件广播器
            initApplicationEventMulticaster();
            
            // 9. 刷新（模板方法，子类实现）
            onRefresh();
            
            // 10. 注册监听器
            registerListeners();
            
            // 11. 实例化所有非懒加载的单例 Bean
            finishBeanFactoryInitialization(beanFactory);
            
            // 12. 完成刷新
            finishRefresh();
        } catch (BeansException ex) {
            // 销毁已创建的单例 Bean
            destroyBeans();
            cancelRefresh(ex);
            throw ex;
        }
    }
}
```

### 2.2 核心步骤详解

#### 步骤 1: prepareRefresh() - 准备刷新

```java
protected void prepareRefresh() {
    // 设置启动时间
    this.startupDate = System.currentTimeMillis();
    // 设置关闭标志为 false
    this.closed.set(false);
    // 设置激活标志为 true
    this.active.set(true);
    
    // 初始化属性源（可由子类覆盖）
    initPropertySources();
    
    // 验证必需的属性
    getEnvironment().validateRequiredProperties();
    
    // 存储预刷新的 ApplicationListener
    this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### 步骤 2: obtainFreshBeanFactory() - 获取 BeanFactory

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    // 刷新 BeanFactory（创建或刷新）
    refreshBeanFactory();
    // 返回 BeanFactory
    return getBeanFactory();
}

// AbstractRefreshableApplicationContext 实现
protected final void refreshBeanFactory() throws BeansException {
    // 如果已存在 BeanFactory，先销毁
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    
    try {
        // 创建 DefaultListableBeanFactory
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        
        // 定制 BeanFactory（是否允许覆盖、循环依赖）
        customizeBeanFactory(beanFactory);
        
        // 加载 BeanDefinition
        loadBeanDefinitions(beanFactory);
        
        this.beanFactory = beanFactory;
    } catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source", ex);
    }
}
```

#### 步骤 5: invokeBeanFactoryPostProcessors() - 调用 BeanFactory 后置处理器 🔥

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 委托给 PostProcessorRegistrationDelegate
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, 
        getBeanFactoryPostProcessors());
}

// 核心逻辑
public static void invokeBeanFactoryPostProcessors(
        ConfigurableListableBeanFactory beanFactory, 
        List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    
    // 1. 先处理 BeanDefinitionRegistryPostProcessor
    // 这个接口可以注册新的 BeanDefinition
    
    // 2. 再处理 BeanFactoryPostProcessor
    // 这个接口可以修改 BeanDefinition
    
    // 重要实现：ConfigurationClassPostProcessor
    // 负责处理 @Configuration、@ComponentScan、@Import 等注解
}
```

**ConfigurationClassPostProcessor 的作用**：
- 解析 `@Configuration` 类
- 处理 `@ComponentScan` 扫描包
- 处理 `@Import` 导入配置
- 处理 `@Bean` 方法
- 这是 Spring Boot 自动配置的核心！

#### 步骤 11: finishBeanFactoryInitialization() - 实例化单例 Bean 🔥

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 初始化转换服务
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }
    
    // 冻结配置（不再修改 BeanDefinition）
    beanFactory.freezeConfiguration();
    
    // 实例化所有非懒加载的单例 Bean
    beanFactory.preInstantiateSingletons();
}

// DefaultListableBeanFactory 实现
public void preInstantiateSingletons() throws BeansException {
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    
    // 遍历所有 BeanDefinition
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        
        // 非抽象、单例、非懒加载
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                // 处理 FactoryBean
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                // ...
            } else {
                // 普通 Bean，调用 getBean() 创建
                getBean(beanName);
            }
        }
    }
}
```

### 2.3 流程图

```
启动 Spring 应用
    ↓
prepareRefresh() - 准备刷新
    ↓
obtainFreshBeanFactory() - 创建 BeanFactory
    ↓
prepareBeanFactory() - 配置 BeanFactory
    ↓
invokeBeanFactoryPostProcessors() - 处理 @Configuration 等
    ↓
registerBeanPostProcessors() - 注册 Bean 后置处理器
    ↓
finishBeanFactoryInitialization() - 实例化所有单例 Bean
    ↓
finishRefresh() - 完成刷新，发布事件
    ↓
Spring 容器启动完成
```

---

## 3. Bean 生命周期 🔥

### 3.1 完整生命周期

```java
// Bean 生命周期的完整流程
1. 实例化 Bean (Instantiation)
   ↓
2. 设置属性 (Populate Properties)
   ↓
3. 调用 BeanNameAware.setBeanName()
   ↓
4. 调用 BeanFactoryAware.setBeanFactory()
   ↓
5. 调用 ApplicationContextAware.setApplicationContext()
   ↓
6. 调用 BeanPostProcessor.postProcessBeforeInitialization()
   ↓
7. 调用 @PostConstruct 注解的方法
   ↓
8. 调用 InitializingBean.afterPropertiesSet()
   ↓
9. 调用自定义的 init-method
   ↓
10. 调用 BeanPostProcessor.postProcessAfterInitialization()
   ↓
Bean 可以使用了
   ↓
容器关闭
   ↓
11. 调用 @PreDestroy 注解的方法
   ↓
12. 调用 DisposableBean.destroy()
   ↓
13. 调用自定义的 destroy-method
```


### 3.2 Bean 创建核心源码

```java
// AbstractAutowireCapableBeanFactory.doCreateBean()
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    // 1. 实例化 Bean
    BeanWrapper instanceWrapper = createBeanInstance(beanName, mbd, args);
    Object bean = instanceWrapper.getWrappedInstance();
    
    // 2. 允许后置处理器修改 BeanDefinition
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            mbd.postProcessed = true;
        }
    }
    
    // 3. 提前暴露单例 Bean（解决循环依赖）
    boolean earlySingletonExposure = (mbd.isSingleton() && 
        this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
    
    // 4. 填充属性（依赖注入）
    Object exposedObject = bean;
    populateBean(beanName, mbd, instanceWrapper);
    
    // 5. 初始化 Bean
    exposedObject = initializeBean(beanName, exposedObject, mbd);
    
    return exposedObject;
}

// 初始化 Bean
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    // 调用 Aware 接口
    invokeAwareMethods(beanName, bean);
    
    // 调用 BeanPostProcessor.postProcessBeforeInitialization()
    Object wrappedBean = applyBeanPostProcessorsBeforeInitialization(bean, beanName);
    
    // 调用初始化方法
    invokeInitMethods(beanName, wrappedBean, mbd);
    
    // 调用 BeanPostProcessor.postProcessAfterInitialization()
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    
    return wrappedBean;
}
```

### 3.3 循环依赖解决 🔥

**什么是循环依赖？**
```java
@Service
public class A {
    @Autowired
    private B b;  // A 依赖 B
}

@Service
public class B {
    @Autowired
    private A a;  // B 依赖 A
}
```

**Spring 如何解决？**

使用三级缓存：

```java
public class DefaultSingletonBeanRegistry {
    // 一级缓存：完整的单例 Bean
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
    
    // 二级缓存：早期的单例 Bean（已实例化，未初始化）
    private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
    
    // 三级缓存：单例工厂
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
}

// 获取单例 Bean
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 1. 从一级缓存获取
    Object singletonObject = this.singletonObjects.get(beanName);
    
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        // 2. 从二级缓存获取
        singletonObject = this.earlySingletonObjects.get(beanName);
        
        if (singletonObject == null && allowEarlyReference) {
            synchronized (this.singletonObjects) {
                // 3. 从三级缓存获取
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    // 放入二级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    // 从三级缓存移除
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

**循环依赖解决流程**：

```
1. 创建 A，实例化 A（未初始化）
2. 将 A 的工厂放入三级缓存
3. 填充 A 的属性，发现依赖 B
4. 创建 B，实例化 B（未初始化）
5. 将 B 的工厂放入三级缓存
6. 填充 B 的属性，发现依赖 A
7. 从三级缓存获取 A 的早期引用
8. B 初始化完成，放入一级缓存
9. A 获取到 B，继续初始化
10. A 初始化完成，放入一级缓存
```

**注意**：
- 只能解决单例 Bean 的循环依赖
- 不能解决构造器注入的循环依赖
- 不能解决 prototype 作用域的循环依赖

---

## 4. 依赖注入原理

### 4.1 注入方式

```java
// 1. 构造器注入（推荐）
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired  // Spring 4.3+ 可省略
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 2. Setter 注入
@Service
public class UserService {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 3. 字段注入（不推荐）
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

### 4.2 依赖注入源码

```java
// AbstractAutowireCapableBeanFactory.populateBean()
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // 1. 给 InstantiationAwareBeanPostProcessor 机会修改 Bean
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                return;
            }
        }
    }
    
    // 2. 获取属性值
    PropertyValues pvs = mbd.getPropertyValues();
    
    // 3. 自动装配（byName 或 byType）
    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }
    
    // 4. 处理 @Autowired、@Resource 等注解
    // 由 AutowiredAnnotationBeanPostProcessor 处理
    if (hasInstantiationAwareBeanPostProcessors()) {
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
            pvs = pvsToUse;
        }
    }
    
    // 5. 应用属性值
    applyPropertyValues(beanName, mbd, bw, pvs);
}
```

### 4.3 @Autowired 处理

```java
// AutowiredAnnotationBeanPostProcessor
public class AutowiredAnnotationBeanPostProcessor implements 
    InstantiationAwareBeanPostProcessor, BeanFactoryAware {
    
    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        // 1. 查找需要注入的元数据（字段、方法）
        InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
        
        // 2. 执行注入
        metadata.inject(bean, beanName, pvs);
        
        return pvs;
    }
    
    // 查找 @Autowired 注解的字段和方法
    private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, 
            @Nullable PropertyValues pvs) {
        // 从缓存获取
        InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
        if (metadata == null) {
            // 构建注入元数据
            metadata = buildAutowiringMetadata(clazz);
            this.injectionMetadataCache.put(cacheKey, metadata);
        }
        return metadata;
    }
    
    // 构建注入元数据
    private InjectionMetadata buildAutowiringMetadata(Class<?> clazz) {
        List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
        Class<?> targetClass = clazz;
        
        do {
            List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();
            
            // 处理字段
            ReflectionUtils.doWithLocalFields(targetClass, field -> {
                MergedAnnotation<?> ann = findAutowiredAnnotation(field);
                if (ann != null) {
                    if (Modifier.isStatic(field.getModifiers())) {
                        return;  // 忽略静态字段
                    }
                    boolean required = determineRequiredStatus(ann);
                    currElements.add(new AutowiredFieldElement(field, required));
                }
            });
            
            // 处理方法
            ReflectionUtils.doWithLocalMethods(targetClass, method -> {
                Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
                if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
                    return;
                }
                MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
                if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
                    if (Modifier.isStatic(method.getModifiers())) {
                        return;  // 忽略静态方法
                    }
                    boolean required = determineRequiredStatus(ann);
                    PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
                    currElements.add(new AutowiredMethodElement(method, required, pd));
                }
            });
            
            elements.addAll(0, currElements);
            targetClass = targetClass.getSuperclass();
        }
        while (targetClass != null && targetClass != Object.class);
        
        return InjectionMetadata.forElements(elements, clazz);
    }
}
```

---

## 5. AOP 实现原理 🔥

### 5.1 AOP 核心概念

```java
// 切面（Aspect）
@Aspect
@Component
public class LogAspect {
    
    // 切点（Pointcut）
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}
    
    // 前置通知（Before Advice）
    @Before("serviceLayer()")
    public void before(JoinPoint joinPoint) {
        System.out.println("Before: " + joinPoint.getSignature());
    }
    
    // 后置通知（After Advice）
    @After("serviceLayer()")
    public void after(JoinPoint joinPoint) {
        System.out.println("After: " + joinPoint.getSignature());
    }
    
    // 返回通知（AfterReturning Advice）
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("AfterReturning: " + result);
    }
    
    // 异常通知（AfterThrowing Advice）
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Exception ex) {
        System.out.println("AfterThrowing: " + ex.getMessage());
    }
    
    // 环绕通知（Around Advice）
    @Around("serviceLayer()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Around Before");
        Object result = pjp.proceed();  // 执行目标方法
        System.out.println("Around After");
        return result;
    }
}
```

### 5.2 AOP 代理创建

```java
// AbstractAutoProxyCreator.postProcessAfterInitialization()
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            // 如果需要代理，创建代理对象
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    // 1. 获取该 Bean 的所有 Advisor（增强器）
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(
        bean.getClass(), beanName, null);
    
    // 2. 如果有 Advisor，创建代理
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 创建代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }
    
    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

### 5.3 JDK 动态代理 vs CGLIB 代理

```java
// DefaultAopProxyFactory.createAopProxy()
@Override
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: " +
                "Either an interface or a target is required for proxy creation.");
        }
        // 如果目标类是接口或者已经是代理类，使用 JDK 动态代理
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        // 否则使用 CGLIB 代理
        return new ObjenesisCglibAopProxy(config);
    } else {
        // 默认使用 JDK 动态代理
        return new JdkDynamicAopProxy(config);
    }
}

// JDK 动态代理
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
    
    @Override
    public Object getProxy(@Nullable ClassLoader classLoader) {
        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
        findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object oldProxy = null;
        boolean setProxyContext = false;
        
        TargetSource targetSource = this.advised.targetSource;
        Object target = null;
        
        try {
            // 获取拦截器链
            List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
            
            if (chain.isEmpty()) {
                // 没有拦截器，直接调用目标方法
                Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
                retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
            } else {
                // 创建方法调用
                MethodInvocation invocation = new ReflectiveMethodInvocation(
                    proxy, target, method, args, targetClass, chain);
                // 执行拦截器链
                retVal = invocation.proceed();
            }
            
            return retVal;
        } finally {
            if (target != null && !targetSource.isStatic()) {
                targetSource.releaseTarget(target);
            }
            if (setProxyContext) {
                AopContext.setCurrentProxy(oldProxy);
            }
        }
    }
}
```

---

## 6. 事务管理机制 🔥

### 6.1 事务管理器

```java
// PlatformTransactionManager 接口
public interface PlatformTransactionManager extends TransactionManager {
    // 获取事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) 
        throws TransactionException;
    
    // 提交事务
    void commit(TransactionStatus status) throws TransactionException;
    
    // 回滚事务
    void rollback(TransactionStatus status) throws TransactionException;
}

// DataSourceTransactionManager
public class DataSourceTransactionManager extends AbstractPlatformTransactionManager 
    implements ResourceTransactionManager, InitializingBean {
    
    @Override
    protected void doBegin(Object transaction, TransactionDefinition definition) {
        DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
        Connection con = null;
        
        try {
            if (!txObject.hasConnectionHolder() ||
                txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
                // 获取数据库连接
                Connection newCon = obtainDataSource().getConnection();
                txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
            }
            
            txObject.getConnectionHolder().setSynchronizedWithTransaction(true);
            con = txObject.getConnectionHolder().getConnection();
            
            // 设置隔离级别
            Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);
            txObject.setPreviousIsolationLevel(previousIsolationLevel);
            txObject.setReadOnly(definition.isReadOnly());
            
            // 关闭自动提交
            if (con.getAutoCommit()) {
                txObject.setMustRestoreAutoCommit(true);
                con.setAutoCommit(false);
            }
            
            // 绑定连接到线程
            if (txObject.isNewConnectionHolder()) {
                TransactionSynchronizationManager.bindResource(obtainDataSource(), txObject.getConnectionHolder());
            }
        } catch (Throwable ex) {
            if (txObject.isNewConnectionHolder()) {
                DataSourceUtils.releaseConnection(con, obtainDataSource());
                txObject.setConnectionHolder(null, false);
            }
            throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);
        }
    }
    
    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
        Connection con = txObject.getConnectionHolder().getConnection();
        try {
            con.commit();
        } catch (SQLException ex) {
            throw new TransactionSystemException("Could not commit JDBC transaction", ex);
        }
    }
    
    @Override
    protected void doRollback(DefaultTransactionStatus status) {
        DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
        Connection con = txObject.getConnectionHolder().getConnection();
        try {
            con.rollback();
        } catch (SQLException ex) {
            throw new TransactionSystemException("Could not roll back JDBC transaction", ex);
        }
    }
}
```

### 6.2 @Transactional 注解处理

```java
// TransactionInterceptor - 事务拦截器
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable {
    
    @Override
    @Nullable
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Class<?> targetClass = (invocation.getThis() != null ? 
            AopUtils.getTargetClass(invocation.getThis()) : null);
        
        // 执行事务方法
        return invokeWithinTransaction(invocation.getMethod(), targetClass, invocation::proceed);
    }
}

// TransactionAspectSupport.invokeWithinTransaction()
@Nullable
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
    final InvocationCallback invocation) throws Throwable {
    
    // 获取事务属性
    TransactionAttributeSource tas = getTransactionAttributeSource();
    final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
    final TransactionManager tm = determineTransactionManager(txAttr);
    
    PlatformTransactionManager ptm = asPlatformTransactionManager(tm);
    final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);
    
    if (txAttr == null || !(ptm instanceof CallbackPreferringPlatformTransactionManager)) {
        // 标准事务处理
        TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);
        
        Object retVal;
        try {
            // 执行目标方法
            retVal = invocation.proceedWithInvocation();
        } catch (Throwable ex) {
            // 异常回滚
            completeTransactionAfterThrowing(txInfo, ex);
            throw ex;
        } finally {
            cleanupTransactionInfo(txInfo);
        }
        
        // 提交事务
        commitTransactionAfterReturning(txInfo);
        return retVal;
    }
}
```

### 6.3 事务传播行为

```java
/**
 * 事务传播行为
 * 
 * @author erik.zhou
 */
public enum Propagation {
    // REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务
    REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
    
    // SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务方式执行
    SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),
    
    // MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常
    MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),
    
    // REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起
    REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),
    
    // NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，则把当前事务挂起
    NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),
    
    // NEVER：以非事务方式执行，如果当前存在事务，则抛出异常
    NEVER(TransactionDefinition.PROPAGATION_NEVER),
    
    // NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行
    NESTED(TransactionDefinition.PROPAGATION_NESTED);
}

// 示例
@Service
public class UserService {
    
    @Autowired
    private OrderService orderService;
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void createUser(User user) {
        // 保存用户
        userDao.save(user);
        
        // 调用订单服务（加入当前事务）
        orderService.createOrder(user.getId());
    }
}

@Service
public class OrderService {
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createOrder(Long userId) {
        // 创建新事务，即使 createUser 回滚，订单也会提交
        Order order = new Order();
        order.setUserId(userId);
        orderDao.save(order);
    }
}
```

---

## 7. Spring MVC 原理 🔥

### 7.1 DispatcherServlet 请求处理流程

```java
// DispatcherServlet.doDispatch()
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        
        try {
            // 1. 检查是否是文件上传请求
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);
            
            // 2. 根据请求找到 Handler（Controller 方法）
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }
            
            // 3. 根据 Handler 找到 HandlerAdapter
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
            
            // 4. 执行拦截器的 preHandle 方法
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }
            
            // 5. 执行 Handler（Controller 方法）
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            
            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }
            
            applyDefaultViewName(processedRequest, mv);
            
            // 6. 执行拦截器的 postHandle 方法
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        } catch (Exception ex) {
            dispatchException = ex;
        }
        
        // 7. 处理结果（渲染视图）
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    } catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    } finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        } else {
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

### 7.2 HandlerMapping 映射处理

```java
// RequestMappingHandlerMapping
public class RequestMappingHandlerMapping extends RequestMappingInfoHandlerMapping 
    implements MatchableHandlerMapping, EmbeddedValueResolverAware {
    
    @Override
    protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
        // 获取请求路径
        String lookupPath = initLookupPath(request);
        this.mappingRegistry.acquireReadLock();
        try {
            // 查找匹配的 Handler
            HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
            return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
        } finally {
            this.mappingRegistry.releaseReadLock();
        }
    }
    
    @Nullable
    protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) 
        throws Exception {
        List<Match> matches = new ArrayList<>();
        // 直接匹配
        List<T> directPathMatches = this.mappingRegistry.getMappingsByDirectPath(lookupPath);
        if (directPathMatches != null) {
            addMatchingMappings(directPathMatches, matches, request);
        }
        if (matches.isEmpty()) {
            // 模式匹配
            addMatchingMappings(this.mappingRegistry.getRegistrations().keySet(), matches, request);
        }
        
        if (!matches.isEmpty()) {
            Match bestMatch = matches.get(0);
            if (matches.size() > 1) {
                Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
                matches.sort(comparator);
                bestMatch = matches.get(0);
            }
            request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.getHandlerMethod());
            handleMatch(bestMatch.mapping, lookupPath, request);
            return bestMatch.getHandlerMethod();
        } else {
            return handleNoMatch(this.mappingRegistry.getRegistrations().keySet(), lookupPath, request);
        }
    }
}
```

### 7.3 参数解析和返回值处理

```java
// RequestMappingHandlerAdapter.invokeHandlerMethod()
@Nullable
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
    HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
    
    ServletWebRequest webRequest = new ServletWebRequest(request, response);
    try {
        WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
        ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);
        
        ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
        if (this.argumentResolvers != null) {
            // 设置参数解析器
            invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
        }
        if (this.returnValueHandlers != null) {
            // 设置返回值处理器
            invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
        }
        invocableMethod.setDataBinderFactory(binderFactory);
        invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);
        
        ModelAndViewContainer mavContainer = new ModelAndViewContainer();
        mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
        modelFactory.initModel(webRequest, mavContainer, invocableMethod);
        mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);
        
        // 执行方法
        invocableMethod.invokeAndHandle(webRequest, mavContainer);
        if (asyncManager.isConcurrentHandlingStarted()) {
            return null;
        }
        
        return getModelAndView(mavContainer, modelFactory, webRequest);
    } finally {
        webRequest.requestCompleted();
    }
}
```

---

## 8. 设计模式应用

### 8.1 工厂模式

```java
// BeanFactory - 工厂模式
public interface BeanFactory {
    Object getBean(String name) throws BeansException;
    <T> T getBean(Class<T> requiredType) throws BeansException;
}

// FactoryBean - 工厂 Bean
public interface FactoryBean<T> {
    @Nullable
    T getObject() throws Exception;
    @Nullable
    Class<?> getObjectType();
    default boolean isSingleton() {
        return true;
    }
}
```

### 8.2 单例模式

```java
// DefaultSingletonBeanRegistry
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    // 单例对象缓存
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
    
    @Override
    public Object getSingleton(String beanName) {
        return getSingleton(beanName, true);
    }
    
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        // 从缓存获取
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            // 从早期引用缓存获取（解决循环依赖）
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                synchronized (this.singletonObjects) {
                    singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        singletonObject = this.earlySingletonObjects.get(beanName);
                        if (singletonObject == null) {
                            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                            if (singletonFactory != null) {
                                singletonObject = singletonFactory.getObject();
                                this.earlySingletonObjects.put(beanName, singletonObject);
                                this.singletonFactories.remove(beanName);
                            }
                        }
                    }
                }
            }
        }
        return singletonObject;
    }
}
```

### 8.3 模板方法模式

```java
// AbstractApplicationContext.refresh() - 模板方法
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 1. 准备刷新
        prepareRefresh();
        
        // 2. 获取 BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // 3. 准备 BeanFactory
        prepareBeanFactory(beanFactory);
        
        try {
            // 4. 后置处理 BeanFactory（子类扩展点）
            postProcessBeanFactory(beanFactory);
            
            // 5. 执行 BeanFactoryPostProcessor
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // 6. 注册 BeanPostProcessor
            registerBeanPostProcessors(beanFactory);
            
            // 7. 初始化消息源
            initMessageSource();
            
            // 8. 初始化事件广播器
            initApplicationEventMulticaster();
            
            // 9. 刷新（子类扩展点）
            onRefresh();
            
            // 10. 注册监听器
            registerListeners();
            
            // 11. 实例化所有非懒加载的单例 Bean
            finishBeanFactoryInitialization(beanFactory);
            
            // 12. 完成刷新
            finishRefresh();
        } catch (BeansException ex) {
            destroyBeans();
            cancelRefresh(ex);
            throw ex;
        } finally {
            resetCommonCaches();
        }
    }
}
```

### 8.4 观察者模式

```java
// ApplicationEvent - 事件
public abstract class ApplicationEvent extends EventObject {
    private final long timestamp;
    
    public ApplicationEvent(Object source) {
        super(source);
        this.timestamp = System.currentTimeMillis();
    }
}

// ApplicationListener - 监听器
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E event);
}

// ApplicationEventPublisher - 事件发布器
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        publishEvent((Object) event);
    }
    
    void publishEvent(Object event);
}

// 使用示例
@Component
public class UserCreatedListener implements ApplicationListener<UserCreatedEvent> {
    @Override
    public void onApplicationEvent(UserCreatedEvent event) {
        System.out.println("用户创建事件：" + event.getUser().getName());
    }
}
```

---

## 9. 总结

Spring 框架的核心原理：

1. **IoC 容器**：通过反射和工厂模式管理 Bean 的生命周期
2. **依赖注入**：通过构造器、Setter、字段注入实现依赖关系
3. **AOP**：基于动态代理实现横切关注点的模块化
4. **事务管理**：通过 AOP 和 ThreadLocal 实现声明式事务
5. **MVC 框架**：通过 DispatcherServlet 统一处理请求

掌握这些核心原理，可以帮助我们：
- 更好地使用 Spring 框架
- 排查和解决复杂问题
- 进行性能优化和扩展开发

---

## 📚 参考资料

- [Spring Framework 官方文档](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring Framework GitHub](https://github.com/spring-projects/spring-framework)
- 《Spring 源码深度解析》
- 《Spring 技术内幕》
