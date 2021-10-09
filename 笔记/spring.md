 2021年7月13，后续待学习

 # SpringMvc

## WebMvcConfigurerAdapter

spring MVC主要的配置都可以通过继承WebMvcConfigurerAdapter(或者WebMvcConfigurationSupport)类进行修改，这两个类主要方法有：
- addFormatters:增加格式化工具，用于接收参数
- configureMessageConverters: 配置消息转换器，用于@RequestBody和@ResponseBody
- configurePathMatch：配置路径映射
- addArgumentResolvers:配置参数解析器，用于接收参数
- addInterceptors：添加拦截器
- configureContentNegotiation:
- configureAsyncSupport:
- configureDefaultServletHandling:
- addResourceHandlers: 静态资源
- addCorsMappings: 跨域
- addViewControllers: 跳转指定页面
- configureViewResolvers:
- addArgumentResolvers:
- addReturnValueHandlers:
- extendMessageConverters:
- configureHandlerExceptionResolvers:
- extendHandlerExceptionResolvers:
- getValidator:
- getMessageCodesResolver:
  
总之几乎所有关于spring MVC都可以在这个类中配置。只需要将其设为```@Configuration```，spring boot就会在运行时加载这些配置。

### WebMvcConfigurerAdapter使用方式
#### 过时方式：继承WebMvcConfigurerAdapter
Spring 5.0 以后WebMvcConfigurerAdapter会取消掉WebMvcConfigurerAdapter是实现WebMvcConfigurer接口
```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
    //TODO
}
```
#### 现用方式
1）实现WebMvcConfigurer，需要实现全部接口
```java
@Configuration
public class WebMvcConfg implements WebMvcConfigurer {
    //TODO
}
```
2）继承WebMvcConfigurationSupport，按需实现
```java
@Configuration
public class WebMvcConfg extends WebMvcConfigurationSupport {
    //TODO
}
```
### addInterceptors

- addInterceptors：需要一个实现HandlerInterceptor接口的拦截器实例
- addPathPatterns: 用于设置拦截器的过滤路径规则
- excludePathPatterns: 用于设置不需要拦截的过滤规则

### addCorsMappings:跨域
```java
@Override
public void addCorsMappings(CorsRegistry registry) {
    super.addCorsMappings(registry);
    registry.addMapping("/cors/**")
            .allowedHeaders("*")
            .allowedMethods("POST","GET")
            .allowedOrigins("*");
}
```
### addViewControllers:跳转指定页面 
```java
@Override
 public void addViewControllers(ViewControllerRegistry registry) {
     super.addViewControllers(registry);
     registry.addViewController("/").setViewName("/index");
     //实现一个请求到视图的映射，而无需书写controller
     registry.addViewController("/login").setViewName("forward:/index.html");  
}
```
### resourceViewResolver：视图解析器
```java
/**
 * 配置请求视图映射
 * @return
 */
@Bean
public InternalResourceViewResolver resourceViewResolver()
{
    InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
    //请求视图文件的前缀地址
    internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
    //请求视图文件的后缀
    internalResourceViewResolver.setSuffix(".jsp");
    return internalResourceViewResolver;
}

/**
 * 视图配置
 * @param registry
 */
@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
    super.configureViewResolvers(registry);
    registry.viewResolver(resourceViewResolver());
    /*registry.jsp("/WEB-INF/jsp/",".jsp");*/
}
```
### configureMessageConverters：信息转换器

```java
/**
* 消息内容转换配置
 * 配置fastJson返回json转换
 * @param converters
 */
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    //调用父类的配置
    super.configureMessageConverters(converters);
    //创建fastJson消息转换器
    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
    //创建配置类
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    //修改配置返回内容的过滤
    fastJsonConfig.setSerializerFeatures(
            SerializerFeature.DisableCircularReferenceDetect,
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.WriteNullStringAsEmpty
    );
    fastConverter.setFastJsonConfig(fastJsonConfig);
    //将fastjson添加到视图消息转换器列表内
    converters.add(fastConverter);

}
```
### addResourceHandlers：静态资源
```java
 @Override
 public void addResourceHandlers(ResourceHandlerRegistry registry) {
     //处理静态资源的，例如：图片，js，css等
     registry.addResourceHandler("/resource/**").addResourceLocations("/WEB-INF/static/");
 }
```

## HandlerInterceptorAdapter

在springboot中我们可以使用这个适配器来实现自己的拦截器，这样就可以拦截所有的请求并做相应的处理。应用场景：日志记录(可以记录请求信息的日志，以便进行信息监控、信息统计等)；权限检查(如登录检测)；性能监控(慢日志).

在HandlerInterceptorAdapter中主要提供了以下的方法：
- perHandle：在方法被调用前执行。在该方法中可以做类似校验的功能。如果返回true，则继续调用下一个拦截器。如果返回false，则中断执行。也就是说我们想调用的方法不会被执行，可以修改response为想要的内容。默认是true。
- postHandle：在方法执行后调用。
- afterCompletion：在整个请求处理完毕后进行回调，也就是视图渲染完毕或者调用方已经拿到响应。

如果实现HandlerInterceptor接口的话，三个方法必须实现，此时spring提供一个HandlerInterceptorAdapter适配器，允许我们只实现需要的回调方法。运行流程如下：
- 拦截器执行顺序是按照Spring配置文件中定义的顺序而定的
- 会先按照顺序执行拦截器的perHandler方法，一般遇到return false为止。
- 执行主方法(自己的controller接口)，若中间抛出异常，则不会继续执行postHandler，只会倒序执行afterCompletion方法
- 在主方法执行完业务逻辑时，按倒序执行postHandler方法。若第三个拦截器的preHandler方法return false，则会执行第二个和第一个的postHandler方法和afterCompletion(postHandler都执行完才会执行afterCompletion，postHandler和afterCompletion都是倒序执行).

## 代码实践
### 第一个拦截器
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author bianjinyue
 * @Description
 * @date 2021-08-04 10:38
 */
@Configuration
public class OneInterceptor implements HandlerInterceptor {

    /**
     * 调用方法前之前
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("11");
        String requestURI = httpServletRequest.getRequestURI();
        //获取的是httpServletRequest.request.coyoteRequest.uriMB，得到请求路径
        System.out.println(requestURI);
        return true;
    }

    /**
     * 调用方法后执行
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("12");
    }

    /**
     * 请求方接收返回值后回调
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o
     * @param e
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("13");
    }
}
```
### 第二个拦截器
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author bianjinyue
 * @Description
 * @date 2021-08-04 10:38
 */
@Configuration
public class TwoInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("21");
        RequestWrapper requestWrapper = new RequestWrapper(httpServletRequest);
        // 获取请求参数
        String body = requestWrapper.getBody();
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("22");
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("23");
    }
}
```
### 解析http请求
```java
import lombok.extern.slf4j.Slf4j;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

/**
 * @author bianjinyue
 * @Description
 * @date 2021-08-04 10:55
 */
@Slf4j
public class RequestWrapper extends HttpServletRequestWrapper {

    private final String body;

    public RequestWrapper(HttpServletRequest request) {
        super(request);
        StringBuilder stringBuilder = new StringBuilder();
        BufferedReader bufferedReader = null;
        InputStream inputStream = null;
        try {
            inputStream = request.getInputStream();
            if (inputStream != null) {
                bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                char[] charBuffer = new char[128];
                int bytesRead = -1;
                while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
                    stringBuilder.append(charBuffer, 0, bytesRead);
                }
            } else {
            }
        } catch (IOException ex) {

        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bufferedReader != null) {
                try {
                    bufferedReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        body = stringBuilder.toString();
    }
    public String getBody() {
        return this.body;
    }
}
```
### 添加拦截器
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;


/**
 * @author bianjinyue
 * @Description
 * @date 2021-08-04 10:13
 */
@Configuration
public class WebAdapter extends WebMvcConfigurationSupport {
    @Autowired
    private OneInterceptor oneInterceptor;
    @Autowired
    private TwoInterceptor twoInterceptor;

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);
        registry.addInterceptor(oneInterceptor)
                .addPathPatterns("/**");
        registry.addInterceptor(twoInterceptor)
                .addPathPatterns("/**");
    }
}
```
### 运行结果示例
输出顺序依次为：11 21 (执行方法) 22 12 23 13.拦截器中的postHandle、afterCompletion均为倒序执行。且afterCompletion必须等所有拦截器的postHandle都执行完毕后才开始倒序执行。