### SpringMVC-方法四种类型返回值

#### 1. ModelAndView

以前前后端不分的情况下，ModelAndView 应该是最最常见的返回值类型了，现在前后端分离后，后端都是以返回 JSON 数据为主了。后端返回 ModelAndView 这个比较容易理解，开发者可以在 ModelAndView 对象中指定视图名称，然后也可以绑定数据，像下面这样：

> ```java
> @RequestMapping("/book")
> public ModelAndView getAllBook() {
>     
>     //创建视图对象
>     ModelAndView mv = new ModelAndView();
>    
>     List<Book> books = new ArrayList<>();
>     
>     Book b1 = new Book();
>     b1.setId(1);
>     b1.setName("三国演义");
>     b1.setAuthor("罗贯中");
>     books.add(b1);
>     
>     Book b2 = new Book(); 
>     b2.setId(2);
>     b2.setName("红楼梦");
>     b2.setAuthor("曹雪芹");
>     books.add(b2);
>     
>     //指定数据模型
>     mv.addObject("bs", books);
>     
>     //指定视图名
>     mv.setViewName("book");
>     
>     return mv;
> }
> ```

#### 2. Void

返回值为 void 时，可能是你真的没有值要返回，也可能是你有其他办法，松哥将之归为如下四类，大伙来看下。

**2.1 没有值**

如果确实没有返回值，那就返回 void ，但是一定要注意，此时，方法上需要添加 @ResponseBody 注解，像下面这样：

> ```java
> @RequestMapping("/test2")
> @ResponseBody
> public void test2(){
>     //Todo...
> }
> ```

**2.2 重定向**

由于 SpringMVC 中的方法默认都具备 HttpServletResponse 参数，因此可以重拾 Servlet/Jsp 中的技能，可以实现重定向，像下面这样手动设置响应头：

> ```java
> @RequestMapping("/test1")
> @ResponseBody
> public void test1(HttpServletResponse resp){
>     resp.setStatus(302);
>     resp.addHeader("Location","/aa/index");
> }
> ```

也可以像下面这样直接调用重定向的方法：

> ```java
> @RequestMapping("/test1")
> @ResponseBody
> public void test1(HttpServletResponse resp){
>     resp.sendRedirect("/aa/index");
> }
> ```

当然，重定向无论你怎么写，都是 Servlet/Jsp 中的知识点，上面两种写法都相当于是重回远古时代。

**2.3 服务端跳转**

既然可以重定向，当然也可以服务端跳转，像下面这样：

> ```java
> @GetMapping("/test5")
> public void test5(HttpServletRequest req,HttpServletResponse resp) {
>     req.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(req,resp);
> }
> ```

**2.4 返回字符串**

当然也可以利用 HttpServletResponse 返回其他字符串数据，包括但不局限于 JSON，像下面这样：

> ```java
> @RequestMapping("/test2")
> @ResponseBody
> public void test2(HttpServletResponse resp) throws IOException {
>     
>     resp.setContentType("application/json;charset=utf-8");
>     PrintWriter out = resp.getWriter();
>     
>     List<Book> books = new ArrayList<>();
>     Book b1 = new Book();
>     b1.setId(1);
>     b1.setName("三国演义");
>     b1.setAuthor("罗贯中");
>     books.add(b1);
>     
>     Book b2 = new Book();
>     b2.setId(2);
>     b2.setName("红楼梦");
>     b2.setAuthor("曹雪芹");
>     books.add(b2);
>     
>     String s = new Gson().toJson(books);
>     
>     out.write(s);
>     out.flush();
>     out.close();
> }
> ```

这是返回值为 void 时候的情况，方法返回值为 void ，不一定就真的不返回了，可能还有其他的方式给前端数据。



#### 3. String

当 SpringMVC 方法的返回值为 String 类型时，也有几种不同情况。

**3.1 逻辑视图名**

返回 String 最常见的是逻辑视图名，这种时候一般利用默认的参数 Model 来传递数据，像下面这样 ：

> ```java
> @RequestMapping("/hello")
> public String aaa(Model model) {
>     model.addAttribute("username", "张三");
>     return "hello";
> }
> ```

此时返回的 `hello` 就是逻辑视图名，需要携带的数据放在 model 中。

**3.2 重定向**

也可以重定向，事实上，如果在 SpringMVC 中有重定向的需求，一般采用这种方式：

> ```java
> @RequestMapping("/test4")
> public String test4() {
>     return "redirect:/aa/index";
> }
> ```

**3.3 forward 转发**

也可以 forward 转发，事实上，如果在 SpringMVC 中有 forward 转发的需求，一般采用这种方式：

> ```java
> @RequestMapping("/test3")
> public String test3() {
>     return "forward:/WEB-INF/jsp/order.jsp";
> }
> ```

**3.4 真的是 String**

当然，也有一种情况，就是你真的想返回一个 String ，此时，只要在方法上加上 @ResponseBody 注解即可，或者 Controller 上本身添加的是组合注解 @RestController，像下面这样：

> ```java
> @RestController
> public class HelloController {
>     @GetMapping("/hello")
>     public String hello() {
>         return "hello provider!";
>     }
> }
> ```

也可以像下面这样 ：

> ```java
> @Controller
> public class HelloController {
>     @GetMapping("/hello")
>     @ResponseBody
>     public String hello() {
>         return "hello provider!";
>     }
> }
> ```

这是返回值为 String 的几种情况。

#### 4. JSON

返回 JSON 算是最最常见的了，现在前后端分离的趋势下，大部分后端只需要返回 JSON 即可，那么常见的 List 集合、Map，实体类等都可以返回，这些数据由 HttpMessageConverter 自动转为 JSON ，如果大家用了 Jackson 或者  Gson ，不需要额外配置就可以自动返回 JSON 了，因为框架帮我们提供了对应的 HttpMessageConverter ，如果大家使用了 Alibaba 的 Fastjson 的话，则需要自己手动提供一个相应的 HttpMessageConverter 的实例，方法的返回值像下面这样：

> ```java
> @GetMapping("/user")
> @ResponseBody
> public User getUser() {
>     User user = new User();
>     List<String> favorites = new ArrayList<>();
>     favorites.add("足球");
>     favorites.add("篮球");
>     user.setFavorites(favorites);
>     user.setUsername("zhagnsan");
>     user.setPassword("123");
>     return user;
> }
> 
> @GetMapping("/users")
> @ResponseBody
> public List<User> getALlUser() {
>     List<User> users = new ArrayList<>();
>     for (int i = 0; i < 10; i++) {
>         User e = new User();
>         e.setUsername("zhangsan:" + i);
>         e.setPassword("pwd:" + i);
>         users.add(e);
>     }
>     return users;
> }
> ```