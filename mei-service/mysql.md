# 安装MySQL及可视化工具
1. 安装包
2. 切换至root用户：
```
sudo su
su -
```
3. DOS进入MySQL环境：
```
cd /usr/local/mysql/bin
mysql -u root -p
# 输入数据库密码
```
4. 可视化工具连接本地MySQL
```sql
#修改加密规则 
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;   
#更新一下用户的密码 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';   
#刷新权限
FLUSH PRIVILEGES;  
#单独重置密码
alter user 'root'@'localhost' identified by 'newpassword';
```
5. 创建数据表
```

```

6. 服务连接并使用数据库
* 添加依赖
```xml
<parent>    
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.16.RELEASE</version>
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
```
* 添加config
```Java
/**pom配置
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: bianjinyue20191120
*/
@Configuration
public class DataSourceConfig {
    @Bean(name = "dataSourceDevice")
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }
}
```
* mybatis自动生成
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 该处需要改动，每个人存放mysql-connector-java位置不同 -->
    <classPathEntry location="/Users/macuser/Documents/mavenRepo/mysql/mysql-connector-java/5.1.46/mysql-connector-java-5.1.46.jar"/>

    <context id="mysqlTables" targetRuntime="MyBatis3Simple" defaultModelType="flat" >

        <!-- TKmybatis配置 -->
        <property name="javaFileEncoding" value="UTF-8"/>
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper,tk.mybatis.mapper.common.MySqlMapper"/>
        </plugin>

        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>


        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/mei-service"
                        userId="root" password="bianjinyue20191120"/>

        <!-- 生成model模型，对应的包，存放位置可以指定具体的路径 -->
        <javaModelGenerator targetProject="src/main/java" targetPackage="com.project.mapper">
            <property name="enableSubPackages" value="true"/>
        </javaModelGenerator>

        <!--对应的xml mapper文件  -->
        <sqlMapGenerator targetProject="src/main/java" targetPackage="com.project.mapper">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!-- 对应的dao接口 -->
        <javaClientGenerator type="XMLMAPPER" targetProject="src/main/java"
                             targetPackage="com.project.mapper">
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!-- 表映射为实体 -->
        <table tableName="pro_test" domainObjectName="ProTest">
            <!-- 使用经典驼峰命名法 -->
            <property name="useActualColumnNames" value="false"/>
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>

    </context>
</generatorConfiguration>
```
* 数据表使用
```Java
@Configuration
@MapperScan(basePackages = {"com.project.mapper"},
        sqlSessionFactoryRef="sqlDeviceSessionFactory")
public class DeviceDBConfig {

    @Autowired
    @Qualifier(value = "dataSourceDevice")
    private DataSource dataSourceDevice;

    @Bean
    public SqlSessionFactory sqlDeviceSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceDevice);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath:com/project/*Mapper.xml"));
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlDeviceSessionTemplate() throws Exception {
        SqlSessionTemplate template = new SqlSessionTemplate(sqlDeviceSessionFactory());
        return template;
    }

}
```