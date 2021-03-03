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
| String[] getBeanNamesForType(ResolvableType type); | 通过类型获取所有的BeanDefinition，包括子类。通常用getBeanNamesForType(type, true, true)实现 |
| String[] getBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit); | |
| String[] getBeanNamesForType(@Nullable Class&lt;?> type); | |
| String[] getBeanNamesForType(@Nullable Class&lt;?> type, boolean includeNonSingletons, boolean allowEagerInit); | |
| &lt;T> Map<String, T> getBeansOfType(@Nullable Class&lt;T> type) throws BeansException; | 通过类型获取所有的Bean，包括子类，通常用getBeansOfType(type, true, true)实现 |
| &lt;T> Map<String, T> getBeansOfType(@Nullable Class&lt;T> type, boolean includeNonSingletons, boolean allowEagerInit) throws BeansException; | |
| String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType); | 获取带指定注解的Bean name |
| Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException; | 获取带指定注解的Bean |
| &lt;A extends Annotation> A findAnnotationOnBean(String beanName, Class&lt;A> annotationType) throws NoSuchBeanDefinitionException; | 获取bean上的注解 |

