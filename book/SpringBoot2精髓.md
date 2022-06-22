> [学习过程中的实践项目地址](https://github.com/Consck/study-pro)

## IOC & AOP

### IOC

spring container 负责管理任意对象，并结合对 对象的描述进行初始化和加强。spring可以管理和增强任意对象，还可以为被管理的Bean提供其他被管理和被增强的Bean。

Bean通过注解@Service声明为一个Spring管理的Bean，Spring容器会扫描classpath下的所有类，找到带有@Service注解的UserService类，并根据Spring注解对其进行初始化和增强，如果发现此类属性crditUserService也有注解@Autowired，则会从Spring容器里查找一个已经初始化好的CreditUserService，如果没有，则先初始化CreditUserService。

在Spring boot中就是依靠注解成为容器管理的Bean。如在类上注解@Controller、@Service、@Configuration等，方法上注解@Bean。还有一种办法是实现Spring的某些接口类，Spring会在容器的生命周期里调用接口类，如BeanPostProcessor接口是在所有容器管理Bean初始化后会调用此接口实现，对Bean进一步配置。

### AOP

面向切面编程，通过预编译方式或者运行时刻对目标对象动态地添加功能。应用可以在运行时刻动态地在方法调用前后织入一些公共代码，从而提供系统的公共服务。

- @Configuration注解成功引起spring容器的注意
- @Aspect让spring容器知道 这是一个AOP类，在Aspect中会包含一些Pointcut及相应的Advice。
- Joint point：表示程序中明确定义的点，典型的包括方法调用、对类成员的访问，以及异常处理程序块的执行等。Spring中的joint point只支持方法调用。
- pointcut：一组joint point，用来判断在joint point中执行Advice
- Advice：定义在Pointcut里面定义的程序点具体要做的操作，通过before、around、after来区别是在每个joint Point 之前 、之后还是执行前后要调用的代码
  - before：在执行方法前调用Advice，比如cache功能可以在执行方法前先判断是否有缓存
  - around：在执行方法前后调用Advice
  - after：执行方法后调用Advance，after return是方法正常返回后调用，after throw是方法抛出异常后调用
  - finally：方法调用后执行Advance，无论是否抛出异常还是正常返回
- @Around是AOP的一种具体方式，能对目标方法调用前和调用后进行处理
- within(@org.springframework.stereotype.Controller*)可以理解为对所有使用@Controller注解的类进行AOP
- execution(public * *(..))：所有public方法，后面的星号代表类路径和方法名
- execution(* set*(..))：所有set开头的方法
- execution(public set*(..))：所有set开头的public方法
- execution(public com.xyz.service.* set*(..))：所有set开头的public方法，且位于com.xyz.service包下
- target(com.xyz.service.CommonService)：所有实现了CommonService接口的类的方法
- @target(org.springframework.transaction.Transaction)：所有用@Transaction注解的方法
- @annotation(function)表示另外一个条件，也就是对具有function参数对应的注解方法进行AOP
- functionAccessCheck是实现AOP的具体代码

对于spring boot应用，建议启动程序的包名层次最高，其他类均在其下，这样spring boot默认自动搜索启动程序之下的所有类。

- @Controller是spring MVC注解，表示此类用于负责处理Web请求
- @RequestMapping是Spring MVC注解，表示如果请求路径匹配，被注解的方法将被调用
- @ResponseBody表示此方法返回的是文本而不是视图名称
- @RestController相当于@Controller和@ResponseBody

## 热部署依赖包

在修改类或者配置文件时自动重新加载spring boot应用。LiveReload server用于监控spring  boot应用文件变化，是因为加了devtools依赖，另外启动时间变快。因为spring boot再次重启，避免了重启Tomcat server，也避免重启已经加载的spring相关类，只重新加载变化的类。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <version>2.7.0</version>
</dependency>
```

## maven基础

maven的核心是pom.xml，用xml方式描述项目模型：
- groupId：项目所属的组，通常是一个公司或组织的名称
- artifactId：项目唯一的标识
- packaging：项目的类型，常用的有jar和war两种
- version：项目的版本号，主版本号.次版本号.修订版本号。主版本号变动代表架构变动或者不兼容实现，次版本号是兼容性修改、功能增强，修订版本号则是bug修复。SNAPSHOT标识开发中的版本，会修复bug和添加新功能；RELEASE标识一个正式发布版，中间还可能有M1、M2、RC、GA等表示即将发布前的各个过程
- modelVersion：代表pom文件的maven版本
- dependencies：此元素下包含多个dependency
- denpendency：声明项目依赖
- scope：此类库与项目的关系，
  - 默认是compile，也就是编译和打包都需要此类库。
  - test表示仅仅在单元测试的时候需要；
  - provider表示在编译阶段需要此类库，但打包阶段不需要，这是因为项目的目标环境已经提供了；
  - runtime表示在编译和打包的时候都不需要，但在运行的时候需要。
- build：可选，包含多个插件plugin，辅助项目构建

```java
进入Maven安装目录，进入conf目录，编辑setting.xml；
找到mirros元素，添加仓库镜像。
<mirror>
    <id>alimaven</id>
    <name>aliyun mavem</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

- mvn compile：编译
- mvn package：编译并打包工程
- mvn install：打包并安装到本地仓库，其他maven项目可以通过项目坐标引用
- mvn deploy：同install，但打包并安装到远程仓库
- mvn clean：删除target目录

maven仓库两大类，第一类远程仓库，包括中心仓库，位于http://search.maven.org/；还包括镜像仓库，比如国内常用的镜像http://maven.aliyun.com/nexus/，还有利用nexus软件自己搭建的公司私服。还有一类是本地仓库。

pom指明spring boot 的仓库位置，添加如下内容：

```java
<repositories>
    <repository>
        <id>spring-snapshots</id>
        <url>http://repo.spring.io/snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>spring-milestones</id>
        <url>http://repo.spring.io/milestone</url>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>spring-snapshots</id>
        <url>http://repo.spring.io/snapshot</url>
    </pluginRepository>
    <pluginRepository>
        <id>spring-milestones</id>
        <url>http://repo.spring.io/milestone</url>
    </pluginRepository>
</pluginRepositories>

```

## 常用注解

Spring提供多个注解声明Bean为Spring管理的Bean，注解不同代表的含义不同，但对于Spring容器来说，都是Spring管理的Bean。

- Controller：声明此类是一个MVC类，通常与@RequestMapping一起使用
- Service：声明此类是一个业务处理类，通常与@Transactional一起配合使用
- Repository：声明此类是一个数据库或其他NoSQL访问类
- RestController：同Controller，用于rest服务
- Component：声明此类是一个Spring管理的类，通常用于无法用上述注解描述的Spring管理类
- Configuration：声明此类是一个配置类，与@Bean配合使用
- Bean：作用在方法上，声明该方法执行的返回结果是一个Spring容器管理的Bean

Spring负责实例化Bean，开发者可以提供一系列回调函数，用于进一步配置Bean，包括@POSTConstruct和@PreDestory注解。当Bean被容器初始化后，会调用@POSTConstruct的注解方法；在容器被销毁之前，会调用@PreDestory注解的方法。

Spring有两种方式来引用容器管理的Bean，一种是根据名字，为每个管理的Bean指定一个名字，随后可以通过名字引用此Bean，可以使用@Qualifier来引用。另外一种是根据类型(类名)，使用@Autowired引用，作用于属性或者构造函数参数，甚至是方法调用参数上。



