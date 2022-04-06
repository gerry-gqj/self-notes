```java
public class JwtTest {

/*  <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
        <version>3.15.0</version>
    </dependency>*/

    @Test
    public void createJwt(){
        HashMap<String, Object> map = new HashMap<>();


        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.SECOND,60*10);

        String token = JWT.create()
                /*负荷
                * 注意:请不要将敏感信息存放在token payload里面
                * */
                .withHeader(map)
                .withClaim("name", "YourName")
                .withClaim("password", 123)


                /*签证过期时间*/
                .withExpiresAt(instance.getTime())
                /*认证盐
                * 密钥:请注意不要公开此密钥
                * */
                .sign(Algorithm.HMAC256("toke!#*^$123"));
        System.out.println(token);
    }

    @Test
    public void verificationToken(){
        JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("toke!#*^$123")).build();
        DecodedJWT verify = jwtVerifier.verify("eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXNzd29yZCI6MTIzLCJuYW1lIjoiWW91ck5hbWUiLCJleHAiOjE2MTkxNzA5NjZ9.mUsKBOiWG3jWXoL9PM7GcefWLy-Kw9VMwKEgO0AiRr0");
            System.out.println(verify.getHeader());
            System.out.println(verify.getExpiresAt());
            System.out.println(verify.getClaim("name").asString());
            System.out.println(verify.getClaim("password").asInt());
            System.out.println(verify.getSignature());
    }
}
```



~~~java
public class JwtUtils{

    private static final String SIGN = "!token*($^HJKHKJHSAF(@dd";

    /*生成token令牌*/
    public static String getToken(Map<String,String> map){
        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.DATE,7);
        JWTCreator.Builder builder = JWT.create();
        map.forEach((k,v)->{
            builder.withClaim(k,v);
        });
        String Token = builder.withExpiresAt(instance.getTime())
                .sign(Algorithm.HMAC256(SIGN));
        return Token;
    }

    /*验证token令牌是否合法*/
    public static void verifyToken(String token){
      JWT.require(Algorithm.HMAC256(SIGN)).build().verify(token);
    }


    /*获取token信息*/
    public static DecodedJWT getTokenInfo(String token){
        return JWT.require(Algorithm.HMAC256(SIGN)).build().verify(token);
    }
}
~~~



```java
try{
    JwtUtils.verifyToken(token);
    log.info("token签名有效");
    return true;
} catch (SignatureVerificationException e){
    log.info("token签名无效");
    map.put("msg","无效签名");
    map.put("state","false");
}
catch (TokenExpiredException e){
    log.info("token签名已过期");
    map.put("msg","token已过期");
    map.put("state","false");
}catch (AlgorithmMismatchException e){
    e.printStackTrace();
    map.put("msg","token算法不一致");
    log.info("token签名算法有误");
    map.put("state","false");
}catch (Exception e){
    map.put("msg","token无效");
    log.info("token无效");
    map.put("state","false");
}
```

