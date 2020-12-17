# 单元测试

我主要在spring boot项目中写过单测，还没接触过其他的

一般单元测试在test文件夹下，test文件位置要与main文件同一级别。然后在test文件夹下编写对应的单元测试文件，依次覆盖。可以通过SonarQube平台进行查看覆盖率，需要自己配置到项目中。

## 接入SonarQube

在.gitlab-ci.yml文件中，增加：

```
install -Dmaven.test.skip=false sonar:sonar
-Dsonar.host.url=(https:sonar平台URL地址)
-Dsonar.gitlab.ignore_certificate=true
-Dsonar.core.codeCoveragePlugin=jacoco
```

(我也没自己新建项目试过，公司都封装好了的)

## 单元测试

### 引入pom依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>

<dependency>
	<groupId>org.powermock</groupId>
	<artifactId>powermock-module-junit4</artifactId>
	<version>2.0.2</version>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>org.powermock</groupId>
	<artifactId>powermock-api-mockito2</artifactId>
	<version>2.0.2</version>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>org.mockito</groupId>
	<artifactId>mockito-core</artifactId>
	<version>2.23.0</version>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>org.dbunit</groupId>
	<artifactId>dbunit</artifactId>
	<version>2.5.0</version>
	<type>jar</type>
	<scope>test</scope>
</dependency>
```

### 相关注解

- @SpringBootTest：测试类，应用于类
- @RunWith(SpringRunner.class)：测试类，应用于类
> 加了注解后再启动测试函数时，实际编译了整个项目，并不是单纯启动一个测试函数
- @InjectMocks：应用于函数变量，相当于@Autowired注解，新建一个对应类的实例
- @Mock：应用于函数变量，虚假mock实例，并没有实际意义

### 新建testService.java

```
package com.project.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import lombok.Data;

@Data
@Service
public class TestService {
    @Autowired
    private QueryData queryData;
    @Autowired
    private WebService webService;

    public void test(String str){
        /**
         * 调用同一个项目另一个类的函数
         */
        queryData.query(1);

        /**
         * 调用另一个项目某一个类的函数
         */
        webService.peint(str);
    }
}
```

### 编写对应的测试文件

```
import com.project.service.QueryData;
import com.project.service.TestService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.when;

@SpringBootTest
@RunWith(SpringRunner.class)
public class TestServiceTest {
    /**
     * 本项目中的类实例
     */
    @InjectMocks
    private QueryData queryData;
    /**
     * 另一个项目的类实例
     */
    @Mock
    private WebService webService;
    
    @Test
    public void test(){
        /**
         * 声明被测试类实例
         */
        TestService testService = new TestService();
        /**
         * 遇到调用另一个项目某一个类的方法时需要mock该方法的返回值
         */
        when(webService.print(anyString())).thenReturn("test");
        /**
         * 将被测试类的变量加入测试类实例，否则会报空指针错误
         */
        testService.setQueryData(queryData);
        testService.setWebService(webService);
        /**
         * 调用对应的测试方法
         */
        testService.test("test");
    }
    
}

```

> 其他单测的深入了解，自己去百度吧












