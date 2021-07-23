### Cookie

服务端给客户端的数据，**存储于客户端**（浏览器）。由于是保存在客户端上的，所以存在安全问题，并且cookie是有个数和大小限制的（4KB），所以一般cookie用来存储一些比较小且安全性要求不高的数据，而且一般数据都会进行加密。

**Cookie操作方法**

#### 新建一个Cookie

> ```java
> //1、创建一个Cookie对象
> Cookie cookie = new Cookie(name,value);
> 
> cookie.setPath("/");
> cookie.setMaxAge(maxAge);
> 
> //2、通知客户端保存Cookie
> //通过响应头set-cookie通知客户保存Cookie
> //一次性可以创建cookie
> response.addCookie(cookie);
> ```

#### Cookie 获取

> ```java
> //注意:服务端获取到的Cookie放回值是一个数组
> //要获取具体的Cookie信息要自行封装
> Cookie[] cookies = request.getCookies();
> ```

> **Cookie封装类CookieUtil**
>
> ```java
> public class CookieUtil {
> 
>     public static void set(HttpServletResponse response, String name , String value, int maxAge) {
> 
>         Cookie cookie = new Cookie(name,value);
>         cookie.setPath("/");
>         cookie.setMaxAge(maxAge);
>         response.addCookie(cookie);
>     }
> 
>     public static Cookie get(HttpServletRequest request, String name) {
>         Map<String, Cookie> cookieMap = readCookieMap(request);
>         if (cookieMap.containsKey(name)) {
>             return cookieMap.get(name);
>         } else {
>             return null;
>         }
>     }
> 
>     public static Map<String, Cookie> readCookieMap(HttpServletRequest request) {
> 
>         Map<String, Cookie> cookieMap = new HashMap<>();
>         Cookie[] cookies = request.getCookies();
>         if (cookies != null) {
>             for (Cookie cookie : cookies) {
>                 cookieMap.put(cookie.getName(),cookie);
>             }
>         }
>         return cookieMap;
>     }
> 
> }
> ```



#### 更新Cookie

> 方案一、更新Cookie只需要新建一个相同的Cookie对象并覆盖原来的Cookie对象即可
>
> 方案二、先找到原来的Cookie的值使用setValue()方法进行修改



**Cookie的生命控制**

> ```java
> // 正数 Cookie的存在秒树
> // 负数 Cookie在浏览器关闭后销毁
> // 0   Cookie马上销毁
> cookie.setMaxAge()
> ```



#### Cookie Path属性

Cookie的path属性可以有效过滤哪些Cookie可以发送给服务器哪些可以不发，path属性是通过请求地址进行有效的过滤。

> ```java
> cookie.setPath(request.getContextPath+"/abc");
> //request.getContextPath是工程路径 在工程路径后追加/abc
> ```

> 假设有两Cookie
>
> **CookieA path/工程路径** 
>
> **CookieB path/工程路径/abc**  
>
> 
>
> **`http:ip:port/工程路径/a.html `会发送`CookieA path/工程路径 `这个Cookie**
>
> **`http:ip:port/工程路径/abc/a.html`会发送`CookieB path/工程路径/abc  `这个Cookie**



### Session

1、session是一个接口(HttpSession)。

2、session就是会话。他是用来维护一个客户端和服务器之间关联的一种技术。

3、每个客户端都有自己的一个Session会话。

4、Session会话中，我们经常用来保存用户登陆之后的信息。



#### Session的创建和获取

`request.getSession()`

**第一次调用是创建Session的会话**

**之后调用是获取前面创建好的Session会话对象**

`session.idNew()`判断是不是刚刚创建出来的(新的)

​	true 表示刚刚创建

​	false 表示获取之前的创建



每个会话都有一个sessionid。这个ID是唯一的。

`session.getId()`获取sessionid



#### Session域中数据

> ```java
> //往session中设置属性的值
> request.getSession.setAttribute（"key1","value"）;
> 
> //获取session属性中的值
> request.getSession.getAttribute("key1")
> ```



#### Session生命周期

> ```java
> //设置session的超时时间，超过指定时间的时长，session就会被销毁
> //以秒为单位
> setMaxInactiveInterval(int interval) 
> 
> //获取session超时时间
> getMaxInactiveInval()
>     
> //session马上超时无效 
> invalidate()
> ```



### Cookie和Session

服务器每次创建Session会话的时候，都会创建一个Cookie对象。这个Cookie对象的key永远都是:JSESSIONID值，是新创建出来的Session的id值。

`request.getSession()`

通过Cookie中的id值找到自己之前创建好的Session对象，并返回

