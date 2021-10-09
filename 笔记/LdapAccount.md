# LDAP

轻量级目录访问协议，OpenLDAP是最常用的目录服务之一，提供了目录服务的所有功能，包括目录搜索、身份认证、安全通道、过滤器等等。OpenLDAP服务默认使用非加密的TCP/IP协议来接收服务的请求，并将查询结果传回到客户端。由于大多数目录都是用于系统的安全认证部门，如用户登录和身份验证，所以也支持使用基于SSL/TLS的加密协议来保证数据传送的保密性和完整性。OpenLDAP 是使用 OpenSSL 来实现 SSL/TLS 加密通信的。

## 常用关键字列表

LDAP通过属性objectClass来控制哪一个属性必须出现或允许出现在一个条目中，它的值决定了该条目必须遵守的模式规则。

| 关键字 | 英文全称	 | 含义 |
|:--------|:--------:|--------:|
|dc| Domain Component|	域名的部分，其格式是将完整的域名分成几部分，如域名为example.com变成dc=example,dc=com|
|uid|	User Id	| 用户ID，如“tom”|
|ou	|Organization Unit	 |组织单位，类似于Linux文件系统中的子目录，它是一个容器对象，组织单位可以包含其他各种对象（包括其他组织单元），如“market”|
|cn|	Common Name	 |公共名称，如“Thomas Johansson”|
|sn	|Surname 	| 姓，如“Johansson”|
|dn	|Distinguished Name	| 惟一辨别名，类似于Linux文件系统中的绝对路径，每个对象都有一个惟一的名称，如“uid= tom,ou=market,dc=example,dc=com”，在一个目录树中DN总是惟一的|
|rdn	|Relative dn |	相对辨别名，类似于文件系统中的相对路径，它是与目录树结构无关的部分，如“uid=tom”或“cn= Thomas Johansson”|
|c	|Country	|国家，如“CN”或“US”等|
|o	|Organization	|组织名，如“Example, Inc.”|

<font face="微软雅黑" size=3 color=#FF0000 >[注意]</font> 目录服务不适于进行频繁的更新，属于典型的分布式结构。dn必须是全局唯一的。

## 功能

在LDAP的功能模型中定义了一系列利用LDAP协议的操作，主要包含以下4部分：
- 查询操作：允许查询目录和取得数据，其查询性能比关系数据库好。
- 更新操作：目录的更新操作没关系数据库方便，更新性能较差，但也同样允许进行添加、删除、修改等操作。
- 复制操作：前面也提到过，LDAP是一种典型的分布式结构，提供复制操作，可将主服务器的数据的更新复制到设置的从服务器中。
- 认证和管理操作：允许客户端在目录中识别自己，并且能够控制一个会话的性质。

Ldap具体高级功能：
- 实现账号统一集中管理；
- 权限控制策略管理；
- 密码控制策略管理；
- 密码审计管理；
- 主机控制管理；
- 同步机制管理；
- TLS/SASL加密传输；
- 高可用负载均衡架构；
- 自定义schema；
- 各种集中平台账号集中管理；

# JWT

为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准。该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

## 好处
- 支持跨域访问：cookie是不允许跨域访问的，这一点对token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输
- 无状态：token机制在服务端不需要存储session信息，因为token自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息
- 更适用CDN：可以通过内容分发网络请求你服务端的资料，而服务端只要提供API即可
- 去耦：不需要绑定到一个特定的身份验证方案。token可以在任何地方生成，只要在你的API被调用的时候，你可以进行token生成调用即可
- 更适用于移动应用：当客户端是一个原生平台时，cookie是不被支持的
- CSRF：在http请求中以参数的形式加入一个服务器端产生的token；放入http请求头中，一次性给所有该类请求加上csrftoken属性
- 性能：一次网络往返时间(通过数据库查询session信息)会比做一次HMACSHA256计算的token验证和解析费时
- 不需要为登录页面做特殊处理：
- 基于标准化

## 组成

header，payload，signature

### header

头部承载两部分信息：声明类型(jwt)、声明加密的算法(HMAC SHA256).将头部进行base64加密
```json
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

### payload

存放有效信息的地方，包含三部分信息：标准中注册的声明、公共的声明、私有的声明

### signature

header、payload、secret.这个部分需要base64加密后的header和加密后的payload使用连接组成字符串，然后通过header中声明的加密方式进行加盐secret组合加密，构成jwt的第三部分。

<font face="微软雅黑" size=3 color=#FF0000 >[注意]</font> secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret是用来进行jwt的签发和验证。所以，在任何场景都不应该泄露出去。一旦客户端得知这个secret，就意味着客户端可以自我签发jwt了

## 代码示例
- 依赖包
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```
- 示例代码
```java
/**
    * 创建jwt
    * @param id
    * @param subject
    * @param ttlMillis 过期的时间长度
    * @return
    * @throws Exception
    */
public String createJWT(String id, String subject, long ttlMillis) throws Exception {
    SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256; //指定签名的时候使用的签名算法，也就是header那部分，jjwt已经将这部分内容封装好了。
    long nowMillis = System.currentTimeMillis();//生成JWT的时间
    Date now = new Date(nowMillis);
    Map<String,Object> claims = new HashMap<String,Object>();//创建payload的私有声明（根据特定的业务需要添加，如果要拿这个做验证，一般是需要和jwt的接收方提前沟通好验证方式的）
    claims.put("uid", "DSSFAWDWADAS...");
    claims.put("user_name", "admin");
    claims.put("nick_name","DASDA121");
    SecretKey key = generalKey();//生成签名的时候使用的秘钥secret,这个方法本地封装了的，一般可以从本地配置文件中读取，切记这个秘钥不能外露哦。它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
    //下面就是在为payload添加各种标准声明和私有声明了
    JwtBuilder builder = Jwts.builder() //这里其实就是new一个JwtBuilder，设置jwt的body
            .setClaims(claims)          //如果有私有声明，一定要先设置这个自己创建的私有的声明，这个是给builder的claim赋值，一旦写在标准的声明赋值之后，就是覆盖了那些标准的声明的
            .setId(id)                  //设置jti(JWT ID)：是JWT的唯一标识，根据业务需要，这个可以设置为一个不重复的值，主要用来作为一次性token,从而回避重放攻击。
            .setIssuedAt(now)           //iat: jwt的签发时间
            .setSubject(subject)        //sub(Subject)：代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，可以存放什么userid，roldid之类的，作为什么用户的唯一标志。
            .signWith(signatureAlgorithm, key);//设置签名使用的签名算法和签名使用的秘钥
    if (ttlMillis >= 0) {
        long expMillis = nowMillis + ttlMillis;
        Date exp = new Date(expMillis);
        builder.setExpiration(exp);     //设置过期时间
    }
    return builder.compact();           //就开始压缩为xxxxxxxxxxxxxx.xxxxxxxxxxxxxxx.xxxxxxxxxxxxx这样的jwt
    //打印了一哈哈确实是下面的这个样子
    //eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJEU1NGQVdEV0FEQVMuLi4iLCJzdWIiOiIiLCJ1c2VyX25hbWUiOiJhZG1pbiIsIm5pY2tfbmFtZSI6IkRBU0RBMTIxIiwiZXhwIjoxNTE3ODI4MDE4LCJpYXQiOjE1MTc4Mjc5NTgsImp0aSI6Imp3dCJ9.xjIvBbdPbEMBMurmwW6IzBkS3MPwicbqQa2Y5hjHSyo
}

/**
    * 解密jwt
    * @param jwt
    * @return
    * @throws Exception
    */
public Claims parseJWT(String jwt) throws Exception{
    SecretKey key = generalKey();  //签名秘钥，和生成的签名的秘钥一模一样
    Claims claims = Jwts.parser()  //得到DefaultJwtParser
        .setSigningKey(key)         //设置签名的秘钥
        .parseClaimsJws(jwt).getBody();//设置需要解析的jwt
    return claims;
}

/**
    * 由字符串生成加密key
    * @return
    */
public SecretKey generalKey(){
    String stringKey = Constant.JWT_SECRET;//本地配置文件中加密的密文7786df7fc3a34e26a61c034d5ec8245d
    byte[] encodedKey = Base64.decodeBase64(stringKey);//本地的密码解码[B@152f6e2
    System.out.println(encodedKey);//[B@152f6e2
    System.out.println(Base64.encodeBase64URLSafeString(encodedKey));//7786df7fc3a34e26a61c034d5ec8245d
    SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");// 根据给定的字节数组使用AES加密算法构造一个密钥，使用 encodedKey中的始于且包含 0 到前 leng 个字节这是当然是所有。（后面的文章中马上回推出讲解Java加密和解密的一些算法）
    return key;
}

```
- 实践代码
  
```Java
public static String getToken(Map<String,String> map) throws IOException {
    Account account = new Account();
    String user = map.getOrDefault("user", "");
    String psd = map.getOrDefault("psd", "");
    account.setUserName(user);
    account.setPassword("你猜猜");
    ObjectMapper objectMapper = new ObjectMapper();
    //account内容可以结合LdapAccount技术，不能把密码放到token生成过程中，base64解码第二段Jwt就可以获取到内容信息，仅可把用户名放入subject中
    return Jwts.builder().setSubject(objectMapper.writeValueAsString(account)).signWith(SignatureAlgorithm.HS512,"good").compact();
}

public static void parse(String token){
    String c = Jwts.parser().setSigningKey("good").parseClaimsJws(token).getBody().getSubject();
    System.out.println(c);
}

public static void main(String[] args) throws IOException {
    Map map = Maps.newHashMap();
    map.put("user", "1");
    map.put("psd", "2");
    String token = getToken(map);
    System.out.println(token);

    parse(token);
}
```