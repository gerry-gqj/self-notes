# [Spring源码解读『占位符@Value(“${…}”)替换』](http://lidol.top/frame/2623/)

2020-04-29 分类：[Spring](http://lidol.top/category/frame/spring/) / [框架](http://lidol.top/category/frame/) 阅读(783) 评论(0)

上篇文章介绍了xml配置文件中占位符${…}的解析过程，本片文章我们来继续介绍Spring中另一种占位符@Value(“${…}”)，这种占位符一般出现在Java Config中，如下：

``` java
@Configuration
public class MyConfiguration {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String userName;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource1() {
        return new DriverManagerDataSource(url, userName, password);
    }
}
```

Spring对这种占位符的解析是通过Bean后置处理器完成的，这里用来解析的是AutowiredAnnotationBeanPostProcessor。

在理解AutowiredAnnotationBeanPostProcessor前，我们先来明确一个问题，就是上篇文章介绍的**PropertySourcesPlaceholderConfigurer Bean工厂后置处理器跟本篇文章介绍的Bean后置处理器不是孤立的**。通过Java Config配置方式，如果需要使用占位符，我们也需要告诉Spring容器.properties文件位置，所以可以通过上篇文章介绍的两种方式声明，那么相应的，也就引入了PropertySourcesPlaceholderConfigurer Bean工厂后置处理器。在上篇文章介绍PropertySourcesPlaceholderConfigurer的2.4节，doProcessProperties方法最后会讲valueResolver添加到BeanFactory中：

``` java
// 添加valueResolver，可用于@Value("${}")解析
// New in Spring 2.5: resolve placeholders in alias target names and aliases as well.
beanFactoryToProcess.resolveAliases(valueResolver);

// New in Spring 3.0: resolve placeholders in embedded values such as annotation attributes.
beanFactoryToProcess.addEmbeddedValueResolver(valueResolver);
```

这里是AutowiredAnnotationBeanPostProcessor解析@Value(“${…}”)占位符的重要逻辑，后面详细介绍。

## 1. AutowiredAnnotationBeanPostProcessor原理

[![img](http://cdn.lidol.top/lidol_blog/AutowiredAnnotationBeanPostProcessor.png)](http://cdn.lidol.top/lidol_blog/AutowiredAnnotationBeanPostProcessor.png)

AutowiredAnnotationBeanPostProcessor是一个Bean后置处理器，按照我们文章的介绍，后置处理器会作用于bean实例化和初始化过程。AutowiredAnnotationBeanPostProcessor同时也是一个InstantiationAwareBeanPostProcessor，该类型的Bean后置处理器会在org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean方法中的调用，如下：

[![img](http://cdn.lidol.top/lidol_blog/20200617082231.png)](http://cdn.lidol.top/lidol_blog/20200617082231.png)

就是在AutowiredAnnotationBeanPostProcessor的postProcessPropertyValues方法中完成了@Value(“${…}”)占位符的替换。

还有一个问题需要关注一下，AutowiredAnnotationBeanPostProcessor是何时初始化的，因为我们并没有在xml配置文件中显式声明这样一个bean。我们再使用Java Config配置Bean时，一般会在spring启动配置xml文件中声明如下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.1.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:my.properties"/>
    
    <context:annotation-config/>

    <context:component-scan base-package="com.zhuoli.service.spring.explore.property.config"/>

</beans>
```

这里可以看到我们声明”annotation-config”和”component-scan”，就是通过这两个注解的声明，完成了AutowiredAnnotationBeanPostProcessor的初始化（annotation-config可以省略，只声明component-scan也可以）。

[![img](http://cdn.lidol.top/lidol_blog/20200628080749.png)](http://cdn.lidol.top/lidol_blog/20200628080749.png)

## 2. AutowiredAnnotationBeanPostProcessor替换占位符

下面我们来看Java Config中，@Value(“${…}”)占位符如何替换的。上面我们讲到，在org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean方法中会调用AutowiredAnnotationBeanPostProcessor的postProcessPropertyValues方法，该方法就是用来替换占位符的。

### 2.1 postProcessPropertyValues

``` java
public PropertyValues postProcessPropertyValues(
        PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeanCreationException {

    // 1. 从Bean的class对象中获取所有需要注入的（Autowired、Value、Inject注解）元数据
    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
    try {
        // 2. 数据注入
        metadata.inject(bean, beanName, pvs);
    }
    catch (BeanCreationException ex) {
        throw ex;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
    }
    return pvs;
}
```

首先从Bean Class中获取所有需要注入的元数据（包括Field和Method），然后进行数据注入。

### 2.2 findAutowiringMetadata

``` java
private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, PropertyValues pvs) {
	// 获取bean的beanName作为缓存的key如果不存在beanName就用类名
	String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());
	// 从需要注入的缓存中查找是否存在已经解析过的需要注入的元数据
	InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
	//如果缓存中元数据不存在或者元数据对应的目标class不是当前bean的beanclass则需要刷新
	if (InjectionMetadata.needsRefresh(metadata, clazz)) {
		synchronized (this.injectionMetadataCache) {
			metadata = this.injectionMetadataCache.get(cacheKey);
			if (InjectionMetadata.needsRefresh(metadata, clazz)) {
				if (metadata != null) {
					metadata.clear(pvs);
				}
				try {
					//查找并创建需要注入的元数据，然后保存到缓存中取
					metadata = buildAutowiringMetadata(clazz);
					this.injectionMetadataCache.put(cacheKey, metadata);
				}
				catch (NoClassDefFoundError err) {
					throw new IllegalStateException("Failed to introspect bean class [" + clazz.getName() +
							"] for autowiring metadata: could not find class that it depends on", err);
				}
			}
		}
	}
	return metadata;
}
private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {
	LinkedList<InjectionMetadata.InjectedElement> elements = new LinkedList<InjectionMetadata.InjectedElement>();
	Class<?> targetClass = clazz;

	do {
		final LinkedList<InjectionMetadata.InjectedElement> currElements =
				new LinkedList<InjectionMetadata.InjectedElement>();

		// 1. 查找Autowired，Value或者Inject注解的字段，并保存到currElements中
		ReflectionUtils.doWithLocalFields(targetClass, new ReflectionUtils.FieldCallback() {
			@Override
			public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
				AnnotationAttributes ann = findAutowiredAnnotation(field);
				if (ann != null) {
					if (Modifier.isStatic(field.getModifiers())) {
						if (logger.isWarnEnabled()) {
							logger.warn("Autowired annotation is not supported on static fields: " + field);
						}
						return;
					}
					boolean required = determineRequiredStatus(ann);
					currElements.add(new AutowiredFieldElement(field, required));
				}
			}
		});

		// 2.查找Autowired，Value或者Inject注解的方法，并保存到currElements中
		ReflectionUtils.doWithLocalMethods(targetClass, new ReflectionUtils.MethodCallback() {
			@Override
			public void doWith(Method method) throws IllegalArgumentException, IllegalAccessException {
				Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
				if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
					return;
				}
				AnnotationAttributes ann = findAutowiredAnnotation(bridgedMethod);
				if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
					if (Modifier.isStatic(method.getModifiers())) {
						if (logger.isWarnEnabled()) {
							logger.warn("Autowired annotation is not supported on static methods: " + method);
						}
						return;
					}
					if (method.getParameterTypes().length == 0) {
						if (logger.isWarnEnabled()) {
							logger.warn("Autowired annotation should only be used on methods with parameters: " +
									method);
						}
					}
					boolean required = determineRequiredStatus(ann);
					PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
					currElements.add(new AutowiredMethodElement(method, required, pd));
				}
			}
		});

		elements.addAll(0, currElements);
		targetClass = targetClass.getSuperclass();
	}
	while (targetClass != null && targetClass != Object.class);

	// 将Bean Class中需要注入的Field和Method包装为InjectionMetadata返回
	return new InjectionMetadata(clazz, elements);
}
```

### 2.3 org.springframework.beans.factory.annotation.InjectionMetadata#inject

``` java
public void inject(Object target, String beanName, PropertyValues pvs) throws Throwable {
	Collection<InjectedElement> elementsToIterate =
			(this.checkedElements != null ? this.checkedElements : this.injectedElements);
	if (!elementsToIterate.isEmpty()) {
		for (InjectedElement element : elementsToIterate) {
			if (logger.isDebugEnabled()) {
				logger.debug("Processing injected element of bean '" + beanName + "': " + element);
			}
			element.inject(target, beanName, pvs);
		}
	}
}
```

遍历通过上面方法获取的需要注入的Field和Method元数据，进行数据注入。element.inject方法会对于Field和Method元数据，会调用到不同的方法。这里我们关注占位符的替换，最终会调用到org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement#inject方法。

``` java
protected void inject(Object bean, String beanName, PropertyValues pvs) throws Throwable {
		Field field = (Field) this.member;
		Object value;
		if (this.cached) {
			value = resolvedCachedArgument(beanName, this.cachedFieldValue);
		}
		else {
			DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
			desc.setContainingClass(bean.getClass());
			Set<String> autowiredBeanNames = new LinkedHashSet<String>(1);
			TypeConverter typeConverter = beanFactory.getTypeConverter();
			try {
				// 1. 从beanFactory解析需要注入的Field的value值
				value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
			}
			catch (BeansException ex) {
				throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);
			}
			synchronized (this) {
				if (!this.cached) {
					if (value != null || this.required) {
						this.cachedFieldValue = desc;
						registerDependentBeans(beanName, autowiredBeanNames);
						if (autowiredBeanNames.size() == 1) {
							String autowiredBeanName = autowiredBeanNames.iterator().next();
							if (beanFactory.containsBean(autowiredBeanName) &&
									beanFactory.isTypeMatch(autowiredBeanName, field.getType())) {
								this.cachedFieldValue = new ShortcutDependencyDescriptor(
										desc, autowiredBeanName, field.getType());
							}
						}
					}
					else {
						this.cachedFieldValue = null;
					}
					this.cached = true;
				}
			}
		}
		if (value != null) {
			// 2. 反射设置Field value
			ReflectionUtils.makeAccessible(field);
			field.set(bean, value);
		}
	}
}
```

### 2.4 AutowireCapableBeanFactory#resolveDependency

调用AutowireCapableBeanFactory的resolveDependency方法，最终会调用到org.springframework.beans.factory.support.DefaultListableBeanFactory#resolveDependency方法。

``` java
public Object resolveDependency(DependencyDescriptor descriptor, String requestingBeanName,
		Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException {

	descriptor.initParameterNameDiscovery(getParameterNameDiscoverer());
	if (javaUtilOptionalClass == descriptor.getDependencyType()) {
		return new OptionalDependencyFactory().createOptionalDependency(descriptor, requestingBeanName);
	}
	else if (ObjectFactory.class == descriptor.getDependencyType() ||
			ObjectProvider.class == descriptor.getDependencyType()) {
		return new DependencyObjectProvider(descriptor, requestingBeanName);
	}
	else if (javaxInjectProviderClass == descriptor.getDependencyType()) {
		return new Jsr330ProviderFactory().createDependencyProvider(descriptor, requestingBeanName);
	}
	else {
		Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(
				descriptor, requestingBeanName);
		if (result == null) {
			// 解析Field的value值
			result = doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
		}
		return result;
	}
}
public Object doResolveDependency(DependencyDescriptor descriptor, String beanName,
		Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException {

	InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
	try {
		Object shortcut = descriptor.resolveShortcut(this);
		if (shortcut != null) {
			return shortcut;
		}

		Class<?> type = descriptor.getDependencyType();
		// 1. 获取Field对应的value值
		Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
		if (value != null) {
			// 2. 如果value类型为String，可能存在占位符，解析占位符
			if (value instanceof String) {
				String strVal = resolveEmbeddedValue((String) value);
				BeanDefinition bd = (beanName != null && containsBean(beanName) ? getMergedBeanDefinition(beanName) : null);
				value = evaluateBeanDefinitionString(strVal, bd);
			}
			TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
			return (descriptor.getField() != null ?
					converter.convertIfNecessary(value, type, descriptor.getField()) :
					converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
		}

		// ……
	}
	finally {
		ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
	}
}
```

对于value是String类型的Field，获取到value之后可能会是占位符，所以会通过resolveEmbeddedValue方法做进一步解析，占位符的替换就是在该方法中完成的。

``` java
public String resolveEmbeddedValue(@Nullable String value) {
    if (value == null) {
        return null;
    }
    String result = value;
    // 遍历容器中所有的StringValueResolver，解析占位符
    for (StringValueResolver resolver : this.embeddedValueResolvers) {
        result = resolver.resolveStringValue(result);
        if (result == null) {
            return null;
        }
    }
    return result;
}
```

关于这个resolveEmbeddedValue这个方法，我在看源码的时候一度很疑惑。因为容器中很有可能会存在多个StringValueResolver的，如果第一个StringValueResolver占位符没有解析成功，那么result就为null，直接返回null了。虽然存在多个StringValueResolver，相当于后面的也不生效了吗？其实这么理解错误的地方在于，**StringValueResolver在做占位符解析时，如果待解析的值中不存在占位符（不存在占位符前缀）会直接原样返回，如果存在占位符，但是没有解析到结果是会直接抛异常或者原样返回，并不会返回null**。

先**不考虑抛异常的情况，如果前面的StringValueResolver没有解析到占位符，会将占位符原样返回，通过上述遍历StringValueResolver，交由后面的StringValueResolver解析，直到解析到结果；如果该过程中，某个StringValueResolver将占位符解析为真正的值了，那么之后的StringValueResolver都是将解析成功的value直接返回**。

那么什么情况下会抛出异常呢？回顾一下Spring容器的embeddedValueResolvers中的元素什么时候添加进去的。通过上篇文章[Spring源码解读『占位符${…}替换』](http://lidol.top/frame/2616/)我们知道，**在解析“context:property-placeholder”标签时初始化了PropertySourcesPlaceholderConfigurer的BeanDefinition，在调用PropertySourcesPlaceholderConfigurer的postProcessBeanFactory方法中，将StringValueResolver添加到embeddedValueResolvers中，一个“context:property-placeholder”标签，最终会生成一个StringValueResolver添加到embeddedValueResolvers中**。同时我们在使用“context:property-placeholder”标签时，可以设置一个属性“ignore-unresolvable”，如下：

``` xml
<context:property-placeholder location="classpath:my.properties" ignore-unresolvable="true"/>
```

该属性默认情况下为false，表示不允许解析器忽略未解析的占位符。那么如果StringValueResolver没有解析到占位符，就会抛异常：

``` java
throw new IllegalArgumentException("Could not resolve placeholder '" + placeholder + "'" + " in value \"" + value + "\"");
```

这里我们延伸一个问题，spring启动配置文件中设置了多个“context:property-placeholder”，并且都没有设置“ignore-unresolvable”，容器启动时报错的问题。关于这个问题的解释，搜了很多答案都是这样的：

Spring容器采用反射扫描的发现机制，在探测到Spring容器中有一个org.springframework.beans.factory.config.PropertyPlaceholderConfigurer的Bean就会停止对剩余PropertyPlaceholderConfigurer的扫描。Spring容器仅允许最多定义一个PropertyPlaceholderConfigurer(或<context:property-placeholder/>)，其余的会被Spring忽略掉。（[spring多个context:property-placeholder不生效问题](https://blog.csdn.net/liuxiao723846/article/details/82466992)）

其实这样讲是不对的（PropertyPlaceholderConfigurer作用跟PropertySourcesPlaceholderConfigurer一样，在Spring 3.1及之后的版本，使用PropertySourcesPlaceholderConfigurer替换了PropertyPlaceholderConfigurer），**Spring允许定义多个“context:property-placeholder”，可以存在多个PropertySourcesPlaceholderConfigurer，报错的原因是因为PropertySourcesPlaceholderConfigurer不允许忽略未解析的占位符，导致后面的StringValueResolver没机会去解析占位符**。

最后关于@Value(“${…}”)占位符替换我们做个总结：

- 每个**context:property-placeholder**标签会生成一个PropertySourcesPlaceholderConfigurer，对应一个StringValueResolver
- PropertySourcesPlaceholderConfigurer是一种BeanFactoryPostProcessor，在Spring容器启动过程中会调用该类的postProcessBeanFactory，并在该方法中替换“${…}”占位符，并将StringValueResolver添加到BeanFactory，用于@Value(“${…}”)占位符解析
- @Value(“${…}”)占位符的解析是通过AutowiredAnnotationBeanPostProcessor完成的，该类是一种BeanPostProcessor，可以通过“annotation-config”标签或者“component-scan”标签解析的时候初始化BeanDefinition，并在bean初始化过程中生效
- 当存在多个StringValueResolver时，在替换占位符时，排在前面的StringValueResolver优先级更高

## 3. 分布式配置中心

在实际开发中，有很多成熟的分布式配置中心供我们使用，比如美团点评的Lion、携程的Apollo等。在使用上比如美团的Lion，我们需要在应用中声明一个Bean，该Bean是一个BeanFactoryPostProcessor，在实现的postProcessBeanFactory的方法中，实现${…}占位符的解析（核心是实现了一个StringValueResolver，该StringValueResolver会向配置中心服务端获取值），并将StringValueResolver添加到embeddedValueResolvers，这就是分布式配置中心生效的原理。

这里还有另外一个问题需要关注，如果同时使用分布式配置中心和本地的properties配置文件，会存在多个跟占位符解析相关的BeanFactoryPostProcessor，也会存在多个StringValueResolver，上面我们讲到，如果前面的StringValueResolver没有解析到占位符，可能会抛异常的，后面的StringValueResolver即使可以解析占位符，也没有机会解析了。那么分布式配置中心是如何解决这个问题的？

携程的Apollo我没仔细研究过，点评的Lion是在自定义的BeanFactoryPostProcessor类实现PriorityOrdered，并给了一个较小的order值（PropertySourcesPlaceholderConfigurer也实现了PriorityOrdered接口，默认order值是2147483647），这样自定义实现的BeanFactoryPostProcessor就会有较高的优先级，进而自定义实现的StringValueResolver也会先进行解析。如果解析不到，不会抛异常，而是将占位符原样返回，交由其他的StringValueResolver解析，这样就完成了分布式配置中心和本地配置中心的配合工作。

> 参考链接：
>
> 1. Spring 源码
>
> 2. [Spring源码—–AutowiredAnnotationBeanPostProcessor解析Autowired](https://www.jianshu.com/p/fae3953edea0)
>
> 3. [spring的启动过程04.1-value注解替换过程](https://blog.csdn.net/qq_28580959/article/details/60129329)