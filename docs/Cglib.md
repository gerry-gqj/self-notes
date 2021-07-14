### Cglib动态代理

**示例**

> ```java
> public class SampleClass {
> 
>     public String test(String input){
>         return "hello cglib001...";
>     }
> 
>     public static void main(String [] args){
>         Enhancer enhancer = new Enhancer();
>         enhancer.setSuperclass(SampleClass.class);
> /*        enhancer.setCallback(new MethodInterceptor() {
>             @Override
>             public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy)
>                     throws Throwable {
>                 System.out.println("before method run...");
>                 Object o1 = methodProxy.invokeSuper(o, objects);
>                 System.out.println("after method run...");
> 
>                 return o1;
>             }
>         });*/
> 
>         //Lambda表达式
>         enhancer.setCallback((MethodInterceptor)(Object o, Method method, 
>                                                  Object[] objects, MethodProxy methodProxy)->{
>             System.out.println("before method run...");
>             Object o1 = methodProxy.invokeSuper(o, objects);
>             System.out.println("after method run...");
>             return o1;
>         });
>         SampleClass sampleClass = (SampleClass) enhancer.create();
>         sampleClass.test("test....");
>     }
> }
> 
> ```



#### 1、Enhancer类

Enhancer可能是CGLIB中最常用的一个类，和Java1.3动态代理中引入的Proxy类差不多。和Proxy不同的是，**Enhancer既能够代理普通的class，也能够代理口。**Enhancer创建一个被代理对象的子类并且**拦截所有的方法调用（包括从Object中继承的toString和hashCode方法）**。**Enhancer不能够拦截final方法**，例如Object.getClass()方法，这是由于Java final方法语义决定的。基于同样的道理，Enhancer也不能对fianl类进行代理操作。这也是Hibernate为什么不能持久化final class的原因。

> ```java
> public class SampleClass {
>     public String test(String input){
>         return "hello world";
>     }
> }
> ```

> ```java
>     @Test
>     public void test01(){
>         Enhancer enhancer = new Enhancer();
> 
>         enhancer.setSuperclass(SampleClass.class);
> 
>         enhancer.setCallback(new FixedValue() {
>             @Override
>             public Object loadObject() throws Exception {
>                 return "Hello cglib002...";
>             }
>         });
> 
>         SampleClass sampleClass = (SampleClass)enhancer.create();
> 
>         System.out.println(sampleClass.test(null));
> 
>         System.out.println(sampleClass.getClass());
> 
>         System.out.println(sampleClass.hashCode());
> 
>     }
> ```

输出

> ```java
> Hello cglib002...
> class com.atcglib.SampleClass$$EnhancerByCGLIB$$845377c6
> 
> java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Number
> 
> 	at com.atcglib.SampleClass$$EnhancerByCGLIB$$845377c6.hashCode(<generated>)
> 	at com.atcglib.MyTest.test01(MyTest.java:29)
> 	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
> 	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
> 	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
> 	at java.lang.reflect.Method.invoke(Method.java:498)
> 	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:59)
> 	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
> 	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:56)
> 	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
> 	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306)
> 	at org.junit.runners.BlockJUnit4ClassRunner$1.evaluate(BlockJUnit4ClassRunner.java:100)
> 	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:366)
> 	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:103)
> 	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:63)
> 	at org.junit.runners.ParentRunner$4.run(ParentRunner.java:331)
> 	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:79)
> 	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:329)
> 	at org.junit.runners.ParentRunner.access$100(ParentRunner.java:66)
> 	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:293)
> 	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306)
> 	at org.junit.runners.ParentRunner.run(ParentRunner.java:413)
> 	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
> 	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
> 	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
> 	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
> 	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
> ```

上述代码中，FixedValue用来对所有拦截的方法返回相同的值，从输出我们可以看出来，Enhancer对非final方法test()、toString()、hashCode()进行了拦截，没有对getClass进行拦截。由于hashCode()方法需要返回一个Number，但是我们返回的是一个String，这解释了上面的程序中为什么会抛出异常。

Enhancer.setSuperclass用来设置父类型，从toString方法可以看出，使用CGLIB生成的类为被代理类的一个子类，形如：SampleClass\<script type="math/tex" id="MathJax-Element-1">\</script>EnhancerByCGLIB\<script type="math/tex" id="MathJax-Element-2">\</script>e3ea9b7

Enhancer.create(Object…)方法是用来创建增强对象的，其提供了很多不同参数的方法用来匹配被增强类的不同构造方法。**（虽然类的构造放法只是Java字节码层面的函数，但是Enhancer却不能对其进行操作。Enhancer同样不能操作static或者final类）。我们也可以先使用Enhancer.createClass()来创建字节码(.class)，然后用字节码动态的生成增强后的对象。**

**可以使用一个InvocationHandler作为回调，使用invoke方法来替换直接访问类的方法，但是你必须注意死循环。**因为invoke中调用的任何原代理类方法，均会重新代理到invoke方法中。

```java
public void testInvocationHandler() throws Exception{
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(SampleClass.class);
    enhancer.setCallback(new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            if(method.getDeclaringClass() != Object.class && method.getReturnType() == String.class){
                return "hello cglib";
            }else{
                throw new RuntimeException("Do not know what to do");
            }
        }
    });
    SampleClass proxy = (SampleClass) enhancer.create();
    Assert.assertEquals("hello cglib", proxy.test(null));
    Assert.assertNotEquals("Hello cglib", proxy.toString());
}
```

为了避免这种死循环，我们可以使用MethodInterceptor，MethodInterceptor的例子在前面的hello world中已经介绍过了，这里就不浪费时间了。



#### 2、CallbackFilter

有些时候我们可能只想对特定的方法进行拦截，对其他的方法直接放行，不做任何操作，使用Enhancer处理这种需求同样很简单,只需要一个CallbackFilter即可：

```java
@Test
public void testCallbackFilter() throws Exception{
    Enhancer enhancer = new Enhancer();
    CallbackHelper callbackHelper = new CallbackHelper(SampleClass.class, new Class[0]) {
        @Override
        protected Object getCallback(Method method) {
            if(method.getDeclaringClass() != Object.class && method.getReturnType() == String.class){
                return new FixedValue() {
                    @Override
                    public Object loadObject() throws Exception {
                        return "Hello cglib";
                    }
                };
            }else{
                return NoOp.INSTANCE;
            }
        }
    };
    enhancer.setSuperclass(SampleClass.class);
    enhancer.setCallbackFilter(callbackHelper);
    enhancer.setCallbacks(callbackHelper.getCallbacks());
    SampleClass proxy = (SampleClass) enhancer.create();
    Assert.assertEquals("Hello cglib", proxy.test(null));
    Assert.assertNotEquals("Hello cglib",proxy.toString());
    System.out.println(proxy.hashCode());
}
```



#### 3、ImmutableBean

通过名字就可以知道，不可变的Bean。ImmutableBean允许创建一个原来对象的包装类，这个包装类是不可变的，**任何改变底层对象的包装类操作都会抛出IllegalStateException**。但是我们可以通过直接操作底层对象来改变包装类对象。这有点类似于Guava中的不可变视图

为了对ImmutableBean进行测试，这里需要再引入一个bean

```java
public class SampleBean {
    private String value;

    public SampleBean() {
    }

    public SampleBean(String value) {
        this.value = value;
    }
    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

然后编写测试类如下：

```java
@Test(expected = IllegalStateException.class)
public void testImmutableBean() throws Exception{
    SampleBean bean = new SampleBean();
    bean.setValue("Hello world");
    SampleBean immutableBean = (SampleBean) ImmutableBean.create(bean); //创建不可变类
    Assert.assertEquals("Hello world",immutableBean.getValue()); 
    
    //可以通过底层对象来进行修改
    bean.setValue("Hello world, again"); 
    
    Assert.assertEquals("Hello world, again", immutableBean.getValue());
    
    //直接修改将throw exception
    immutableBean.setValue("Hello cglib"); 
}
```



#### 4、Bean generator

cglib提供的一个操作bean的工具，使用它能够在运行时动态的创建一个bean。

```java
@Test
public void testBeanGenerator() throws Exception{
    
    BeanGenerator beanGenerator = new BeanGenerator();
    beanGenerator.addProperty("value",String.class);
    
    Object myBean = beanGenerator.create();
    Method setter = myBean.getClass().getMethod("setValue",String.class);
    setter.invoke(myBean,"Hello cglib");
    Method getter = myBean.getClass().getMethod("getValue");
    
    Assert.assertEquals("Hello cglib",getter.invoke(myBean));
}
```

在上面的代码中，我们使用cglib动态的**创建了一个和SampleBean相同的Bean对象，包含一个属性value以及getter、setter方法**



#### 5、Bean Copier

cglib提供的能够从一个bean复制到另一个bean中，而且其还提供了一个转换器，用来在转换的时候对bean的属性进行操作。

```java
public class OtherSampleBean {
    private String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}

@Test
public void testBeanCopier() throws Exception{
    
    //设置为true，则使用converter
    BeanCopier copier = BeanCopier.create(SampleBean.class, OtherSampleBean.class, false);
    SampleBean myBean = new SampleBean();
    myBean.setValue("Hello cglib");
    OtherSampleBean otherBean = new OtherSampleBean();
    
    //设置为true，则传入converter指明怎么进行转换
    copier.copy(myBean, otherBean, null); 
    
   assertEquals("Hello cglib", otherBean.getValue());
}
```



#### 6、BulkBean

相比于BeanCopier，BulkBean将copy的动作拆分为getPropertyValues和setPropertyValues两个方法，允许自定义处理属性

```java
@Test
public void testBulkBean() throws Exception{
    BulkBean bulkBean = BulkBean.create(SampleBean.class,
            new String[]{"getValue"},
            new String[]{"setValue"},
            new Class[]{String.class});
    SampleBean bean = new SampleBean();
    bean.setValue("Hello world");
    Object[] propertyValues = bulkBean.getPropertyValues(bean);
    assertEquals(1, bulkBean.getPropertyValues(bean).length);
    assertEquals("Hello world", bulkBean.getPropertyValues(bean)[0]);
    bulkBean.setPropertyValues(bean,new Object[]{"Hello cglib"});
    assertEquals("Hello cglib", bean.getValue());
}
```

使用注意：

1. 避免每次进行BulkBean.create创建对象，一般将其声明为static的
2. 应用场景：针对特定属性的get,set操作，一般适用通过xml配置注入和注出的属性，运行时才确定处理的Source,Target类，只需要关注属性名即可。



#### 7、BeanMap

BeanMap类实现了Java Map，将一个bean对象中的所有属性转换为一个String-to-Obejct的Java Map

```java
@Test
public void testBeanMap() throws Exception{
    
    BeanGenerator generator = new BeanGenerator();
    generator.addProperty("username",String.class);
    generator.addProperty("password",String.class);
    
    Object bean = generator.create();
    Method setUserName = bean.getClass().getMethod("setUsername", String.class);
    Method setPassword = bean.getClass().getMethod("setPassword", String.class);
    
    setUserName.invoke(bean, "admin");
    setPassword.invoke(bean,"password");
    
    BeanMap map = BeanMap.create(bean);
    
    Assert.assertEquals("admin", map.get("username"));
    Assert.assertEquals("password", map.get("password"));
}
```

我们使用BeanGenerator生成了一个含有两个属性的Java Bean，对其进行赋值操作后，生成了一个BeanMap对象，通过获取值来进行验证



#### 8、keyFactory

keyFactory类用来动态生成接口的实例，接口需要只包含一个newInstance方法，返回一个Object。keyFactory为构造出来的实例动态生成了Object.equals和Object.hashCode方法，能够确保相同的参数构造出的实例为单例的。

```java
public interface SampleKeyFactory {
    Object newInstance(String first, int second);
}
```

我们首先构造一个满足条件的接口，然后进行测试

```java
@Test
public void testKeyFactory() throws Exception{
    SampleKeyFactory keyFactory = (SampleKeyFactory) KeyFactory.create(SampleKeyFactory.class);
    Object key = keyFactory.newInstance("foo", 42);
    Object key1 = keyFactory.newInstance("foo", 42);
    
    //测试参数相同，结果是否相等
    Assert.assertEquals(key,key1);
}
```



#### 9、Mixin(混合)

**Mixin能够让我们将多个对象整合到一个对象中去，前提是这些对象必须是接口的实现。**可能这样说比较晦涩，以代码为例：

```java
public class MixinInterfaceTest {
    interface Interface1{
        String first();
    }
    interface Interface2{
        String second();
    }

    class Class1 implements Interface1{
        @Override
        public String first() {
            return "first";
        }
    }

    class Class2 implements Interface2{
        @Override
        public String second() {
            return "second";
        }
    }

    interface MixinInterface extends Interface1, Interface2{

    }

    @Test
    public void testMixin() throws Exception{
        Mixin mixin = Mixin.create(new Class[]{Interface1.class, Interface2.class,
                        MixinInterface.class}, new Object[]{new Class1(),new Class2()});
        MixinInterface mixinDelegate = (MixinInterface) mixin;
        assertEquals("first", mixinDelegate.first());
        assertEquals("second", mixinDelegate.second());
    }
}
```

Mixin类比较尴尬，因为他要求Minix的类（例如MixinInterface）实现一些接口。既然被Minix的类已经实现了相应的接口，那么我就直接可以通过纯Java的方式实现，没有必要使用Minix类。



#### 10、String switcher

用来模拟一个String到int类型的Map类型。如果在Java7以后的版本中，类似一个switch语句。

```java
@Test
public void testStringSwitcher() throws Exception{
    String[] strings = new String[]{"one", "two"};
    int[] values = new int[]{10,20};
    
    StringSwitcher stringSwitcher = StringSwitcher.create(strings,values,true);
    
    assertEquals(10, stringSwitcher.intValue("one"));
    assertEquals(20, stringSwitcher.intValue("two"));
    assertEquals(-1, stringSwitcher.intValue("three"));
}
```



#### 11、Interface Maker

正如名字所言，Interface Maker用来创建一个新的Interface

```java
@Test
public void testInterfaceMarker() throws Exception{
    Signature signature = new Signature("foo", Type.DOUBLE_TYPE, new Type[]{Type.INT_TYPE});
    InterfaceMaker interfaceMaker = new InterfaceMaker();
    interfaceMaker.add(signature, new Type[0]);
    Class iface = interfaceMaker.create();
    assertEquals(1, iface.getMethods().length);
    assertEquals("foo", iface.getMethods()[0].getName());
    assertEquals(double.class, iface.getMethods()[0].getReturnType());
}
```

上述的Interface Maker创建的接口中只含有一个方法，签名为double foo(int)。Interface Maker与上面介绍的其他类不同，它依赖ASM中的Type类型。由于接口仅仅只用做在编译时期进行类型检查，因此在一个运行的应用中动态的创建接口没有什么作用。但是InterfaceMaker可以用来自动生成代码，为以后的开发做准备。



#### 12、Method delegate

MethodDelegate主要用来对方法进行代理

```java
interface BeanDelegate{
    String getValueFromDelegate();
}

@Test
public void testMethodDelegate()  throws Exception{
    SampleBean bean = new SampleBean();
    bean.setValue("Hello cglib");
    BeanDelegate delegate = (BeanDelegate) MethodDelegate.create(bean,"getValue", BeanDelegate.class);
    assertEquals("Hello cglib", delegate.getValueFromDelegate());
}
```

关于Method.create的参数说明：

1. 第二个参数为即将被代理的方法
2. 第一个参数必须是一个无参数构造的bean。因此MethodDelegate.create并不是你想象的那么有用
3. 第三个参数为只含有一个方法的接口。当这个接口中的方法被调用的时候，将会调用第一个参数所指向bean的第二个参数方法

缺点：

1. 为每一个代理类创建了一个新的类，这样可能会占用大量的永久代堆内存
2. 你不能代理需要参数的方法
3. 如果你定义的接口中的方法需要参数，那么代理将不会工作，并且也不会抛出异常；如果你的接口中方法需要其他的返回类型，那么将抛出IllegalArgumentException



#### 13、MulticastDelegate

1. 多重代理和方法代理差不多，都是将代理类方法的调用委托给被代理类。使用前提是需要一个接口，以及一个类实现了该接口
2. 通过这种interface的继承关系，我们能够将接口上方法的调用分散给各个实现类上面去。
3. 多重代理的缺点是接口只能含有一个方法，如果被代理的方法拥有返回值，那么调用代理类的返回值为最后一个添加的被代理类的方法返回值

```java
public interface DelegatationProvider {
    void setValue(String value);
}

public class SimpleMulticastBean implements DelegatationProvider {
    private String value;
    @Override
    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}

@Test
public void testMulticastDelegate() throws Exception{
    MulticastDelegate multicastDelegate = MulticastDelegate.create(DelegatationProvider.class);
    SimpleMulticastBean first = new SimpleMulticastBean();
    SimpleMulticastBean second = new SimpleMulticastBean();
    multicastDelegate = multicastDelegate.add(first);
    multicastDelegate  = multicastDelegate.add(second);

    DelegatationProvider provider = (DelegatationProvider) multicastDelegate;
    provider.setValue("Hello world");

    assertEquals("Hello world", first.getValue());
    assertEquals("Hello world", second.getValue());
}
```



#### 14、Constructor delegate

为了对构造函数进行代理，我们需要一个接口，这个接口只含有一个Object newInstance(…)方法，用来调用相应的构造函数

```java
interface SampleBeanConstructorDelegate{
    Object newInstance(String value);
}

/**
 * 对构造函数进行代理
 * @throws Exception
 */
@Test
public void testConstructorDelegate() throws Exception{
    SampleBeanConstructorDelegate constructorDelegate = (SampleBeanConstructorDelegate) ConstructorDelegate.create(
            SampleBean.class, SampleBeanConstructorDelegate.class);
    SampleBean bean = (SampleBean) constructorDelegate.newInstance("Hello world");
    assertTrue(SampleBean.class.isAssignableFrom(bean.getClass()));
    System.out.println(bean.getValue());
}
```



#### 15、Parallel Sorter(并行排序器)

能够对多个数组同时进行排序，目前实现的算法有归并排序和快速排序

```java
@Test
public void testParallelSorter() throws Exception{
    Integer[][] value = {
            {4, 3, 9, 0},
            {2, 1, 6, 0}
    };
    ParallelSorter.create(value).mergeSort(0);
    for(Integer[] row : value){
        int former = -1;
        for(int val : row){
            assertTrue(former < val);
            former = val;
        }
    }
}
```



#### 16、FastClass

顾明思义，FastClass就是对Class对象进行特定的处理，比如通过数组保存method引用，因此FastClass引出了一个index下标的新概念，比如getIndex(String name, Class[] parameterTypes)就是以前的获取method的方法。通过数组存储method,constructor等class信息，从而将原先的反射调用，转化为class.index的直接调用，从而体现所谓的FastClass。

```java
@Test
public void testFastClass() throws Exception{
    FastClass fastClass = FastClass.create(SampleBean.class);
    FastMethod fastMethod = fastClass.getMethod("getValue",new Class[0]);
    SampleBean bean = new SampleBean();
    bean.setValue("Hello world");
    assertEquals("Hello world",fastMethod.invoke(bean, new Object[0]));
}
```



**注意**

由于CGLIB的大部分类是直接对Java字节码进行操作，这样生成的类会在Java的永久堆中。如果动态代理操作过多，容易造成永久堆满，触发OutOfMemory异常。



### CGLIB和Java动态代理的区别

1. Java动态代理只能够对接口进行代理，不能对普通的类进行代理（因为所有生成的代理类的父类为Proxy，Java类继承机制不允许多重继承）；CGLIB能够代理普通类；
2. Java动态代理使用Java原生的反射API进行操作，在生成类上比较高效；CGLIB使用ASM框架直接对字节码进行操作，在类的执行过程中比较高效





### **使用Cglib代码对类做代理**

下面演示一下Cglib代码示例----对类做代理。首先定义一个Dao类，里面有一个select()方法和一个update()方法：

```java
public class Dao {
    
    public void update() {
        System.out.println("PeopleDao.update()");
    }
    
    public void select() {
        System.out.println("PeopleDao.select()");
    }
}
```

创建一个Dao代理，实现MethodInterceptor接口，目标是在update()方法与select()方法调用前后输出两句话：

```java
public class DaoProxy implements MethodInterceptor {

    @Override
    public Object intercept(Object object, Method method, Object[] objects, MethodProxy proxy) throws Throwable {
        System.out.println("Before Method Invoke");
        proxy.invokeSuper(object, objects);
        System.out.println("After Method Invoke");
        
        return object;
    }
    
}
```

intercept方法的参数名并不是原生的参数名，我做了自己的调整，几个参数的含义为：

- Object表示要进行增强的对象
- Method表示拦截的方法
- Object[]数组表示参数列表，基本数据类型需要传入其包装类型，如int-->Integer、long-Long、double-->Double
- MethodProxy表示对方法的代理，invokeSuper方法表示对被代理对象方法的调用

写一个测试类：



```java
public class CglibTest {

    @Test
    public void testCglib() {
        DaoProxy daoProxy = new DaoProxy();
        
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Dao.class);
        enhancer.setCallback(daoProxy);
        
        Dao dao = (Dao)enhancer.create();
        dao.update();
        dao.select();
    }
    
}
```

这是使用Cglib的通用写法，setSuperclass表示设置要代理的类，setCallback表示设置回调即MethodInterceptor的实现类，使用create()方法生成一个代理对象，注意要强转一下，因为返回的是Object。最后看一下运行结果：

```java
Before Method Invoke
PeopleDao.update()
After Method Invoke
Before Method Invoke
PeopleDao.select()
After Method Invoke
```

符合我们的期望。

 

**使用Cglib定义不同的拦截策略**

再扩展一点点，比方说在AOP中我们经常碰到的一种复杂场景是：**我们想对类A的B方法使用一种拦截策略、类A的C方法使用另外一种拦截策略**。

在本例中，即我们想对select()方法与update()方法使用不同的拦截策略，那么我们先定义一个新的Proxy：

```java
public class DaoAnotherProxy implements MethodInterceptor {

    @Override
    public Object intercept(Object object, Method method, Object[] objects, MethodProxy proxy) throws Throwable {
        
        System.out.println("StartTime=[" + System.currentTimeMillis() + "]");
        method.invoke(object, objects);
        System.out.println("EndTime=[" + System.currentTimeMillis() + "]");
        return object;
    }
    
}
```

方法调用前后输出一下开始时间与结束时间。为了实现我们的需求，实现一下CallbackFilter：

```java
public class DaoFilter implements CallbackFilter {

    @Override
    public int accept(Method method) {
        if ("select".equals(method.getName())) {
            return 0;
        }
        return 1;
    }
    
}
```

返回的数值表示顺序，结合下面的代码解释，测试代码要修改一下：

```java
public class CglibTest {

    @Test
    public void testCglib() {
        DaoProxy daoProxy = new DaoProxy();
        DaoAnotherProxy daoAnotherProxy = new DaoAnotherProxy();
        
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Dao.class);
        enhancer.setCallbacks(new Callback[]{daoProxy, daoAnotherProxy, NoOp.INSTANCE});
        enhancer.setCallbackFilter(new DaoFilter());
        
        Dao dao = (Dao)enhancer.create();
        dao.update();
        dao.select();
    }
    
}
```

意思是CallbackFilter的accept方法返回的数值表示的是顺序，顺序和setCallbacks里面Proxy的顺序是一致的。再解释清楚一点，Callback数组中有三个callback，那么：

- 方法名为"select"的方法返回的顺序为0，即使用Callback数组中的0位callback，即DaoProxy
- 方法名不为"select"的方法返回的顺序为1，即使用Callback数组中的1位callback，即DaoAnotherProxy

因此，方法的执行结果为：

```java
StartTime=[1491198489261]
PeopleDao.update()
EndTime=[1491198489275]
Before Method Invoke
PeopleDao.select()
After Method Invoke
```

符合我们的预期，因为update()方法不是方法名为"select"的方法，因此返回1，返回1使用DaoAnotherProxy，即打印时间；select()方法是方法名为"select"的方法，因此返回0，返回0使用DaoProxy，即方法调用前后输出两句话。

这里要额外提一下，Callback数组中我特意定义了一个NoOp.INSTANCE，这表示一个空Callback，即如果不想对某个方法进行拦截，可以在DaoFilter中返回2，具体效果可以自己尝试一下。

 

**构造函数不拦截方法**

如果Update()方法与select()方法在构造函数中被调用，那么也是会对这两个方法进行相应的拦截的，现在我想要的是构造函数中调用的方法不会被拦截，那么应该如何做？先改一下Dao代码，加一个构造方法Dao()，调用一下update()方法：

```java
public class Dao {
    
    public Dao() {
        update();
    }
    
    public void update() {
        System.out.println("PeopleDao.update()");
    }
    
    public void select() {
        System.out.println("PeopleDao.select()");
    }
}
```

如果想要在构造函数中调用update()方法时，不拦截的话，Enhancer中有一个setInterceptDuringConstruction(boolean interceptDuringConstruction)方法设置为false即可，默认为true，即构造函数中调用方法也是会拦截的。那么测试方法这么写：

```java
public class CglibTest {

    @Test
    public void testCglib() {
        DaoProxy daoProxy = new DaoProxy();
        DaoAnotherProxy daoAnotherProxy = new DaoAnotherProxy();
        
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Dao.class);
        enhancer.setCallbacks(new Callback[]{daoProxy, daoAnotherProxy, NoOp.INSTANCE});
        enhancer.setCallbackFilter(new DaoFilter());
        enhancer.setInterceptDuringConstruction(false);
        
        Dao dao = (Dao)enhancer.create();
        dao.update();
        dao.select();
    }
    
}
```

运行结果为：

```
PeopleDao.update()
StartTime=[1491202022297]
PeopleDao.update()
EndTime=[1491202022311]
Before Method Invoke
PeopleDao.select()
After Method Invoke
```

