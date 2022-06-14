# [Spring源码解读『Spring Bean循环依赖』](http://lidol.top/frame/2574/)

2020-04-14 分类：[Spring](http://lidol.top/category/frame/spring/) / [框架](http://lidol.top/category/frame/) 阅读(472) 评论(0)

由于Spring的IOC特性，Bean都是由Spring容器生成的，那么如果Bean是单例的，存在两个Bean，分别为beanA、beanB，beanA依赖beanB，同时beanB也依赖beanA，那么可以想象假如容器不做特殊处理的话，就会发生循环依赖，产生死锁，Bean构造就进行不下去了。但是我们在使用时，其实并没有关注循环依赖的问题，Spring是可以解决这种循环依赖的情况的，本篇文章我们来看一下Spring是如何解决循环依赖的。

## 1. Spring循环依赖示例

首先定义两个Bean，BeanA和BeanB，两个Bean分别由一个对方类型的成员变量，如下：

```
public class BeanA {
    private BeanB beanB;

    public BeanB getBeanB() {
        return beanB;
    }

    public void setBeanB(BeanB beanB) {
        this.beanB = beanB;
    }
}
public class BeanB {
    private BeanA beanA;

    public BeanA getBeanA() {
        return beanA;
    }

    public void setBeanA(BeanA beanA) {
        this.beanA = beanA;
    }
}
```

spring xml配置文件，配置属性依赖：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
	http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="beanA" class="com.zhuoli.service.spring.explore.circular.dependence.BeanA">
        <property name="beanB" ref="beanB"/>
    </bean>

    <bean id="beanB" class="com.zhuoli.service.spring.explore.circular.dependence.BeanB">
        <property name="beanA" ref="beanA"/>
    </bean>

</beans>
```

测试代码：

```
public class CircularDependenceTest {
    public static void main(String[] args) {
        AbstractApplicationContext abstractApplicationContext = new ClassPathXmlApplicationContext("classpath:spring-circular-dependence.xml");
        BeanA beanA = (BeanA) abstractApplicationContext.getBean("beanA");
        BeanB beanB = (BeanB) abstractApplicationContext.getBean("beanB");
        System.out.println(beanA.getBeanB() == beanB);
        System.out.println(beanA == beanB.getBeanA());
    }
}
```

运行结果：

```
true
true
```

也就是讲，beanA拿到了beanB的引用，beanB同时也拿到了beanA的引用。虽然BeanA和BeanB之间存在循环依赖，但是Spring容器并没有发生死锁，成功解决了循环依赖问题，并构造了BeanA和BeanB对象。

## 2. Spring解决循环依赖

其实Spring是如何解决循环依赖问题，在上篇文章介绍Spring初始化流程的时候已经简单提过，我们这里突出再来理一下。同时要注意的是，**我们所说的Spring解决循环依赖，只限于单例Bean，对于非单例Bean，是不支持的**。

简单来讲，Spring解决循依赖，其实是通过提早缓存未实例结束的bean来实现的。首先在doGetBean()方法中，该方法获取bean，首先会尝试从缓存中获取，如下：

```
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
		@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

	final String beanName = transformedBeanName(name);
	Object bean;

	// 尝试从缓存中获取bean实例
	Object sharedInstance = getSingleton(beanName);
	if (sharedInstance != null && args == null) {
		//
	}

	else {
		//
	}

	// Check if required type matches the type of the actual bean instance.
	if (requiredType != null && !requiredType.isInstance(bean)) {
		//
	}
	return (T) bean;
}
public Object getSingleton(String beanName) {
	return getSingleton(beanName, true);
}

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	//1. 查询缓存中是否有创建好的单例
	Object singletonObject = this.singletonObjects.get(beanName);
	//2. 如果缓存不存在，判断是否正在创建中
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		//加锁防止并发
		synchronized (this.singletonObjects) {
			// 从earlySingletonObjects中查询是否有early缓存
			singletonObject = this.earlySingletonObjects.get(beanName);
			// early缓存也不存在，且允许early引用
			if (singletonObject == null && allowEarlyReference) {
				// 从单例工厂Map里查询beanName对应的ObjectFactory
				ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
				if (singletonFactory != null) {
					// 如果beanName对应的ObjectFactory存在，则调用getObject方法拿到单例对象
					singletonObject = singletonFactory.getObject();
					// 将单例对象添加到early缓存中
					this.earlySingletonObjects.put(beanName, singletonObject);
					// 移除单例工厂中对应的singletonFactory
					this.singletonFactories.remove(beanName);
				}
			}
		}
	}
	return singletonObject;
}
```

从以上的代码可以看出：

1. 缓存机制，只针对单例的bean
2. 默认的singletonObjects缓存不存在要get的beanName时，**判断beanName是否正在创建中**
3. 从early缓存earlySingletonObjects中再查询，**early缓存是用来缓存已实例化但未组装完成的bean**
4. 如果early缓存也不存在，从singletonFactories中查找是否有beanName对应的ObjectFactory对象工厂
5. 如果对象工厂存在，则调用getObject方法拿到bean对象
6. 将bean对象加入early缓存，并移除singletonFactories的对象工厂

上面最重要的就是singletonFactories何时放入了可以通过getObject获得bean对象的ObjectFactory。考虑到循环依赖的场景，应该会是bean对象实例化后，而属性注入之前。仔细寻找后发现，在AbstractAutowireCapableBeanFactory类的doCreateBean方法，执行完createBeanInstance实例化bean之后，populateBean属性注入之前，有这样一段代码：

```
// Eagerly cache singletons to be able to resolve circular references
// even when triggered by lifecycle interfaces like BeanFactoryAware.
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
		isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
	if (logger.isDebugEnabled()) {
		logger.debug("Eagerly caching bean '" + beanName +
				"' to allow for resolving potential circular references");
	}
	addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
	Assert.notNull(singletonFactory, "Singleton factory must not be null");
	synchronized (this.singletonObjects) {
		if (!this.singletonObjects.containsKey(beanName)) {
			this.singletonFactories.put(beanName, singletonFactory);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.add(beanName);
		}
	}
}
```

当判断bean为单例且正在创建中，而Spring允许循环引用时，将能获得bean对象的引用的ObjectFactory添加到singletonFactories中，此时就与之前的getSingleton方法相呼应。而allowCircularReferences标识在spring中默认为true，但是也可以通过setAllowCircularReferences方法对AbstractAutowireCapableBeanFactory进行设置。

再来看下getObject方法中的getEarlyBeanReference方法。这里也设置了一个InstantiationAwareBeanPostProcessor后置处理器的扩展点，允许在对象返回之前修改甚至替换bean。

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
	Object exposedObject = bean;
	if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
				SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
				exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
			}
		}
	}
	return exposedObject;
}
```

最后，来梳理一下上面abstractApplicationContext.getBean(“beanA”)的执行过程：

1. 实例化BeanA
2. 将能获取BeanA对象的ObjectFactory添加到singletonFactories中
3. BeanA注入BeanB属性，调用getBean(“beanB”)方法
4. 实例化BeanB
5. 将能获取BeanB对象的ObjectFactory添加到singletonFactories中
6. BeanB注入BeanA属性，调用getBean(“beanA”)
7. 从singletonFactories中获取ObjectFactory并调用getObject方法拿到beanA对象的引用
8. BeanB创建完成，注入到BeanA的beanB属性中
9. BeanA创建完成返回

上面我们了解了单例的bean循环引用的处理过程，那么多例的呢？其实我们可以按上面的思路来思考一下，单例bean的循环引用是因为每个对象都是固定的，只是提前暴露对象的引用，最终这个引用对应的对象是创建完成的。但是多例的情况下，每次getBean都会创建一个新的对象，那么应该引用哪一个对象呢，这本身就已经是矛盾的了。因而spring中对于多例之间相互引用是会提示错误的。在doGetBean protoType处理的逻辑中，第一步就存在下面的判断：

```
// Fail if we're already creating this bean instance:
// We're assumably within a circular reference.
if (isPrototypeCurrentlyInCreation(beanName)) {
	throw new BeanCurrentlyInCreationException(beanName);
}
```

> 参考链接：
>
> \1. Spring源码
>
> \2. [bean的循环依赖](https://my.oschina.net/u/2377110/blog/979226)