## SpringBoot 自动配置机制

#### 触发机制

SpringBoot 自动配置机制基于 `@EnableAutoConfiguration` 注解，通常与 `@SpringBootApplication` 一起使用。`@EnableAutoConfiguration` 会触发自动配置的流程。

#### 实现机制

`@EnableAutoConfiguration` 注解会导入 `AutoConfigurationImportSelector`，该类负责加载自动配置类。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

`AutoConfigurationImportSelector` 是一个 `DeferredImportSelector` ，它会加载 spring.factories 文件中定义的自动配置类。

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // 加载spring.factories文件中的自动配置类
        return getCandidateConfigurations(annotationMetadata, new SpringFactoriesLoader.FactoryClassLoader());
    }
}
```

Spring Boot 通过 spring.factories 文件定义了自动配置类。spring.factories 文件位于 META-INF 目录下，内容如下：

```pro
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
...
```

#### 条件判断

Spring Boot 通过条件注解来判断是否启动某个自动配置类。常见的条件注解包括：

- @ConditionalOnClass 当类路径中存在指定类时，启用配置
- @ConditionalOnMissingBean 当容器中没有指定类型的 Bean 时，启动配置
- @ConditionalOnProperty 当配置文件中存在指定属性时，启用配置
- @ConditionalOnWebApplication 当应用是 Web 应用时，启动配置

#### 加载顺序

由 `AutoConfigureOrder` 注解决定，如果没有指定顺序，Spring Boot 会按照 spring.factories 文件中的顺序加载。

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
public class DataSourceAutoConfiguration {
    
}
```

#### 禁用自动配置

- `@EnableAutoConfiguration` 的 exclude 属性：

  ```java
  @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
  public class MyApplication {
      
  }
  ```

- `application.properties` 文件：

  ```properties
  spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
  ```

总结：

1. `@EnableAutoConfiguration` 触发自动配置机制
2. `AutoConfigurationImportSelector` 加载 spring.factories 文件中的自动配置类
3. `spring.factories` 定义自动配置类
4. `@ConditionalOn***` 条件注解决定是否启用某个配置
5. `@AutoConfigureOrder` 或 `spring.factories` 文件中的顺序决定加载顺序