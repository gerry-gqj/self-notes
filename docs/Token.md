### Token



**Jwt依赖包**

> ```xml
>         <dependency>
>             <groupId>com.auth0</groupId>
>             <artifactId>java-jwt</artifactId>
>             <version>3.15.0</version>
>         </dependency>
> ```



> ```java
>     @Test
>     public void createJwt(){
>         HashMap<String, Object> map = new HashMap<>();
>         
>         Calendar instance = Calendar.getInstance();
>         
>         instance.add(Calendar.SECOND,60*10);
> 
>         String token = JWT.create()
>                 /*负荷
>                 * 注意:请不要将敏感信息存放在token payload里面
>                 * */
>                 .withHeader(map)
>                 .withClaim("name", "YourName")
>                 .withClaim("password", 123)
> 
>                 /*签证过期时间*/
>                 .withExpiresAt(instance.getTime())
>                 /*认证盐
>                 * 密钥:请注意不要公开此密钥
>                 * */
>                 .sign(Algorithm.HMAC256("toke!#*^$123"));
>         
>         System.out.println(token);
>     }
> ```



> ```java
>     @Test
>     public void verificationToken(){
>         JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("toke!#*^$123")).build();
>         DecodedJWT verify = jwtVerifier.verify("eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXNzd29yZCI6MTIzLCJuYW1lIjoiWW91ck5hbWUiLCJleHAiOjE2MTkxNzA5NjZ9.mUsKBOiWG3jWXoL9PM7GcefWLy-Kw9VMwKEgO0AiRr0");
>         
>         	//获取token头
>             System.out.println(verify.getHeader());
>         	//eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
>         
>         	//获取过期时间
>             System.out.println(verify.getExpiresAt());
>         
>         	//获取token有效负载
>             System.out.println(verify.getClaim("name").asString());
>             System.out.println(verify.getClaim("password").asInt());
>         
>         	//获取签名
>             System.out.println(verify.getSignature());
>         	//mUsKBOiWG3jWXoL9PM7GcefWLy-Kw9VMwKEgO0AiRr0
>     }
> ```



### 封装Token工具类

> ```Java
> public class JwtUtils{
> 
>     private static final String SIGN = "!token*($^HJKHKJHSAF(@dd";
> 
>     /*生成token令牌方法*/
>     public static String getToken(Map<String,String> map){
>         
>         //设置过期时间
>         Calendar instance = Calendar.getInstance();
>         instance.add(Calendar.DATE,7);
>         
>         //生成创建器，一下通过创建器设置token的参数(token类型、携带参数、算法签名)
>         JWTCreator.Builder builder = JWT.create();
>         
>         //设置有效负载
>         //注意不要将敏感信息存储在token里面，并且设置参数避免固定参数
>         //可以设置随机盐或者是时间戳，保证token不要被第三方破解
>         //计算被破解了，token里面的信息并不会泄漏服务器的信息
>         map.forEach((k,v)->{
>             builder.withClaim(k,v);
>         });
>         
>         //生成token令牌
>         String Token = builder.withExpiresAt(instance.getTime())
>                 .sign(Algorithm.HMAC256(SIGN));
>         return Token;
>     }
> 
>     /*验证token令牌是否合法*/
>     public static void verifyToken(String token){
>       JWT.require(Algorithm.HMAC256(SIGN)).build().verify(token);
>     }
> 
>     /*获取token信息*/
>     public static DecodedJWT getTokenInfo(String token){
>         return JWT.require(Algorithm.HMAC256(SIGN)).build().verify(token);
>     }
> }
> ```



### 使用Token进行验证

>```java
>//注意:进行token验证时可能出现以下几种异常，当然如果token正确的话是不会捕获到异常
>//						1、签名异常 SignatureVerificationException 
>//						2、token过期异常 TokenExpiredException 
>//						3、token签名算法异常 AlgorithmMismatchException 
>
>
>try{
>    JwtUtils.verifyToken(token);
>    log.info("token签名有效");
>    return true;
>} catch (SignatureVerificationException e){
>    log.info("token签名无效");
>}
>catch (TokenExpiredException e){
>    log.info("token签名已过期");
>}catch (AlgorithmMismatchException e){
>    log.info("token签名算法有误");
>}catch (Exception e){
>    log.info("token无效");
>}
>```















### 封装Cookie工具类

> ```java
> public class CookieUtil {
> 
>     public static void set(HttpServletResponse response, String name , String value, int maxAge) {
> 
>         Cookie cookie = new Cookie(name,value);
>         cookie.setPath("/");
>         cookie.setMaxAge(maxAge);
>         cookie.setSecure(true);
>         response.addCookie(cookie);
>     }
> 
>     public static Cookie get(HttpServletRequest request, String name) {
>         
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
> 
> ```









