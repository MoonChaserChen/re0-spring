# BeanFactory
1. Spring 容器的顶层规范，此容器中包含了其管理Bean的定义BeanDefinition，通过一个name对其进行唯一识别
2. 此容器管理的Bean既可以是单例的也可以是多例的，2.0开始增加了其它的Bean作用域，如：request，session
3. BeanFactory中既可以存放一此Bean组件，也可以集中存放相关配置信息
4. 通常应使用Push的配置策略，即对其依赖Bean进行注入，而不是手动去BeanFactory中去查找其依赖的Bean
5. BeanDefinition可以存放的位置比较灵活，LDAP, RDBMS, XML, properties file等
6. 当前工厂查找不到Bean时，会从其父工厂查找

此接口主要定义了一些getBean的方法，如果找不到相应的Bean，会抛出NoSuchBeanDefinitionException等BeansException异常，主要方法如下：

| 方法 | 说明 |
| ---- | ---- |
| Object getBean(String name) throws BeansException; | 1. 可以通过Bean别名获取 2.  |
| &lt;T> T getBean(String name, Class&lt;T> requiredType) throws BeansException; | 和上述方法类似，不过提供了指定类型，当类型不符合时会抛出BeanNotOfRequiredTypeException异常 |
| Object getBean(String name, Object... args) throws BeansException; | 通过Bean名称获取Bean时，传递构造器参数或工厂方法参数，同时会修改其BeanDefinition。只适用于多例模式，若不是多例，将会抛出BeanDefinitionStoreException异常 |
| &lt;T> T getBean(Class&lt;T> requiredType) throws BeansException; | 通过类型获取Bean，若能获取到多个，抛出NoUniqueBeanDefinitionException异常 |
| &lt;T> T getBean(Class&lt;T> requiredType, Object... args) throws BeansException; | |
| &lt;T> ObjectProvider&lt;T> getBeanProvider(Class&lt;T> requiredType); | 5.1增加的方法 |
| &lt;T> ObjectProvider&lt;T> getBeanProvider(ResolvableType requiredType); | 5.1增加的方法 |
| boolean containsBean(String name); | |
| boolean isSingleton(String name) throws NoSuchBeanDefinitionException; | |
| boolean isPrototype(String name) throws NoSuchBeanDefinitionException; | |
| boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException; | |
| boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException; | |
| Class<?> getType(String name) throws NoSuchBeanDefinitionException; | |
| Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException; | 5.2增加的方法 |
| String[] getAliases(String name); | |

## Bean的生命周期
同时BeanFactory也定义了Bean的生命周期规范，初始化生命周期如下：
1. BeanNameAware's setBeanName
2. BeanClassLoaderAware's setBeanClassLoader
3. BeanFactoryAware's setBeanFactory
4. EnvironmentAware's setEnvironment
5. EmbeddedValueResolverAware's setEmbeddedValueResolver
6. ResourceLoaderAware's setResourceLoader 只在application context中有效
7. ApplicationEventPublisherAware's setApplicationEventPublisher 只在application context中有效
8. MessageSourceAware's setMessageSource 只在application context中有效
9. ApplicationContextAware's setApplicationContext 只在application context中有效
10. ServletContextAware's setServletContext 只在application context中有效
11. postProcessBeforeInitialization methods of BeanPostProcessors
12. InitializingBean's afterPropertiesSet
13. a custom init-method definition
14. postProcessAfterInitialization methods of BeanPostProcessors

销毁时生命周期如下：
1. postProcessBeforeDestruction methods of DestructionAwareBeanPostProcessors
2. DisposableBean's destroy
3. a custom destroy-method definition

## ListableBeanFactory
BeanFactory通过name获取Bean只能有一个，但是通过Bean类型获取的时候可能用多个，ListableBeanFactory可以（根据类型）列出所有的Bean。
1. 不查找父窗口
2. 只管理通过BeanDefinition注入的Bean
3. 通过 `org.springframework.beans.factory.config.BeanDefinition.getResolvableType` 或者 `org.springframework.beans.factory.FactoryBean.getObjectType`进行类型判断

| 方法 | 说明 |
| ---- | ---- |
| boolean containsBeanDefinition(String beanName); | 通过name判断BeanDefinition是否存在 |
| int getBeanDefinitionCount(); | 获取BeanDefinition总数 |
| String[] getBeanDefinitionNames(); | 获取所有的Bean name |
| String[] getBeanNamesForType(ResolvableType type); | 通过类型获取所有的Bean name，包括子类。通常用getBeanNamesForType(type, true, true)实现 |
| String[] getBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit); | |
| String[] getBeanNamesForType(@Nullable Class&lt;?> type); | |
| String[] getBeanNamesForType(@Nullable Class&lt;?> type, boolean includeNonSingletons, boolean allowEagerInit); | |
| &lt;T> Map<String, T> getBeansOfType(@Nullable Class&lt;T> type) throws BeansException; | 通过类型获取所有的Bean，包括子类，通常用getBeansOfType(type, true, true)实现 |
| &lt;T> Map<String, T> getBeansOfType(@Nullable Class&lt;T> type, boolean includeNonSingletons, boolean allowEagerInit) throws BeansException; | |
| String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType); | 获取带指定注解的Bean name |
| Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException; | 获取带指定注解的Bean |
| &lt;A extends Annotation> A findAnnotationOnBean(String beanName, Class&lt;A> annotationType) throws NoSuchBeanDefinitionException; | 获取bean上的注解 |

## HierarchicalBeanFactory
BeanFactory规定了：当前工厂查找不到Bean时，会从其父工厂查找。HierarchicalBeanFactory定义了获取父BeanFactory方法，同时可从当前工厂判断是否包含Bean的方法：

| 方法 | 说明 |
| ---- | ---- |
| BeanFactory getParentBeanFactory(); | 获取父工厂 |
| boolean containsLocalBean(String name); | 当前工厂是否包含Bean |

这里有点需要理解的是，为什么这个HierarchicalBeanFactory规范只定义了获取父工厂方法，而设置父工厂方法却要放到ConfigurableBeanFactory中呢？
> 感觉上是一个接口规范的强化，因为ConfigurableBeanFactory是继承至HierarchicalBeanFactory的。HierarchicalBeanFactory只需要规定有一个父容器就行了，即get方法。

## ConfigurableBeanFactory
继承至HierarchicalBeanFactory，提供了一些配置BeanFactory的方法。

| 方法 | 说明 |
| void setParentBeanFactory(BeanFactory parentBeanFactory) throws IllegalStateException; | 设置父工厂，只能设置一次，不能改变 |
| void setBeanClassLoader(@Nullable ClassLoader beanClassLoader); | 设置Bean的类加载器 |
| ClassLoader getBeanClassLoader(); | |
| void setTempClassLoader(@Nullable ClassLoader tempClassLoader); | 临时Bean的类加载器，主要用于LTW延时加载 |
| ClassLoader getTempClassLoader(); | |
| void setCacheBeanMetadata(boolean cacheBeanMetadata); | 设置工厂里的BeanDefinition是否进行缓存，默认使用缓存，若不使用缓存则可以动态刷新Bean（比如动态修改xml中的Bean配置） |
| boolean isCacheBeanMetadata(); | |
| void setBeanExpressionResolver(@Nullable BeanExpressionResolver resolver); | 设置Bean配置中一些#{...}等表达式的处理器 |
| BeanExpressionResolver getBeanExpressionResolver(); | |
| void setConversionService(@Nullable ConversionService conversionService); | 设置ConversionService，ConversionService好像用于属性值转换 |
| ConversionService getConversionService(); | |
| void addPropertyEditorRegistrar(PropertyEditorRegistrar registrar); | 设置PropertyEditor相关 |
| void registerCustomEditor(Class<?> requiredType, Class<? extends PropertyEditor> propertyEditorClass); | |
| void copyRegisteredEditorsTo(PropertyEditorRegistry registry); | |
| void setTypeConverter(TypeConverter typeConverter); | 设置TypeConverter相关，TypeConverter优先于PropertyEditor |
| TypeConverter getTypeConverter(); | |
| void addEmbeddedValueResolver(StringValueResolver valueResolver); | 添加StringValueResolver，用于处理注解上的属性等 |
| boolean hasEmbeddedValueResolver(); | |
| String resolveEmbeddedValue(String value); | |
| void addBeanPostProcessor(BeanPostProcessor beanPostProcessor); | BeanPostProcessor相关 |
| int getBeanPostProcessorCount(); | |
| void registerScope(String scopeName, Scope scope); | 对ConfigurableBeanFactory的两个标准的Scope（单例、多例）进行扩展 |
| String[] getRegisteredScopeNames(); | 获取注册的Scope名称，不包括"singleton"、"prototype" |
| Scope getRegisteredScope(String scopeName); | 获取Scope |
| AccessControlContext getAccessControlContext(); | 工厂里的安全访问控制上下文 |
| void copyConfigurationFrom(ConfigurableBeanFactory otherFactory); | 从另一个ConfigurableBeanFactory复制，只复制工厂配置相关的，其它的比如BeanDefinition信息不会复制 |
| void registerAlias(String beanName, String alias) throws BeanDefinitionStoreException; | beanName的别名相关 |
| void resolveAliases(StringValueResolver valueResolver); | |
| BeanDefinition getMergedBeanDefinition(String beanName) throws NoSuchBeanDefinitionException; | 根据beanName获取BeanDefinition |
| boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException; | 根据beanName判断是否是FactoryBean |
| void setCurrentlyInCreation(String beanName, boolean inCreation); | 设置当前Bean正在创建 |
| boolean isCurrentlyInCreation(String beanName); | 判断Bean是否正在创建 |
| void registerDependentBean(String beanName, String dependentBeanName); | 设置bean的依赖关系 |
| String[] getDependentBeans(String beanName); | 获取依赖指定beanName的所有其它beanName |
| String[] getDependenciesForBean(String beanName); | 获取指定beanName依赖的其它beanName |
| void destroyBean(String beanName, Object beanInstance); | 销毁bean相关 |
| void destroyScopedBean(String beanName); | |
| void destroySingletons(); | |