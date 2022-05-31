

##  1. 注解@ConfigurationProperties

### 1. @Configuration+@ConfigurationProperties

```java
@Configuration
@ConfigurationProperties(prefix = "mail")
public class ConfigProperties {
    
    private String hostName;
    private int port;
    private String from;

    // standard getters and setters
}
```



### 2. @EnableConfigurationProperties(ConfigProperties.class)+@ConfigurationProperties

注意：如果我们不在 POJO 中使用*@Configuration*，那么我们需要在主 Spring 应用程序类中添加*@EnableConfigurationProperties(ConfigProperties.class)*来将属性绑定到 POJO 中：

```java
@SpringBootApplication
@EnableConfigurationProperties(ConfigProperties.class)
public class EnableConfigurationDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EnableConfigurationDemoApplication.class, args);
    }
}
```



### 3. @ConfigurationProperties+@ConfigurationPropertiesScan

从[Spring Boot 2.2](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2-Release-Notes#configurationproperties-scanning)开始，Spring通过类路径扫描查找并注册@ConfigurationProperties类。因此，无需使用@Component（以及其他元注释，如@Configuration）来注释此类，甚至无需使用@EnableConfigurationProperties：

```java
@ConfigurationProperties(prefix = "mail") 
public class ConfigProperties { 

    private String hostName; 
    private int port; 
    private String from; 

    // standard getters and setters 
}
```



@SpringBootApplication*启用的类路径扫描器会找到*ConfigProperties类，即使我们没有使用*@Component 注释这个类。*

此外，我们可以使用[@ConfigurationPropertiesScan](https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/api/org/springframework/boot/context/properties/ConfigurationPropertiesScan.html)注解来扫描配置属性类的自定义位置：

```java
@SpringBootApplication
@ConfigurationPropertiesScan("com.baeldung.configurationproperties")
public class EnableConfigurationDemoApplication { 

    public static void main(String[] args) {   
        SpringApplication.run(EnableConfigurationDemoApplication.class, args); 
    } 
}
```

这样，Spring 将仅在*com.baeldung.properties*包中查找配置属性类。



### **4. Nested Properties**  嵌套属性

我们可以在Lists、Maps和Classes 中拥有嵌套属性。

让我们创建一个新的*Credentials*类以用于一些嵌套属性：

```java
public class Credentials {
    private String authMethod;
    private String username;
    private String password;

    // standard getters and setters
}
```

我们还需要更新*ConfigProperties*类以使用*List、* Map*和*Credentials*类*：

```java
public class ConfigProperties {

    private String host;
    private int port;
    private String from;
    private List<String> defaultRecipients;
    private Map<String, String> additionalHeaders;
    private Credentials credentials;
 
    // standard getters and setters
}
```

以下属性文件将设置所有字段：

```properties
#Simple properties
mail.hostname=mailer@mail.com
mail.port=9000
mail.from=mailer@mail.com

#List properties
mail.defaultRecipients[0]=admin@mail.com
mail.defaultRecipients[1]=owner@mail.com

#Map Properties
mail.additionalHeaders.redelivery=true
mail.additionalHeaders.secure=true

#Object properties
mail.credentials.username=john
mail.credentials.password=password
mail.credentials.authMethod=SHA1
```



### 5.在@Bean方法上使用@ConfigurationProperties

我们还可以在@Bean注解的方法上使用@ConfigurationProperties注解。

当我们想要将属性绑定到我们无法控制的第三方组件时，这种方法可能特别有用。

让我们创建一个简单的*Item*类，我们将在下一个示例中使用它：

```java
public class Item {
    private String name;
    private int size;

    // standard getters and setters
}
```

现在让我们看看如何在*@Bean方法上使用@ConfigurationProperties将外部化属性绑定到*Item实例：

```java
@Configuration
public class ConfigProperties {

    @Bean
    @ConfigurationProperties(prefix = "item")
    public Item item() {
        return new Item();
    }
}
```

因此，任何以项目为前缀的属性都将映射到由 Spring 上下文管理的*Item实例。*



### 6. 属性验证

@ConfigurationProperties使用 JSR-303 格式提供属性验证。

例如，让我们强制hostName属性：

```java
@NotBlank
private String hostName;
```

接下来，让我们将*authMethod*属性的长度设置为 1 到 4 个字符：

```java
@Length(max = 4, min = 1)
private String authMethod;
```

然后*端口*属性从 1025 到 65536：

```java
@Min(1025)
@Max(65536)
private int port;
```

最后，*from*属性必须匹配电子邮件地址格式：

```java
@Pattern(regexp = "^[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,6}$")
private String from;
```

这有助于我们减少代码中的大量*if – else*条件，并使其看起来更加简洁明了。

如果这些验证中的任何一个失败，则主应用程序将无法以IllegalStateException启动。

Hibernate Validation 框架使用标准的 Java bean getter 和 setter，因此我们为每个属性声明 getter 和 setter 很重要。



### 7、属性转换

*@ConfigurationProperties*支持将多种类型的属性绑定到其对应的 bean 的转换。

### 7.1. Duration对象

我们将从将属性转换为*Duration*对象开始*。*

这里我们有两个*Duration*类型的字段：

```java
@ConfigurationProperties(prefix = "conversion")
public class PropertyConversion {

    private Duration timeInDefaultUnit;
    private Duration timeInNano;
    ...
}
```

这是我们的属性文件：

```bash
conversion.timeInDefaultUnit=10
conversion.timeInNano=9ns
```

结果，字段*timeInDefaultUnit*的值为 10 毫秒，*timeInNano*的值为 9 纳秒。

**支持的单位是ns、us、ms、s、m、h和d，分别表示纳秒、微秒、毫秒、秒、分钟、小时和天。**

默认单位是毫秒，这意味着如果我们不指定数值旁边的单位，Spring 会将值转换为毫秒。

我们还可以使用@DurationUnit 覆盖默认单位：

```java
@DurationUnit(ChronoUnit.DAYS)
private Duration timeInDays;
```

这是相应的属性：

```bash
conversion.timeInDays=2
```

### 7.2. 数据大小

**同样，Spring Boot @ConfigurationProperties支持DataSize类型转换。**

让我们添加三个*DataSize*类型的字段：

```java
private DataSize sizeInDefaultUnit;

private DataSize sizeInGB;

@DataSizeUnit(DataUnit.TERABYTES)
private DataSize sizeInTB;
```

这些是相应的属性：

```bash
conversion.sizeInDefaultUnit=300
conversion.sizeInGB=2GB
conversion.sizeInTB=4
```

**在这种情况下，sizeInDefaultUnit值将是 300 字节，因为默认单位是字节。**

支持的单位是*B、KB、MB、GB*和*TB。*我们还可以使用*@DataSizeUnit 覆盖默认单位。*

### 7.3. 自定义转换器

我们还可以添加我们自己的自定义*转换器*来支持将属性转换为特定的类类型。

让我们添加一个简单的类*Employee*：

```java
public class Employee {
    private String name;
    private double salary;
}
```

然后我们将创建一个自定义转换器来转换此属性：

```properties
conversion.employee=john,2000
```

我们将其转换为*Employee*类型的文件：

```java
private Employee employee;
```

我们需要实现*Converter*接口，然后**使用@ConfigurationPropertiesBinding注解注册我们的自定义Converter**：

```java
@Component
@ConfigurationPropertiesBinding
public class EmployeeConverter implements Converter<String, Employee> {

    @Override
    public Employee convert(String from) {
        String[] data = from.split(",");
        return new Employee(data[0], Double.parseDouble(data[1]));
    }
}
```

### 8. 不可变的*@ConfigurationProperties*绑定

从 Spring Boot 2.2 开始，**我们可以使用@ConstructorBinding注解来绑定我们的配置属性。**

这实质上意味着*@ConfigurationProperties* -annotated 类现在可能是[不可变](https://www.baeldung.com/java-immutable-object)的。

```java
@ConfigurationProperties(prefix = "mail.credentials")
@ConstructorBinding
public class ImmutableCredentials {

    private final String authMethod;
    private final String username;
    private final String password;

    public ImmutableCredentials(String authMethod, String username, String password) {
        this.authMethod = authMethod;
        this.username = username;
        this.password = password;
    }

    public String getAuthMethod() {
        return authMethod;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}
```

正如我们所见，当使用 *@ConstructorBinding 时，*我们需要为构造函数提供我们想要绑定的所有参数。

请注意， *ImmutableCredentials*的所有字段 都是最终的。此外，没有设置方法。

此外，需要强调的是，**要使用构造函数绑定，我们需要使用 @EnableConfigurationProperties或 @ConfigurationPropertiesScan**显式启用我们的配置类。

### 9.  Java 16 记录

Java 16 引入了 *记录* 类型作为[JEP 395](https://openjdk.java.net/jeps/395)的一部分。记录是充当不可变数据的透明载体的类。这使它们成为配置持有者和 DTO 的完美候选者。事实上，**我们可以在 Spring Boot 中将 Java 记录定义为配置属性**。例如，前面的例子可以重写为：

```java
@ConstructorBinding
@ConfigurationProperties(prefix = "mail.credentials")
public record ImmutableCredentials(String authMethod, String username, String password) {
}
```

显然，与所有那些嘈杂的 getter 和 setter 相比，它更简洁。

此外，从[Spring Boot 2.6](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6.0-M2-Release-Notes#records-and-configurationproperties)开始，对于单构造函数记录，我们可以删除 @ConstructorBinding注释。但是，如果我们的记录有多个构造函数，则仍应使用*@ConstructorBinding*来标识用于属性绑定的构造函数。

### **10. 结论**

在本文中，我们探索了*@ConfigurationProperties*注解并重点介绍了它提供的一些有用功能，例如轻松绑定和 Bean 验证。

像往常一样，代码可以[在 Github 上](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)找到。







## 2. @Import注解



```java
interface ServiceInterface {
    void test();
}

class ServiceA implements ServiceInterface {
    @Override
    public void test() {
        System.out.println("ServiceA");
    }
}

class ServiceB implements ServiceInterface {
    @Override
    public void test() {
        System.out.println("ServiceB");
    }
}
```



```java
@Import(ConfigB.class)
@Configuration
class ConfigA {
    @Bean
    @ConditionalOnMissingBean
    public ServiceInterface getServiceA() {
        return new ServiceA();
    }
}

@Configuration
class ConfigB {
    @Bean
    @ConditionalOnMissingBean
    public ServiceInterface getServiceB() {
        return new ServiceB();
    }
}
```



通过`ConfigA`创建`AnnotationConfigApplicationContext`，获取`ServiceInterface`，看是哪种实现：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigA.class);
    ServiceInterface bean = ctx.getBean(ServiceInterface.class);
    bean.test();
}
```

输出为：`ServiceB`.证明`@Import`的优先于本身的的类定义加载。

 指定实现`ImportSelector`(以及`DefferredServiceImportSelector`)的类，用于个性化加载

指定实现`ImportSelector`的类，通过`AnnotationMetadata`里面的属性，动态加载类。`AnnotationMetadata`是`Import`注解所在的类属性（如果所在类是注解类，则延伸至应用这个注解类的非注解类为止）。

需要实现`selectImports`方法，返回要加载的`@Configuation`或者具体`Bean`类的全限定名的`String`数组。

```java
package com.test;
class ServiceImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //可以是@Configuration注解修饰的类，也可以是具体的Bean类的全限定名称
        return new String[]{"com.test.ConfigB"};
    }
}

@Import(ServiceImportSelector.class)
@Configuration
class ConfigA {
    @Bean
    @ConditionalOnMissingBean
    public ServiceInterface getServiceA() {
        return new ServiceA();
    }
}
```



再次运行`main`方法，输出：`ServiceB`.证明`@Import`的优先于本身的的类定义加载。 一般的，框架中如果基于`AnnotationMetadata`的参数实现动态加载类，一般会写一个额外的`Enable`注解，配合使用。例如：

```java
package com.test;

@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.TYPE)
@Import(ServiceImportSelector.class)
@interface EnableService {
    String name();
}

class ServiceImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //这里的importingClassMetadata针对的是使用@EnableService的非注解类
        //因为`AnnotationMetadata`是`Import`注解所在的类属性，如果所在类是注解类，则延伸至应用这个注解类的非注解类为止
        Map<String , Object> map = importingClassMetadata.getAnnotationAttributes(EnableService.class.getName(), true);
        String name = (String) map.get("name");
        if (Objects.equals(name, "B")) {
            return new String[]{"com.test.ConfigB"};
        }
        return new String[0];
    }
}
```

之后，在`ConfigA`中增加注解`@EnableService(name = "B")`

```java
package com.test;
@EnableService(name = "B")
@Configuration
class ConfigA {
    @Bean
    @ConditionalOnMissingBean
    public ServiceInterface getServiceA() {
        return new ServiceA();
    }
}
```

再次运行`main`方法，输出：`ServiceB`.

还可以实现`DeferredImportSelector`接口,这样`selectImports`返回的类就都是最后加载的，而不是像`@Import`注解那样，先加载。 例如：

```java
package com.test;
class DefferredServiceImportSelector implements DeferredImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        Map<String, Object> map = importingClassMetadata.getAnnotationAttributes(EnableService.class.getName(), true);
        String name = (String) map.get("name");
        if (Objects.equals(name, "B")) {
            return new String[]{"com.test.ConfigB"};
        }
        return new String[0];
    }
}
```

修改`EnableService`注解：

```java
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.TYPE)
@Import(DefferredServiceImportSelector.class)
@interface EnableService {
    String name();
}
```

这样`ConfigA`就优先于`DefferredServiceImportSelector`返回的`ConfigB`加载，执行`main`方法，输出：`ServiceA`

4. 指定实现`ImportBeanDefinitionRegistrar`的类，用于个性化加载

与`ImportSelector`用法与用途类似，但是如果我们想重定义`Bean`，例如动态注入属性，改变`Bean`的类型和`Scope`等等，就需要通过指定实现`ImportBeanDefinitionRegistrar`的类实现。例如：

定义`ServiceC`

```java
package com.test;
class ServiceC implements ServiceInterface {

    private final String name;

    ServiceC(String name) {
        this.name = name;
    }

    @Override
    public void test() {
        System.out.println(name);
    }
}
```

定义`ServiceImportBeanDefinitionRegistrar`动态注册`ServiceC`，修改`EnableService`

```java
package com.test;

@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.TYPE)
@Import(ServiceImportBeanDefinitionRegistrar.class)
@interface EnableService {
    String name();
}

class ServiceImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, 
                                        BeanDefinitionRegistry registry) {
        Map<String, Object> map = 
            importingClassMetadata.getAnnotationAttributes(EnableService.class.getName(), true);
        String name = (String) map.get("name");
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.rootBeanDefinition(ServiceC.class)
                //增加构造参数
                .addConstructorArgValue(name);
        //注册Bean
        registry.registerBeanDefinition("serviceC", beanDefinitionBuilder.getBeanDefinition());
    }
}
```

并且根据后面的源代码解析可以知道，`ImportBeanDefinitionRegistrar` 在 `@Bean` 注解之后加载，所以要修改`ConfigA`去掉其中被`@ConditionalOnMissingBean`注解的`Bean`，否则一定会生成`ConfigA`的`ServiceInterface`

```java
package com.test;
@EnableService(name = "TestServiceC")
@Configuration
class ConfigA {
//    @Bean
//    @ConditionalOnMissingBean
//    public ServiceInterface getServiceA() {
//        return new ServiceA();
//    }
}
```

之后运行`main`，输出：`TestServiceC`



### `@Import`相关源码解析

加载解析`@Import`注解位于`BeanFactoryPostProcessor`处理的时候：

`AbstractApplicationContext`的`refresh`方法

-> `invokeBeanFactoryPostProcessors(beanFactory);`

-> `PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());`

-> `registryProcessor.postProcessBeanDefinitionRegistry(registry);`

这里的`registryProcessor`,我们指`ConfigurationClassPostProcessor`

```java
ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(registry)
```

-> `processConfigBeanDefinitions(registry)`:

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    //省略一些配置检查与设置的逻辑

    //根据@Order注解，排序所有的@Configuration类
    configCandidates.sort((bd1, bd2) -> {
			int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
			int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
			return Integer.compare(i1, i2);
		});

	// 创建ConfigurationClassParser解析@Configuration类
	ConfigurationClassParser parser = new ConfigurationClassParser(
			this.metadataReaderFactory, this.problemReporter, this.environment,
			this.resourceLoader, this.componentScanBeanNameGenerator, registry);

    //剩余没有解析的@Configuration类
	Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
	//已经解析的@Configuration类
	Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
	do {
	    //解析
		parser.parse(candidates);
		parser.validate();

		Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
		configClasses.removeAll(alreadyParsed);

		// 生成类定义读取器读取类定义
		if (this.reader == null) {
			this.reader = new ConfigurationClassBeanDefinitionReader(
					registry, this.sourceExtractor, this.resourceLoader, this.environment,
					this.importBeanNameGenerator, parser.getImportRegistry());
		}
		this.reader.loadBeanDefinitions(configClasses);
		alreadyParsed.addAll(configClasses);

		candidates.clear();
		if (registry.getBeanDefinitionCount() > candidateNames.length) {
			//省略检查是否有其他需要加载的配置的逻辑
		}
	}
	while (!candidates.isEmpty());

	//省略后续清理逻辑
}
```

其中`parser.parse(candidates)`的逻辑主要由`org.springframework.context.annotation.ConfigurationClassParser`实现，功能是加载`@Import`注解还有即系`@Import`注解。`reader.loadBeanDefinitions(configClasses);`的逻辑主要由`org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader`的`loadBeanDefinitionsForConfigurationClass`方法实现，功能是将上面解析的配置转换为`BeanDefinition`就是`Bean`定义。

#### 1. 加载`@Import`注解

```java
org.springframework.context.annotation.ConfigurationClassParser
```

首先是`parse`方法

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
	for (BeanDefinitionHolder holder : configCandidates) {
		BeanDefinition bd = holder.getBeanDefinition();
		try {
			if (bd instanceof AnnotatedBeanDefinition) {
			    //这里的parse实际上就是调用下面即将分析的doProcessConfigurationClass
				parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
			}
			else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
				parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
			}
			else {
				parse(bd.getBeanClassName(), holder.getBeanName());
			}
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(
					"Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
		}
	}
    //最后处理所有的`DeferredImportSelector`，符合上面提到的`DeferredImportSelector`的功能
	this.deferredImportSelectorHandler.process();
}
@Nullable
protected final SourceClass doProcessConfigurationClass(
			ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
			throws IOException {
	//处理`@Component`注解的MemberClass相关代码...
	//处理`@PropertySource`注解相关代码...
	//处理`@ComponentScan`注解相关代码...
    //处理`@Import`注解：
    processImports(configClass, sourceClass, getImports(sourceClass), filter, true);
    //处理`@ImportResource`注解相关代码...
    //处理`@Bean`注解相关代码...
    //处理接口方法相关代码...
    //处理父类相关代码...
}
```

通过`getImports`方法，采集相关的`@Import`里面的类。

```java
private Set<SourceClass> getImports(SourceClass sourceClass) throws IOException {
	Set<SourceClass> imports = new LinkedHashSet<>();
	Set<SourceClass> visited = new LinkedHashSet<>();
	//递归查询所有注解以及注解的注解是否包含@Import
	collectImports(sourceClass, imports, visited);
	return imports;
}
private void collectImports(SourceClass sourceClass, Set<SourceClass> imports, Set<SourceClass> visited)
			throws IOException {
    //记录是否已经扫描过这个类，如果扫描过就不重复添加，防止重复或者死循环
	if (visited.add(sourceClass)) {
		for (SourceClass annotation : sourceClass.getAnnotations()) {
			String annName = annotation.getMetadata().getClassName();
			//对于非@Import注解，递归查找其内部是否包含@Import注解
			if (!annName.equals(Import.class.getName())) {
				collectImports(annotation, imports, visited);
			}
		}
		//添加@Import注解里面的所有配置类
		imports.addAll(sourceClass.getAnnotationAttributes(Import.class.getName(), "value"));
	}
}
```

采集好之后，就可以解析了。

#### 2. 解析`@Import`注解

解析的方法是：`processImports`

```java
//在解析时，入栈，解析结束后，出栈，通过检查栈中是否有当前类，判断是否有循环依赖
private final ImportStack importStack = new ImportStack();

//记录所有的ImportBeanDefinitionRegistrar
private final Map<ImportBeanDefinitionRegistrar, AnnotationMetadata> importBeanDefinitionRegistrars = new LinkedHashMap<>();

//解析也是递归方法
private void processImports(ConfigurationClass configClass, SourceClass currentSourceClass,
			Collection<SourceClass> importCandidates, Predicate<String> exclusionFilter,
			boolean checkForCircularImports) {

	if (importCandidates.isEmpty()) {
		return;
	}
    //通过importStack检查循环依赖
	if (checkForCircularImports && isChainedImportOnStack(configClass)) {
		this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
	}
	else {
	    //入栈
		this.importStack.push(configClass);
		try {
			for (SourceClass candidate : importCandidates) {
				if (candidate.isAssignable(ImportSelector.class)) {
					//处理ImportSelector接口的实现类
					Class<?> candidateClass = candidate.loadClass();
					//创建这些Selector实例
					ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
							this.environment, this.resourceLoader, this.registry);
				    //查看是否有过滤器
					Predicate<String> selectorFilter = selector.getExclusionFilter();
					if (selectorFilter != null) {
						exclusionFilter = exclusionFilter.or(selectorFilter);
					}
					//如果是DeferredImportSelector，则用deferredImportSelectorHandler处理
					if (selector instanceof DeferredImportSelector) {
						this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
					}
					else {
					    //如果不是DeferredImportSelector，调用selectImports方法获取要加载的类全限定名称，递归调用本方法继续解析
						String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
						Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames, exclusionFilter);
						processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
					}
				}
				else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
				    
					// 处理ImportBeanDefinitionRegistrar接口的实现类
					Class<?> candidateClass = candidate.loadClass();
					//同样的，创建这些ImportBeanDefinitionRegistrar实例
					ImportBeanDefinitionRegistrar registrar =
							ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
									this.environment, this.resourceLoader, this.registry);
					//放入importBeanDefinitionRegistrar，用于后面加载
				configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
				}
				else {
					//处理@Configuration注解类，或者是普通类（直接生成Bean）
					//在栈加上这个类
					this.importStack.registerImport(
							currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
					//递归回到doProcessConfigurationClass处理@Configuration注解类	processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);
				}
			}
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(
					"Failed to process import candidates for configuration class [" +
					configClass.getMetadata().getClassName() + "]", ex);
		}
		finally {
			this.importStack.pop();
		}
	}
}
```

这样，所有的`@Conditional`类相关的`@Import`注解就加载解析完成了，这是一个大的递归过程。

#### 3. 转换为`BeanDefinition`注册到容器

`org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader`的`loadBeanDefinitionsForConfigurationClass`方法:

```java
private void loadBeanDefinitionsForConfigurationClass(
			ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {

	if (trackedConditionEvaluator.shouldSkip(configClass)) {
		String beanName = configClass.getBeanName();
		if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
			this.registry.removeBeanDefinition(beanName);
		}
		this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
		return;
	}

    //对Import完成的，加载其Import的BeanDefinition
	if (configClass.isImported()) {
		registerBeanDefinitionForImportedConfigurationClass(configClass);
	}
	//加载@Bean注解的方法生成的Bean的Definition
	for (BeanMethod beanMethod : configClass.getBeanMethods()) {
		loadBeanDefinitionsForBeanMethod(beanMethod);
	}
    //@ImportResource 注解加载的
	loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
	//加载ImportBeanDefinitionRegistrar加载的Bean的Definition
	loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}
```

通过这里可以看出，为啥之前说`@Bean`注解的`Bean`会优先于`ImportBeanDefinitionRegistrar`返回的`Bean`加载。



