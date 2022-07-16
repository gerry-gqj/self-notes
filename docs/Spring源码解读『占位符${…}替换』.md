# [Spring源码解读『占位符${…}替换』](http://lidol.top/frame/2616/)

2020-04-28 分类：[Spring](http://lidol.top/category/frame/spring/) / [框架](http://lidol.top/category/frame/) 阅读(935) 评论(0)

在使用Spring时，对于一些比较固定的参数，我们一般会采用配置的方式，将这些参数配置在.properties配置文件中，然后在Bean初始化过程中替换为配置文件中配置的真实值。在Spring中，这种典型的的使用会存在以下两种方式：

在xml配置中，通过${…}：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.1.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:my.properties"/>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

</beans>
```

在java config中，通过@Value(“${…}”)：

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

Spring处理以上两种占位符的替换采用不同的方式，**xml注入的占位符Spring采用bean工厂后置处理器处理，注解方式的占位符Spring采用bean后置处理器处理**，本篇文章我们先来看一下xml注入的占位符的替换过程。

## 1. xml占位符注入示例

一般我们使用xml注入会在spring配置文件中注明.properties文件的位置，一般我们会添加如下配置：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.1.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:my.properties"/>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

</beans>
```

引入该配置后，**Spring容器会自动创建PropertySourcesPlaceholderConfigurer实体bean，该bean为工厂后置处理器，会加载指定的文件并替换beanDefinition对象里面的占位符**

或者添加如下配置：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.1.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="propertySourcesPlaceholderConfigurer" class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:my.properties</value>
            </list>
        </property>
    </bean>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

</beans>
```

引入该配置，其实就是显式的声明一个PropertySourcesPlaceholderConfigurer实体bean，跟上述方式一致。

``` java
public class SpringPropertyTest {
    public static void main(String[] args) {
        AbstractApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring.xml");
        DataSource dataSource = (DriverManagerDataSource) applicationContext.getBean("dataSource");
        System.out.println(dataSource);
    }
}
```

通过如上配置，DataSource bean是可以成功获取.properties文件中的占位的内容并成功替换的。核心就在上述两种xml的配置中，通过上述两种xml的配置，最终都会生成PropertySourcesPlaceholderConfigurer实体bean，而该bean工厂后置处理器会在Spring容器加载BeanDefinition后，Bean实例化前，bean工厂后置处理器调用中起作用，将指定BeanDefinition中的占位符替换成真正的值。

这里再看一下上述两种xml配置，第二种配置很好理解，我们显式声明了PropertySourcesPlaceholderConfigurer，但是第一种方式我们并没有显式声明该bean，那么PropertySourcesPlaceholderConfigurer实体bean如何生成的？

[![img](Spring源码解读『占位符${…}替换』.assets/20200628075834.png)](http://cdn.lidol.top/lidol_blog/20200628075834.png)

## 2. PropertySourcesPlaceholderConfigurer

上面讲到PropertySourcesPlaceholderConfigurer是一个bean工厂后置处理器，并在读取xml配置文件BeanDefinition的过程中已经将PropertySourcesPlaceholderConfigurer对应的BeanDefinition注册到Spring容器。结合之前的文章[Spring源码解读『IOC容器2-Bean加载过程』](http://lidol.top/frame/2524/)，可以知道在invokeBeanFactoryPostProcessors方法中，会调用PropertySourcesPlaceholderConfigurer的postProcessBeanFactory方法。就是在这个方法中，完成了BeanDefinition中${…}占位符的替换。

### 2.1 postProcessBeanFactory

``` java
@Override
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
	if (this.propertySources == null) {
		this.propertySources = new MutablePropertySources();
		if (this.environment != null) {
			this.propertySources.addLast(
				new PropertySource<Environment>(ENVIRONMENT_PROPERTIES_PROPERTY_SOURCE_NAME, this.environment) {
					@Override
					public String getProperty(String key) {
						return this.source.getProperty(key);
					}
				}
			);
		}
		try {
			// 1. 加载PropertySourcesPlaceholderConfigurer成员变量locations位置的properties文件（spring xml配置文件中配置的properties文件位置）
			PropertySource<?> localPropertySource =
				new PropertiesPropertySource(LOCAL_PROPERTIES_PROPERTY_SOURCE_NAME, mergeProperties());
			if (this.localOverride) {
				this.propertySources.addFirst(localPropertySource);
			}
			else {
				this.propertySources.addLast(localPropertySource);
			}
		}
		catch (IOException ex) {
			throw new BeanInitializationException("Could not load properties", ex);
		}
	}

	// 2. 替换BeanDefinition中占位符
	processProperties(beanFactory, new PropertySourcesPropertyResolver(this.propertySources));
}
```

### 2.2 org.springframework.core.io.support.PropertiesLoaderSupport#mergeProperties

``` java
protected Properties mergeProperties() throws IOException {
	Properties result = new Properties();

	if (this.localOverride) {
		// Load properties from file upfront, to let local properties override.
		loadProperties(result);
	}

	if (this.localProperties != null) {
		for (Properties localProp : this.localProperties) {
			CollectionUtils.mergePropertiesIntoMap(localProp, result);
		}
	}

	if (!this.localOverride) {
		// Load properties from file afterwards, to let those properties override.
		loadProperties(result);
	}

	return result;
}

protected void loadProperties(Properties props) throws IOException {
	if (this.locations != null) {
		for (Resource location : this.locations) {
			if (logger.isInfoEnabled()) {
				logger.info("Loading properties file from " + location);
			}
			try {
				PropertiesLoaderUtils.fillProperties(
						props, new EncodedResource(location, this.fileEncoding), this.propertiesPersister);
			}
			catch (IOException ex) {
				if (this.ignoreResourceNotFound) {
					if (logger.isWarnEnabled()) {
						logger.warn("Could not load properties from " + location + ": " + ex.getMessage());
					}
				}
				else {
					throw ex;
				}
			}
		}
	}
}
```

通过上述方法，完成将本地properties文件中的配置转化为MutablePropertySources，并在随后替换BeanDefinition中的占位符的过程中使用。

### 2.3 processProperties

``` java
protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
		final ConfigurablePropertyResolver propertyResolver) throws BeansException {

	// 1. 设置ConfigurablePropertyResolver
	// 1.1 设置占位符前缀，默认占位符前缀为"${"
	propertyResolver.setPlaceholderPrefix(this.placeholderPrefix);
	// 1.2 设置占位符后缀，默认占位符后缀为"}"
	propertyResolver.setPlaceholderSuffix(this.placeholderSuffix);
	// 1.3 默认占位符默认值分隔符，默认分隔符为":"
	propertyResolver.setValueSeparator(this.valueSeparator);

	// 2. 初始化StringValueResolver，用于将占位符解析为真实value
	StringValueResolver valueResolver = new StringValueResolver() {
		public String resolveStringValue(String strVal) {
			String resolved = ignoreUnresolvablePlaceholders ?
					propertyResolver.resolvePlaceholders(strVal) :
					propertyResolver.resolveRequiredPlaceholders(strVal);
			return (resolved.equals(nullValue) ? null : resolved);
		}
	};

	// 3. 替换容器中所有使用了占位符的BeanDefinition
	doProcessProperties(beanFactoryToProcess, valueResolver);
}
```

### 2.4 doProcessProperties

``` java
protected void doProcessProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
		StringValueResolver valueResolver) {

	BeanDefinitionVisitor visitor = new BeanDefinitionVisitor(valueResolver);

	String[] beanNames = beanFactoryToProcess.getBeanDefinitionNames();
	// 遍历Spring容器中所有的BeanDefinition定义，替换占位符
	for (String curName : beanNames) {
		// Check that we're not parsing our own bean definition,
		// to avoid failing on unresolvable placeholders in properties file locations.
		if (!(curName.equals(this.beanName) && beanFactoryToProcess.equals(this.beanFactory))) {
			BeanDefinition bd = beanFactoryToProcess.getBeanDefinition(curName);
			try {
				visitor.visitBeanDefinition(bd);
			}
			catch (Exception ex) {
				throw new BeanDefinitionStoreException(bd.getResourceDescription(), curName, ex.getMessage(), ex);
			}
		}
	}

	// 添加valueResolver，可用于@Value("${}")解析
	// New in Spring 2.5: resolve placeholders in alias target names and aliases as well.
	beanFactoryToProcess.resolveAliases(valueResolver);

	// New in Spring 3.0: resolve placeholders in embedded values such as annotation attributes.
	beanFactoryToProcess.addEmbeddedValueResolver(valueResolver);
}
```

### 2.5 visitBeanDefinition

``` java
public void visitBeanDefinition(BeanDefinition beanDefinition) {
	// 替换BeanDefinition parentName属性
	visitParentName(beanDefinition);
	// 替换BeanDefinition beanClassName属性
	visitBeanClassName(beanDefinition);
	// 替换BeanDefinition factoryBeanName属性
	visitFactoryBeanName(beanDefinition);
	// 替换BeanDefinition factoryMethodName属性
	visitFactoryMethodName(beanDefinition);
	// 替换BeanDefinition scope属性
	visitScope(beanDefinition);
	// 替换BeanDefinition property属性
	visitPropertyValues(beanDefinition.getPropertyValues());
	// 替换BeanDefinition 
	ConstructorArgumentValues cas = beanDefinition.getConstructorArgumentValues();
	visitIndexedArgumentValues(cas.getIndexedArgumentValues());
	visitGenericArgumentValues(cas.getGenericArgumentValues());
}
```

替换xml bean定义中的property占位符属性，会调用visitPropertyValues方法。

### 2.6 visitPropertyValues

``` java
// 参数pvs为BeanDefinition所有的property，有可能存在占位符，也有可能存在真正的值
protected void visitPropertyValues(MutablePropertyValues pvs) {
	PropertyValue[] pvArray = pvs.getPropertyValues();
	for (PropertyValue pv : pvArray) {
		// 遍历解析，如果成功解析，会进入下面的if判断
		Object newVal = resolveValue(pv.getValue());
		if (!ObjectUtils.nullSafeEquals(newVal, pv.getValue())) {
			pvs.add(pv.getName(), newVal);
		}
	}
}
```

### 2.7 resolveValue

``` java
protected Object resolveValue(Object value) {
	if (value instanceof BeanDefinition) {
		visitBeanDefinition((BeanDefinition) value);
	}
	else if (value instanceof BeanDefinitionHolder) {
		visitBeanDefinition(((BeanDefinitionHolder) value).getBeanDefinition());
	}
	else if (value instanceof RuntimeBeanReference) {
		RuntimeBeanReference ref = (RuntimeBeanReference) value;
		String newBeanName = resolveStringValue(ref.getBeanName());
		if (!newBeanName.equals(ref.getBeanName())) {
			return new RuntimeBeanReference(newBeanName);
		}
	}
	else if (value instanceof RuntimeBeanNameReference) {
		RuntimeBeanNameReference ref = (RuntimeBeanNameReference) value;
		String newBeanName = resolveStringValue(ref.getBeanName());
		if (!newBeanName.equals(ref.getBeanName())) {
			return new RuntimeBeanNameReference(newBeanName);
		}
	}
	else if (value instanceof Object[]) {
		visitArray((Object[]) value);
	}
	else if (value instanceof List) {
		visitList((List) value);
	}
	else if (value instanceof Set) {
		visitSet((Set) value);
	}
	else if (value instanceof Map) {
		visitMap((Map) value);
	}
	else if (value instanceof TypedStringValue) {
		TypedStringValue typedStringValue = (TypedStringValue) value;
		String stringValue = typedStringValue.getValue();
		if (stringValue != null) {
			String visitedString = resolveStringValue(stringValue);
			typedStringValue.setValue(visitedString);
		}
	}
	else if (value instanceof String) {
		return resolveStringValue((String) value);
	}
	return value;
}
```

这里我们来看String类型占位符的解析逻辑，会调用resolveStringValue。

``` java
protected String resolveStringValue(String strVal) {
	if (this.valueResolver == null) {
		throw new IllegalStateException("No StringValueResolver specified - pass a resolver " +
				"object into the constructor or override the 'resolveStringValue' method");
	}
	String resolvedValue = this.valueResolver.resolveStringValue(strVal);
	// Return original String if not modified.
	return (strVal.equals(resolvedValue) ? strVal : resolvedValue);
}
```

这里我们重点看一下这里的valueResolver是什么时候初始化的，其实就是上面的processProperties方法中：

``` java
StringValueResolver valueResolver = new StringValueResolver() {
		@Override
		public String resolveStringValue(String strVal) {
			String resolved = (ignoreUnresolvablePlaceholders ?
					propertyResolver.resolvePlaceholders(strVal) :
					propertyResolver.resolveRequiredPlaceholders(strVal));
			if (trimValues) {
				resolved = resolved.trim();
			}
			return (resolved.equals(nullValue) ? null : resolved);
		}
	};
```

所以这里调用resolveStringValue实际会调用PropertyResolver的resolvePlaceholders方法或resolveRequiredPlaceholders方法，实际会调用哪个方法会根据PropertySourcesPlaceholderConfigurer的ignoreUnresolvablePlaceholders属性，该属性是可以在spring的启动配置xml文件中设置的，如果ignoreUnresolvablePlaceholders设置为true，则表示会忽略不能解析的占位符，调用resolvePlaceholders方法解析。如果ignoreUnresolvablePlaceholders设置为false，则表示不能忽略不能解析的占位符，调用resolveRequiredPlaceholders方法解析。ignoreUnresolvablePlaceholders默认为false。

### 2.8 resolveRequiredPlaceholders

resolveRequiredPlaceholders方法最终会调用到AbstractPropertyResolver类的resolveRequiredPlaceholders方法。

``` java
public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
    if (this.strictHelper == null) {
        this.strictHelper = createPlaceholderHelper(false);
    }
    return doResolvePlaceholders(text, this.strictHelper);
}
private PropertyPlaceholderHelper createPlaceholderHelper(boolean ignoreUnresolvablePlaceholders) {
    return new PropertyPlaceholderHelper(this.placeholderPrefix, this.placeholderSuffix,
            this.valueSeparator, ignoreUnresolvablePlaceholders);
}
private String doResolvePlaceholders(String text, PropertyPlaceholderHelper helper) {
    return helper.replacePlaceholders(text, this::getPropertyAsRawString);
}

/**
 * Retrieve the specified property as a raw String,
 * i.e. without resolution of nested placeholders.
 * @param key the property name to resolve
 * @return the property value or {@code null} if none found
 */
@Nullable
protected abstract String getPropertyAsRawString(String key);
```

所以resolveRequiredPlaceholders方法最终会调用到PropertyPlaceholderHelper的replacePlaceholders方法。

### 2.9 replacePlaceholders

``` java
public String replacePlaceholders(String value, PlaceholderResolver placeholderResolver) {
    Assert.notNull(value, "'value' must not be null");
    return parseStringValue(value, placeholderResolver, new HashSet<>());
}
```

这里我们重点关注一下参数placeholderResolver，该参数在doResolvePlaceholders方法中引入函数表达式作为参数（this::getPropertyAsRawString）。

``` java
protected String parseStringValue(
        String value, PlaceholderResolver placeholderResolver, Set<String> visitedPlaceholders) {

    StringBuilder result = new StringBuilder(value);

    int startIndex = value.indexOf(this.placeholderPrefix);
    while (startIndex != -1) {
        int endIndex = findPlaceholderEndIndex(result, startIndex);
        if (endIndex != -1) {
            String placeholder = result.substring(startIndex + this.placeholderPrefix.length(), endIndex);
            String originalPlaceholder = placeholder;
            if (!visitedPlaceholders.add(originalPlaceholder)) {
                throw new IllegalArgumentException(
                        "Circular placeholder reference '" + originalPlaceholder + "' in property definitions");
            }
            // Recursive invocation, parsing placeholders contained in the placeholder key.
            placeholder = parseStringValue(placeholder, placeholderResolver, visitedPlaceholders);
            // Now obtain the value for the fully resolved key...
            // 使用placeholderResolver解析占位符
            String propVal = placeholderResolver.resolvePlaceholder(placeholder);
            if (propVal == null && this.valueSeparator != null) {
                int separatorIndex = placeholder.indexOf(this.valueSeparator);
                if (separatorIndex != -1) {
                    String actualPlaceholder = placeholder.substring(0, separatorIndex);
                    String defaultValue = placeholder.substring(separatorIndex + this.valueSeparator.length());
                    propVal = placeholderResolver.resolvePlaceholder(actualPlaceholder);
                    if (propVal == null) {
                        propVal = defaultValue;
                    }
                }
            }
            if (propVal != null) {
                // Recursive invocation, parsing placeholders contained in the
                // previously resolved placeholder value.
                propVal = parseStringValue(propVal, placeholderResolver, visitedPlaceholders);
                result.replace(startIndex, endIndex + this.placeholderSuffix.length(), propVal);
                if (logger.isTraceEnabled()) {
                    logger.trace("Resolved placeholder '" + placeholder + "'");
                }
                startIndex = result.indexOf(this.placeholderPrefix, startIndex + propVal.length());
            }
            else if (this.ignoreUnresolvablePlaceholders) {
                // Proceed with unprocessed value.
                startIndex = result.indexOf(this.placeholderPrefix, endIndex + this.placeholderSuffix.length());
            }
            else {
                throw new IllegalArgumentException("Could not resolve placeholder '" +
                        placeholder + "'" + " in value \"" + value + "\"");
            }
            visitedPlaceholders.remove(originalPlaceholder);
        }
        else {
            startIndex = -1;
        }
    }

    return result.toString();
}
```

占位符解析的过程，核心就是通过placeholderResolver的resolvePlaceholder方法解析。上面我们讲到，该参数是一个函数表达式this::getPropertyAsRawString，所以解析的过程实际调用了org.springframework.core.env.PropertySourcesPropertyResolver#getPropertyAsRawString方法。

### 2.10 getPropertyAsRawString

``` java
protected String getPropertyAsRawString(String key) {
    return getProperty(key, String.class, false);
}

@Nullable
protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
    if (this.propertySources != null) {
        // 遍历propertySources，解析占位符key，propertySources中每个元素为一个.properties文件的解析结果
        for (PropertySource<?> propertySource : this.propertySources) {
            if (logger.isTraceEnabled()) {
                logger.trace("Searching for key '" + key + "' in PropertySource '" +
                        propertySource.getName() + "'");
            }
            // 从properties文件中获取key对应的value
            Object value = propertySource.getProperty(key);
            if (value != null) {
                if (resolveNestedPlaceholders && value instanceof String) {
                    value = resolveNestedPlaceholders((String) value);
                }
                logKeyFound(key, propertySource, value);
                return convertValueIfNecessary(value, targetValueType);
            }
        }
    }
    if (logger.isDebugEnabled()) {
        logger.debug("Could not find key '" + key + "' in any property source");
    }
    return null;
}
```

以上就是占位符${…}的解析过程，总的来说就是这种占位符在加载BeanDefinition后，bean实例化之前的一个环节——执行Bean工厂后置处理器的过程中，通过PropertySourcesPlaceholderConfigurer Bean工厂后置处理器，将容器中BeanDefinition中的占位符替换为真正的值，后面Bean实例化过程就可以使用替换后的值了，对${…}占位符无感知。

除了${…}占位符，Spring中通过Java Config配置，还允许使用另一种@Value(“${…}”)占位符，这种占位符的替换是通过Bean后置处理器处理的，我们在下篇文章中介绍。

> 参考链接：
>
> 1. Spring源码
>
> 2. [占位符替换过程-xml配置的参数](https://blog.csdn.net/qq_28580959/article/details/53926874)