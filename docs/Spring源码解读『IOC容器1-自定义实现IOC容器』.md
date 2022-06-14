# [Spring源码解读『IOC容器1-自定义实现IOC容器』](http://lidol.top/frame/2513/)

2020-03-04 分类：[Spring](http://lidol.top/category/frame/spring/) / [框架](http://lidol.top/category/frame/) 阅读(600) 评论(0)

上篇文章介绍了Spring的相关基础概念，我们了解到Spring Framework提供的两个基础功能就是IOC和AOP。关于IOC容器我们分为两篇文章来介绍，本篇文章会介绍IOC容器的基础概念，并自定义实现一个基础的IOC容器，帮助我们后续更好的解读IOC源码。AOP会在之后的文章中介绍。

## 1. IOC容器基本概念

IOC（Inversion Of Control）也叫控制反转，这个概念经常会伴随另一个概念DI（Dependence Injection）依赖注入出现。直白的讲就是，**Java中一个对象依赖另一个对象，当我们使用这个对象时，需要主动去创建或查找这个依赖的对象，而IOC就是借助容器在对象初始化时主动将依赖传递给它。**通过容器初始化对象的过程，就是发生了控制反转，因为对象初始化的控制的控制由程序员反转到容器。同时容器初始化对象的过程就是根据对象的依赖，主动将对象依赖注入的，也就是DI的概念。

这时，我们就比较好了解**IOC容器**的概念了：**IOC容器首先是个容器，内部持有很多对象，并管理着这些对象的生命周期，支持对象的查找和依赖决策，以及配置管理等其他功能。**

举个例子，在做项目时，需要其他多个部门的人提供API，一般都是我分别主动找相关部门的人沟通让他们提供。假如有个项目经理，我给他一份API需求文档，等我需要使用API时，发现已经提供好了可以直接使用了。从这个例子可以发现，**因为项目经理的存在，我不用直接跟各个部门去对接了，只需要一个API需求文档给到项目经理就可以了，当我的需求变更时，只需要更新一下API需求文档，项目经理就会为我提供新的API**。可以看到，项目经理的存在，最直接的好处就是，我跟对方部门解耦了，简化了我的工作。而这个项目经理的角色就相当于IOC容器，我给项目经理的API需求文档，就是IOC容器依赖注入的根据配置文件。

## 2. 自定义实现IOC容器

通过上面的介绍，我们应该可以了解到IOC容器的需要完成以下两件工作：

- 实例化对象
- 根据配置，解读对象的依赖，并填充对象的相关依赖

现在我们以xml文件为配置信息载体，构建一个公司的Demo。公司(Company)开业有名字(name)，也要有员工(Employee)。

### 2.1 业务类定义

- **Company**

```java
package com.zhuoli.service.min.spring.ioc.bean;

public class Company {
    private String name;

    private Employee employee;

    public void open(){
        System.out.println("Company " + name + " is open.");
        employee.work();
    }
}
```

- **Employee**

```java
package com.zhuoli.service.min.spring.ioc.bean;

public class Employee {
    private String name;

    public void work(){
        System.out.println("Employee " + name + " is working");
    }
}
```

- **配置文件min-ioc.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans>
    <bean id="company" class="com.zhuoli.service.min.spring.ioc.bean.Company">
        <property name="name" value="Apple"></property>
        <property name="employee" ref="employee"></property>
    </bean>
    <bean id="employee" class="com.zhuoli.service.min.spring.ioc.bean.Employee">
        <property name="name" value="zhuoli"></property>
    </bean>
</beans>
```

### 2.2 Spring IOC实现

#### 2.2.1 配置文件读取

因为我们是通过xml配置文件通知自定义IOC容器所需的Java对象以及其依赖（本文中之后叫做bean），所以自定义IOC容器首先要支持xml文件的读取，将文件读取为字节流。这里我们定义一个Resource接口：

```java
package com.zhuoli.service.min.spring.ioc.rsource;

import java.io.IOException;
import java.io.InputStream;

public interface Resource {
    InputStream getInputStream() throws IOException;
}
```

通常资源文件都会放在classpath路径下，实现一个ClassPathResource类可以读取classpath的文件：

```java
package com.zhuoli.service.min.spring.ioc.rsource;

import java.io.InputStream;

public class ClassPathResource implements Resource {

    private String path;

    private ClassLoader classLoader;

    public ClassPathResource(String path) {
        this(path, (ClassLoader) null);
    }

    public ClassPathResource(String path, ClassLoader classLoader) {
        this.path = path;
        this.classLoader = (classLoader != null ? classLoader : Thread.currentThread().getContextClassLoader());
    }

    @Override
    public InputStream getInputStream() {
        return classLoader.getResourceAsStream(path);
    }
}
```

资源也可以通过其他方式定义，比如绝对路径或者URL的方式，因而需要一个Resource的处理器ResourceLoader。当前我们就只处理classpath方式：

```java
package com.zhuoli.service.min.spring.ioc.rsource;

public class ResourceLoader {
    final String CLASSPATH_URL_PREFIX = "classpath:";

    public Resource getResource(String location) {
        if (location.startsWith(CLASSPATH_URL_PREFIX)) {
            return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()));
        } else {
            return new ClassPathResource(location);
        }
    }
}
```

这样就可以通过ResourceLoader来获得min-ioc.xml文件的字节流：

```java
InputStream inputStream = new ResourceLoader().getResource("classpath:min-ioc.xml").getInputStream;
```

#### 2.2.2 解析xml配置的bean依赖

有了min-ioc.xml文件的输入流，就可以进行bean及bean关系的解析，在解析之前，我们需要一个数据结构存储解析后的数据。为什么不直接存储对象？因为spring中可以设置lazy-init=true，从而在真正需要bean对象时才创建；另一个是spring中可以设置bean的scope为prototype，也就是bean不是单例的，每次获取的都是一个新的bean对象。这样就要求我们只能存储bean的数据结构，而不能直接存储bean对象。所以xml配置文件解析的存储对象至少需要包含以下几部分：

- bean名称
- bean的Class包路径或Class对象
- 依赖属性

**xml配置文件bean解析的数据结构BeanDefinition：**

```java
package com.zhuoli.service.min.spring.ioc.beans;

public class BeanDefinition {
    // bean名称
    private String beanName;
    // bean的class对象
    private Class beanClass;
    // bean的class的包路径
    private String beanClassName;
    // bean依赖属性
    private PropertyValues propertyValues = new PropertyValues();

    public String getBeanName() {
        return beanName;
    }

    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }

    public Class getBeanClass() {
        return beanClass;
    }

    public void setBeanClass(Class beanClass) {
        this.beanClass = beanClass;
    }

    public String getBeanClassName() {
        return beanClassName;
    }

    public void setBeanClassName(String beanClassName) {
        this.beanClassName = beanClassName;
    }

    public PropertyValues getPropertyValues() {
        return propertyValues;
    }

    public void setPropertyValues(PropertyValues propertyValues) {
        this.propertyValues = propertyValues;
    }
}
```

其中PropertyValues用来存储依赖属性：

```java
package com.zhuoli.service.min.spring.ioc.beans;

import java.util.LinkedList;
import java.util.List;

public class PropertyValues {
    private List<PropertyValue> propertyValues = new LinkedList<>();

    public void addPropertyValue(PropertyValue propertyValue) {
        propertyValues.add(propertyValue);
    }

    public List<PropertyValue> getPropertyValues() {
        return this.propertyValues;
    }
}
package com.zhuoli.service.min.spring.ioc.beans;

public class PropertyValue {
    private final String name;
    private final Object value;

    public PropertyValue(String name, Object value) {
        this.name = name;
        this.value = value;
    }

    public String getName() {
        return name;
    }

    public Object getValue() {
        return value;
    }
}
```

**BeanDefinition读取：**

定义一个接口，该接口功能就是将指定位置的配置文件转化为BeanDefinition：

```java
package com.zhuoli.service.min.spring.ioc.xml;

public interface BeanDefinitionReader {
    void loadBeanDefinitions(String location) throws Exception;
}
```

实现xml文件的BeanDefinition读取类XmlBeanDefinitionReader：

```java
package com.zhuoli.service.min.spring.ioc.xml;

import com.zhuoli.service.min.spring.ioc.beans.BeanDefinition;
import com.zhuoli.service.min.spring.ioc.beans.BeanDefinitionRegistry;
import com.zhuoli.service.min.spring.ioc.beans.BeanReference;
import com.zhuoli.service.min.spring.ioc.beans.PropertyValue;
import com.zhuoli.service.min.spring.ioc.rsource.ResourceLoader;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import java.io.InputStream;

public class XmlBeanDefinitionReader implements BeanDefinitionReader {
    // BeanDefinition注册到BeanFactory接口
    private BeanDefinitionRegistry registry;
    // 资源载入类
    private ResourceLoader resourceLoader;

    public XmlBeanDefinitionReader(BeanDefinitionRegistry registry, ResourceLoader resourceLoader) {
        this.registry = registry;
        this.resourceLoader = resourceLoader;
    }

    @Override
    public void loadBeanDefinitions(String location) throws Exception {
        InputStream is = getResourceLoader().getResource(location).getInputStream();
        doLoadBeanDefinitions(is);
    }

    public BeanDefinitionRegistry getRegistry() {
        return registry;
    }

    public ResourceLoader getResourceLoader() {
        return resourceLoader;
    }

    protected void doLoadBeanDefinitions(InputStream is) throws Exception {
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = factory.newDocumentBuilder();
        Document document = builder.parse(is);
        registerBeanDefinitions(document);
        is.close();
    }

    protected void registerBeanDefinitions(Document document) {
        Element root = document.getDocumentElement();
        parseBeanDefinitions(root);
    }

    protected void parseBeanDefinitions(Element root) {
        NodeList nodeList = root.getChildNodes();
        for (int i = 0; i < nodeList.getLength(); i++) {
            Node item = nodeList.item(i);
            if (item instanceof Element) {
                Element ele = (Element) item;
                processBeanDefinition(ele);
            }
        }
    }

    protected void processBeanDefinition(Element ele) {
        String name = ele.getAttribute("id");
        String className = ele.getAttribute("class");
        if (className == null || className.length() == 0) {
            throw new IllegalArgumentException("Configuration exception: <bean> element must has class attribute.");
        }
        if (name == null || name.length() == 0) {
            name = className;
        }
        BeanDefinition beanDefinition = new BeanDefinition();
        beanDefinition.setBeanClassName(className);
        processBeanProperty(ele, beanDefinition);
        getRegistry().registerBeanDefinition(name, beanDefinition);
    }

    protected void processBeanProperty(Element ele, BeanDefinition beanDefinition) {
        NodeList children = ele.getElementsByTagName("property");
        for (int i = 0; i < children.getLength(); i++) {
            Node node = children.item(i);
            if (node instanceof Element) {
                Element property = (Element) node;
                String name = property.getAttribute("name");
                String value = property.getAttribute("value");
                if (value != null && value.length() > 0) {
                    beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name, value));
                } else {
                    String ref = property.getAttribute("ref");
                    if (ref == null || ref.length() == 0) {
                        throw new IllegalArgumentException("Configuration problem: <property> element for " +
                                name + " must specify a value or ref.");
                    }
                    BeanReference reference = new BeanReference(ref);
                    beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name, reference));
                }
            }
        }
    }
}
```

XmlBeanDefinitionReader的构造参数传入资源文件的载入类和BeanDefinition注册接口实现类，用来定位xml文件和注册BeanDefinition，所以在XmlBeanDefinitionReader中实现了Bean的定位，解析和之后的注册。

在loadBeanDefinitions方法中，通过ResourceLoader获取到xml文件的InputStream，完成了定位操作，解析操作转到doLoadBeanDefinitions方法中。

```java
protected void doLoadBeanDefinitions(InputStream is) throws Exception {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    DocumentBuilder builder = factory.newDocumentBuilder();
    Document document = builder.parse(is);
    registerBeanDefinitions(document);
    is.close();
}

protected void registerBeanDefinitions(Document document) {
    Element root = document.getDocumentElement();
    parseBeanDefinitions(root);
}
```

DOM方式解析xml，拿到Document对象，再拿到根节点，转到parseBeanDefinitions解析具体的 <beans> 信息。

```java
protected void parseBeanDefinitions(Element root) {
    NodeList nodeList = root.getChildNodes();
    for (int i = 0; i < nodeList.getLength(); i++) {
        Node item = nodeList.item(i);
        if (item instanceof Element) {
            Element ele = (Element) item;
            processBeanDefinition(ele);
        }
    }
}

protected void processBeanDefinition(Element ele) {
    String name = ele.getAttribute("id");
    String className = ele.getAttribute("class");
    if (className == null || className.length() == 0) {
        throw new IllegalArgumentException("Configuration exception: <bean> element must has class attribute.");
    }
    if (name == null || name.length() == 0) {
        name = className;
    }
    BeanDefinition beanDefinition = new BeanDefinition();
    beanDefinition.setBeanClassName(className);
    processBeanProperty(ele, beanDefinition);
    getRegistry().registerBeanDefinition(name, beanDefinition);
}
```

遍历<beans>下的每一个<bean>节点，获取属性id和class的值，分别对应BeanDefinition的beanName和BeanClassName。然后解析依赖属性<property>标签：

```java
protected void processBeanProperty(Element ele, BeanDefinition beanDefinition) {
    NodeList children = ele.getElementsByTagName("property");
    for (int i = 0; i < children.getLength(); i++) {
        Node node = children.item(i);
        if (node instanceof Element) {
            Element property = (Element) node;
            String name = property.getAttribute("name");
            String value = property.getAttribute("value");
            if (value != null && value.length() > 0) {
                beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name, value));
            } else {
                String ref = property.getAttribute("ref");
                if (ref == null || ref.length() == 0) {
                    throw new IllegalArgumentException("Configuration problem: <property> element for " +
                            name + " must specify a value or ref.");
                }
                BeanReference reference = new BeanReference(ref);
                beanDefinition.getPropertyValues().addPropertyValue(new PropertyValue(name, reference));
            }
        }
    }
}
```

如果<property>中存在value属性，则作为String类型存储，如果是ref属性，则创建BeanReference对象存储。

```java
package com.zhuoli.service.min.spring.ioc.beans;

public class BeanReference {
    private String ref;

    public BeanReference(String ref) {
        this.ref = ref;
    }

    public String getRef() {
        return ref;
    }
}
```

最后注册BeanDefinition到存储BeanDefinition的地方，我们可以猜测肯定有一个Map<String, BeanDefinition>。

```java
getRegistry().registerBeanDefinition(name, beanDefinition);
```

#### 2.2.3 注册BeanDefinition

上面的getRegistry()获得的就是XmlBeanDefinitionReader构造时传入的BeanDefinitionRegistry实现类。首先BeanDefinitionRegistry是个接口：

```java
package com.zhuoli.service.min.spring.ioc.beans;

public interface BeanDefinitionRegistry {
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition);
}
```

它只定义了一个方法，就是注册BeanDefinition。它的实现类就是要实现这个方法并将BeanDefinition注册到Map<String,BeanDefinition>中。在这里这个实现类是DefaultListableBeanFactory：

```java
package com.zhuoli.service.min.spring.ioc.factory;

import com.zhuoli.service.min.spring.ioc.beans.BeanDefinition;
import com.zhuoli.service.min.spring.ioc.beans.BeanDefinitionRegistry;

import java.util.LinkedList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class DefaultListableBeanFactory extends AbstractBeanFactory implements BeanDefinitionRegistry {
    private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();
    private List<String> beanDefinitionNames = new LinkedList<>();

    public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) {
        beanDefinitionMap.put(beanName, beanDefinition);
        beanDefinitionNames.add(beanName);
    }

    @Override
    protected BeanDefinition getBeanDefinitionByName(String beanName) {
        return beanDefinitionMap.get(beanName);
    }

    @Override
    public void preInstantiateSingletons() throws Exception {
        for (String beanName : beanDefinitionNames) {
            getBean(beanName);
        }
    }
}
```

可以看出它包含一个beanDefinitionMap 和 beanDefinitionNames，一个记录beanName和BeanDefinition的关系，一个记录所有BeanName。而registerBeanDefinition方法就是将BeanDefinition添加到beanDefinitionMap和beanDefinitionNames。

#### 2.2.4 获取bean

DefaultListableBeanFactory实现了ConfigurableListableBeanFactory接口，而ConfigurableListableBeanFactory继承了BeanFactory接口。BeanFactory是一个顶级接口：

```java
package com.zhuoli.service.min.spring.ioc.factory;

public interface BeanFactory {
    Object getBean(String beanName);
}
```

它定义了获取bean对象的接口方法，实现这个方法的是AbstractBeanFactory抽象类：

```java
package com.zhuoli.service.min.spring.ioc.factory;

import com.zhuoli.service.min.spring.ioc.beans.BeanDefinition;
import com.zhuoli.service.min.spring.ioc.beans.BeanReference;
import com.zhuoli.service.min.spring.ioc.beans.PropertyValue;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public abstract class AbstractBeanFactory implements ConfigurableListableBeanFactory {
    private Map<String, Object> singleObjects = new ConcurrentHashMap<>();

    @Override
    public Object getBean(String beanName) {
        Object singleBean = this.singleObjects.get(beanName);
        if (singleBean != null) {
            return singleBean;
        }

        BeanDefinition beanDefinition = getBeanDefinitionByName(beanName);
        if (beanDefinition == null) {
            throw new RuntimeException("bean for name '" + beanName + "' not register.");
        }

        singleBean = doCreateBean(beanDefinition);
        this.singleObjects.put(beanName, singleBean);
        return singleBean;
    }

    protected abstract BeanDefinition getBeanDefinitionByName(String beanName);

    protected Object doCreateBean(BeanDefinition beanDefinition) {
        Object bean = createInstance(beanDefinition);
        applyPropertyValues(bean, beanDefinition);
        return bean;
    }

    protected Object createInstance(BeanDefinition beanDefinition) {
        try {
            if (beanDefinition.getBeanClass() != null) {
                return beanDefinition.getBeanClass().newInstance();
            } else if (beanDefinition.getBeanClassName() != null) {
                try {
                    Class clazz = Class.forName(beanDefinition.getBeanClassName());
                    beanDefinition.setBeanClass(clazz);
                    return clazz.newInstance();
                } catch (ClassNotFoundException e) {
                    throw new RuntimeException("bean Class " + beanDefinition.getBeanClassName() + " not found");
                }
            }
        } catch (Exception e) {
            throw new RuntimeException("create bean " + beanDefinition.getBeanName() + " failed");
        }
        throw new RuntimeException("bean name for " + beanDefinition.getBeanName() + " not define bean class");
    }

    protected void applyPropertyValues(Object bean, BeanDefinition beanDefinition) {
        for (PropertyValue propertyValue : beanDefinition.getPropertyValues().getPropertyValues()) {
            String name = propertyValue.getName();
            Object value = propertyValue.getValue();
            if (value instanceof BeanReference) {
                BeanReference reference = (BeanReference) value;
                value = getBean(reference.getRef());
            }
            try {
                Method method = bean.getClass().getDeclaredMethod("set" + name.substring(0, 1).toUpperCase() +
                        name.substring(1), value.getClass());
                method.setAccessible(true);
                method.invoke(bean, value);
            } catch (Exception e) {
                try {
                    Field field = bean.getClass().getDeclaredField(name);
                    field.setAccessible(true);
                    field.set(bean, value);
                } catch (Exception e1) {
                    throw new RuntimeException("inject bean property " + name + " failed");
                }
            }
        }
    }
}
```

它定义了获取bean对象的接口方法getBean，这个方法的实现是在AbstractBeanFactory抽象类：

```java
@Override
public Object getBean(String beanName) {
    Object singleBean = this.singleObjects.get(beanName);
    if (singleBean != null) {
        return singleBean;
    }

    BeanDefinition beanDefinition = getBeanDefinitionByName(beanName);
    if (beanDefinition == null) {
        throw new RuntimeException("bean for name '" + beanName + "' not register.");
    }

    singleBean = doCreateBean(beanDefinition);
    this.singleObjects.put(beanName, singleBean);
    return singleBean;
}
```

AbstractBeanFactory中定义了单实例(Demo默认都是单实例的bean)的beanName和bean对象关系的singleObjects的Map。getBean方法传入beanName，先查询singleObjects有没有beanName的缓存，如果存在，直接返回；如果不存在则调用getBeanDefinitionByName获取BeanDefinition（本质上是从Bean注册中心获取，而DefaultListableBeanFactory类实现了BeanDefinitionRegistry接口），DefaultListableBeanFactory类中实现了getBeanDefinitionByName方法：

```java
@Override
protected BeanDefinition getBeanDefinitionByName(String beanName) {
    return beanDefinitionMap.get(beanName);
}
```

拿到BeanDefinition后，进入真正创建bean的方法doCreateBean：

```
protected Object doCreateBean(BeanDefinition beanDefinition) {
    Object bean = createInstance(beanDefinition);
    applyPropertyValues(bean, beanDefinition);
    return bean;
}
```

它做了两个操作，一个是创建bean对象，另一个是注入依赖属性：

```java
protected Object createInstance(BeanDefinition beanDefinition) {
    try {
        if (beanDefinition.getBeanClass() != null) {
            return beanDefinition.getBeanClass().newInstance();
        } else if (beanDefinition.getBeanClassName() != null) {
            try {
                Class clazz = Class.forName(beanDefinition.getBeanClassName());
                beanDefinition.setBeanClass(clazz);
                return clazz.newInstance();
            } catch (ClassNotFoundException e) {
                throw new RuntimeException("bean Class " + beanDefinition.getBeanClassName() + " not found");
            }

        }
    } catch (Exception e) {
        throw new RuntimeException("create bean " + beanDefinition.getBeanName() + " failed");
    }
    throw new RuntimeException("bean name for " + beanDefinition.getBeanName() + " not define bean class");
}
```

通过Class.forName加载bean的Class对象，再通过Class的newInstance方法反射生成bean对象。

```java
protected void applyPropertyValues(Object bean, BeanDefinition beanDefinition) {
    for (PropertyValue propertyValue : beanDefinition.getPropertyValues().getPropertyValues()) {
        String name = propertyValue.getName();
        Object value = propertyValue.getValue();
        if (value instanceof BeanReference) {
            BeanReference reference = (BeanReference) value;
            value = getBean(reference.getRef());
        }
        try {
            Method method = bean.getClass().getDeclaredMethod("set" + name.substring(0, 1).toUpperCase() +
                    name.substring(1), value.getClass());
            method.setAccessible(true);
            method.invoke(bean, value);
        } catch (Exception e) {
            try {
                Field field = bean.getClass().getDeclaredField(name);
                field.setAccessible(true);
                field.set(bean, value);
            } catch (Exception e1) {
                throw new RuntimeException("inject bean property " + name + " failed");
            }
        }
    }
}
```

如果依赖属性是对象的话，通过getBean获取或生成依赖对象，并通过反射注入到bean对象中，这也就是依赖注入。对于以来的属性是对象的情况，如果对象间存在循环依赖，我们实现的IOC容器是无法解决的，但是Spring的IOC容器可以完美地解决这个问题，我们会在后续的文章中介绍。至此，Bean的创建和获取就完成了。

#### 2.2.5 ApplicationContext

在我们日常使用spring时，一般不会直接用BeanFactory，而是通过一个更加高级的接口ApplicationContext来初始化容器以及提供其他企业级的功能。在我们的简易IOC容器中，通过ApplicationContext来实例化所有的bean。

```java
package com.zhuoli.service.min.spring.ioc.context;

import com.zhuoli.service.min.spring.ioc.factory.BeanFactory;

public interface ApplicationContext extends BeanFactory {

}
```

ApplicationContext没有提供其他功能，只是继承了BeanFactory。ApplicationContext的一个最终实现类ClasspathXmlApplicationContext：

```java
package com.zhuoli.service.min.spring.ioc.context;

import com.zhuoli.service.min.spring.ioc.beans.BeanDefinitionRegistry;
import com.zhuoli.service.min.spring.ioc.rsource.ResourceLoader;
import com.zhuoli.service.min.spring.ioc.xml.XmlBeanDefinitionReader;

public class ClasspathXmlApplicationContext extends AbstractApplicationContext {
    private String location;

    public ClasspathXmlApplicationContext(String location) {
        this.location = location;
        try {
            refresh();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void loadBeanDefinitions(BeanDefinitionRegistry registry) throws Exception {
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry, new ResourceLoader());
        reader.loadBeanDefinitions(location);
    }
}
```

ClasspathXmlApplicationContext的构造函数里接收传入xml资源文件的路径location，再调用refresh方法。refresh方法在AbstractApplicationContext抽象类中实现：

```java
package com.zhuoli.service.min.spring.ioc.context;

import com.zhuoli.service.min.spring.ioc.beans.BeanDefinitionRegistry;
import com.zhuoli.service.min.spring.ioc.factory.ConfigurableListableBeanFactory;
import com.zhuoli.service.min.spring.ioc.factory.DefaultListableBeanFactory;

public abstract class AbstractApplicationContext implements ApplicationContext {
    private ConfigurableListableBeanFactory beanFactory;

    @Override
    public Object getBean(String beanName) {
        return beanFactory.getBean(beanName);
    }

    public void refresh() throws Exception{
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        loadBeanDefinitions(beanFactory);
        this.beanFactory = beanFactory;
        onRefresh();
    }

    protected void onRefresh() throws Exception {
        beanFactory.preInstantiateSingletons();
    }

    protected abstract void loadBeanDefinitions(BeanDefinitionRegistry registry) throws Exception;
}
```

refresh方法先创建一个DefaultListableBeanFactory，然后通过loadBeanDefinitions解析并注册bean，真正的实现在ClasspathXmlApplicationContext中：

```java
@Override
protected void loadBeanDefinitions(BeanDefinitionRegistry registry) throws Exception {
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry, new ResourceLoader());
    reader.loadBeanDefinitions(location);
}
```

就是通过XmlBeanDefinitionReader的loadBeanDefinitions方法来处理（加载xml配置解析为BeanDefinition，注册到Bean注册中心BeanDefinitionRegistry），这在上面已经详细的讲过了，就不再赘述了。

refresh方法中最后调用onRefresh方法，初始化所有bean实例：

```java
protected void onRefresh() throws Exception {
    beanFactory.preInstantiateSingletons();
}
```

实际调用的是DefaultListableBeanFactory类，方法定义在ConfigurableListableBeanFactory接口中：

```java
package com.zhuoli.service.min.spring.ioc.factory;

public interface ConfigurableListableBeanFactory extends BeanFactory{
    void preInstantiateSingletons() throws Exception;
}
```

DefaultListableBeanFactory的实现：

```java
@Override
public void preInstantiateSingletons() {
    for (String beanName : beanDefinitionNames) {
        getBean(beanName);
    }
}
```

便利析出的所有beanDefinitionName，通过getBean方法获取Bean，getBean方法上面已经介绍过，不多赘述了。

## 3. 测试

```java
package com.zhuoli.service.min.spring.ioc.test;

import com.zhuoli.service.min.spring.ioc.bean.Company;
import com.zhuoli.service.min.spring.ioc.context.ApplicationContext;
import com.zhuoli.service.min.spring.ioc.context.ClasspathXmlApplicationContext;

public class CompanyApplicationContext {
    public static void main(String[] args) {
        ApplicationContext context = new ClasspathXmlApplicationContext("classpath:min-ioc.xml");
        Company company = (Company) context.getBean("company");
        company.open();
    }
}
```

运行结果：

```
Company Apple is open.
Employee zhuoli is working
```

通过自定义实现的IOC容器，我们成功获取了xml文件中配置的对象，说明自定义IOC容器是生效的。

## 4. 总结

上面介绍了很多类的概念，也许会有些乱，最后我们整理一下各个类的作用。

[![img](Spring源码解读『IOC容器1-自定义实现IOC容器』.assets/ClasspathXmlApplicationContext.png)](http://cdn.lidol.top/lidol_blog/ClasspathXmlApplicationContext.png)

分别来梳理一下自定义IOC容器涉及的几个类的作用：

**Resource：**

- Resource：接口，定义了获取字节流的能力，getInputStream
- ClassPathResource：实现了Resource接口，可以获取path指定文件的字节输入流
- ResourceLoader：获取Resource对象，是XmlBeanDefinitionReader类的成员变量，用于XmlBeanDefinitionReader类解析xml配置文件的基础工作——将xml文件为字节输入流

**BeanFactory：**

- BeanFactory：接口，定义了通过beanName获取bean对象的方法
- ConfigurableListableBeanFactory：继承了BeanFactory接口，并拓展了初始化所有单例bean的方法preInstantiateSingletons
- AbstractBeanFactory：抽象类，继承了ConfigurableListableBeanFactory，实现了BeanFactory接口中的getBean方法，该抽象类的作用是方便我们实现自定义的BeanFactory，通过继承AbstractBeanFactory抽象类，我们只用实现ConfigurableListableBeanFactory接口定义的preInstantiateSingletons方法即可
- BeanDefinitionRegistry：接口，定义了注册BeanDefinition的能力
- DefaultListableBeanFactory：继承了AbstractBeanFactory类，实现了BeanDefinitionRegistry接口，所以可以说DefaultListableBeanFactory既是Bean工厂也是Ban注册中心
- ApplicationContext：接口，继承了BeanFactory接口，ApplicationContext也是bean工厂，没有拓展BeanFactory的能力
- AbstractApplicationContext：抽象类，继承了ApplicationContext接口，并持有一个ConfigurableListableBeanFactory成员变量，实现了getBean方法，具有初始化Bean工厂（初始化所有的bean）和获取bean对象的能力
- ClasspathXmlApplicationContext：继承了AbstractApplicationContext，具备AbstractApplicationContext的一切能力。所以可以通过ClasspathXmlApplicationContext初始化Bean工厂，获取bean对象。

**BeanDefinitionReader：**

- BeanDefinitionReader：接口，定义了从指定位置加载BeanDefinition的方法
- XmlBeanDefinitionReader：实现了BeanDefinitionReader接口，持有ResourceLoader和BeanDefinitionRegistry两个成员变量，分别用于在加载BeanDefinition时从指定位置加载字节输入流、将BeanDefinition注册到注册中心。XmlBeanDefinitionReader的使用时机是上述ClasspathXmlApplicationContext类中实现AbstractApplicationContext抽象类定义的loadBeanDefinitions方法

```java
@Override
protected void loadBeanDefinitions(BeanDefinitionRegistry registry) throws Exception {
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry, new ResourceLoader());
    reader.loadBeanDefinitions(location);
}
```

上述就是我们自定义实现的一个简单地IOC容器，实现了IOC和DI的功能。其实Spring Framework也提供了IOC容器，基本思路与我们自定义实现的IOC容器一致，并且相关类的类名也是一致的（因为我们自定义实现的IOC容器其实是对Spring IOC容器的精简==）。只不过Spring实现的IOC容器提供了更多的功能呢，比如实现Bean的作用周期、延迟加载等功能。我们会在接下来的文章来介绍Spring IOC的实现。在理解了本文的基础上，我们再研究Spring IOC的实现就会比较简单。

> 参考链接：
>
> \1. Spring 源码
>
> \2. [构建简单IOC容器](https://my.oschina.net/u/2377110/blog/902073)
>
> \3. [实现一个基本的IoC容器](https://my.oschina.net/flashsword/blog/192551)