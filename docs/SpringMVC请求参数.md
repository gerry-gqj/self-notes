# @PathVariable、@RequestParam、@RequestBody





## @RequestParam

注解@RequestParam接收的参数是来自HTTP请求体或请求url的QueryString中。

RequestParam可以接受简单类型的属性，也可以接受对象类型。

@RequestParam有三个配置参数：

- `required` 表示是否必须，默认为 `true`，必须。
- `defaultValue` 可设置请求参数的默认值。
- `value` 为接收url的参数名（相当于key值）。

**@RequestParam用来处理 `Content-Type` 为 `application/x-www-form-urlencoded` 编码的内容，`Content-Type`默认为该属性，
**也可以接收​​​​​​​application/json**。@RequestParam也可用于其它类型的请求，例如：POST、DELETE等请求**。

所以在postman中，要选择body的类型为 `x-www-form-urlencoded`，这样在headers中就自动变为了 `Content-Type` : `application/x-www-form-urlencoded` 编码格式。

但是如果传整个对象,这样会发现不支持批量插入数据，如果改用 `json` 字符串来传值的话，类型设置为 `application/json`，点击发送的话，会报错，后台接收不到值，为 `null`。

这种时候就可以采用@RequestBody

## @RequestBody

顾名思义,注解@RequestBody接收的参数是**来自requestBody**中，即**请求体**。一般用于处理非 `Content-Type: application/x-www-form-urlencoded`编码格式的数据，比如：`application/json`、`application/xml`等类型的数据。

就`application/json`类型的数据而言，使用注解@RequestBody可以将body里面所有的json数据传到后端，后端再进行解析。

GET请求中，因为没有HttpEntity，所以@RequestBody并不适用。

POST请求中，通过HttpEntity传递的参数，必须要在请求头中声明数据的类型Content-Type，SpringMVC通过使用

HandlerAdapter 配置的HttpMessageConverters来解析HttpEntity中的数据，然后绑定到相应的bean上。







# @RequestBody与@RequestParam



### 一、基础知识介绍

1️⃣`@RequestBody`主要用来接收前端传递给后端的`json`字符串中的数据(请求体中的数据)
2️⃣因为`GET`方式无请求体，所以使用`@RequestBody`接收数据时，前端不能使用`GET`方式提交数据，而是用`POST`方式进行提交。
3️⃣在后端的同一个接收方法里，`@RequestBody`与`@RequestParam`可以同时使用。
4️⃣一个请求，只能有一个`@RequestBody`，却可以有多个`@RequestParam`。
5️⃣当同时使用`@RequestParam`和`@RequestBody`时，`@RequestParam`指定的参数可以是普通元素、数组、集合、对象等等(即：当`@RequestBody`与`@RequestParam`可以同时使用时，原SpringMVC接收参数的机制不变，只不过`@RequestBody`接收的是请求体里面的数据；而`@RequestParam`接收的是key-value里面的参数，所以它会被切面进行处理从而可以用普通元素、数组、集合、对象等接收。)
6️⃣如果参数是放在请求体中，传入后台的话，那么后台要用`@RequestBody`才能接收到；如果不是放在请求体中的话，那么后台接收前台传过来的参数时，要用`@RequestParam`来接收，或形参前什么也不写也能接收。如果参数前写了`@RequestParam(xxx)`，那么前端必须有对应的xxx名字才行(不管其是否有值，当然可以通过设置该注解的required属性来控制是否必须传)，如果没有xxx名的话，那么请求会出错，报[400](https://www.jianshu.com/p/449e9d7db967)。

注：如果参数前不写`@RequestParam(xxx)`的话，那么就前端可有可无对应的xxx名字，如果有xxx名的话，那么就会自动匹配；没有的话，请求也能正确发送。

注意：这里与feign消费服务时不同：

- feign消费服务时，如果参数前什么也不写，那么会被默认是`@RequestBody`的。
- 如果后端参数是一个对象，且该参数前是以`@RequestBody`修饰的，那么前端传递`json`参数时，必须满足以下要求：后端`@RequestBody`注解对应的类在将`HTTP`的输入流(含请求体)装配到目标类(即：`@RequestBody`后面的类)时，会根据`json`字符串中的key来匹配对应实体类的属性，如果匹配一致且`json`中的该key对应的值符合(或可转换为)实体类的对应属性的类型要求时，会调用实体类的setter方法将值赋给该属性。

`json`字符串中，如果value为""的话，后端对应属性如果是String类型的，那么接受到的就是""，如果是后端属性的类型是Integer、Double等类型，那么接收到的就是null。`json`字符串中，如果value为null的话，后端对应收到的就是null。如果某个参数没有value的话，在传`json`字符串给后端时，要么干脆就不把该字段写到`json`字符串中；要么写value时， 必须有值，null 或""都行。

### 二、`@RequestBody`与前端传过来的`json`数据的匹配规则

声明：根据不同的`Content-Type`等情况，`Spring-MVC`会采取不同的`HttpMessageConverter`实现来进行信息转换解析。下面介绍的是最常用的：前端以`Content-Type`为`application/json`传递`json`字符串数据，后端以`@RequestBody`模型接收数据的情况。

解析json数据大体流程概述：Http传递请求体信息，最终会被封装进`com.fasterxml.jackson.core.json.UTF8StreamJsonParser`中(提示：Spring采用`CharacterEncodingFilter`设置了默认编码为UTF-8)，然后在`public class BeanDeserializer extends BeanDeserializerBase implements java.io.Serializable`中，通过 `public Object deserializeFromObject(JsonParser p, DeserializationContext ctxt) throws IOException`方法进行解析。

### 三、结论

1. `@JsonAlias`注解。实现`json`转模型时，使`json`中的特定key能转化为特定的模型属性；但是模型转`json`时，对应的转换后的key仍然与属性名一致，
2. `@JsonProperty`注解。实现`json`转模型时，使`json`中的特定key能转化为指定的模型属性；同样的，模型转`json`时，对应的转换后的key为指定的key。
3. `@JsonAlias`注解需要依赖于setter、getter，而`@JsonProperty`注解不需要。
4. 在不考虑上述两个注解的一般情况下，key与属性匹配时，默认大小写敏感。
5. 有多个相同的key的`json`字符串中，转换为模型时，会以相同的几个key中，排在最后的那个key的值给模型属性复制，因为setter会覆盖原来的值。
6. 后端`@RequestBody`注解对应的类在将`HTTP`的输入流(含请求体)装配到目标类(即：`@RequestBody`后面的类)时，会根据`json`字符串中的key来匹配对应实体类的属性，如果匹配一致且`json`中的该key对应的值符合(或可转换为)实体类的对应属性的类型要求时，会调用实体类的setter方法将值赋给该属性。

### 四、在使用Json传值并且使用@RequestBody注解的时候需要注意的问题

1️⃣一个方法中只能有一个@RequestBody注解。
因为RequestBody就是request的inputStream，这个流在第一次使用该注解后会关闭，后面的都会报错（stream closed）。

2️⃣默认情况下@RequestBody标注的对象必须包含前台传来的所有字段。如果没有包含前台传来的字段，就会报错：Unrecognized field xxx , not marked as ignorable，出现这种问题是因为使用jackson进行json转换时，MappingJacksonHttpMessageConverter默认要求必须存在相应的字段。如果没有前台传来的某个字段或者字段没有提供set方法，就会报错。解决方法：

1. 可以增加一个字段来接收前台传来的这个值，如果存在多个字段，这种方式很不好（就算一个字段，如果没用，新增字段也不好）。
2. 在前台往后台传值的时候，去掉无用的字段。这样还能减少网络传输的大小。
3. 使用Jackson提供的json注解：

- `@JsonIgnore`注解用来忽略某些字段，可以用在Field或者Getter方法上，用在Setter方法时，和Filed效果一样。这个注解只能用在POJO存在的字段要忽略的情况，不能满足现在需要的情况。
- `@JsonIgnoreProperties(ignoreUnknown = true)`，将这个注解写在类上之后，就会忽略类中不存在的字段，可以满足当前的需要。这个注解还可以指定要忽略的字段。如： `@JsonIgnoreProperties({ "internalId", "secretKey"})`指定的字段不会被序列化和反序列化。

