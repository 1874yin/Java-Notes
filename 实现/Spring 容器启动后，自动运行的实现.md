### Spring 容器启动后，自动运行的实现

#### 使用 `@PostConstruct` 注解

用于标记一个方法，在 Bean 初始化完成后执行。

```java
@Component
@Slf4j
public class MyBean {

    @PostConstruct
    public void init() {
        log.info("Spring容器启动完成，自动运行代码");
    }
}
```

运行结果：

```shell
: Initializing Spring embedded WebApplicationContext
: Root WebApplicationContext: initialization completed in 608 ms
: Spring容器启动完成，自动运行代码
: Tomcat started on port(s): 8080 (http) with context path ''
: Started SpringAutoRunDemoApplication in 1.086 seconds (process running for 1.405)
```

在 Spring 容器初始化完成之后，Tomcat 启动完成前运行。

#### 实现 `ApplicationRunner` 接口

实现该接口的类会在 Spring 容器启动后自动调用 run 方法

```java
@Component
@Slf4j
public class MyApplicationRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("Spring容器启动完成，自动运行代码");
    }
}
```

运行结果：

```shell
: Initializing Spring embedded WebApplicationContext
: Root WebApplicationContext: initialization completed in 534 ms
: Tomcat started on port(s): 8080 (http) with context path ''
: Started SpringAutoRunDemoApplication in 1.005 seconds (process running for 1.333)
: Spring容器启动完成，自动运行代码
```

Spring 启动过程完成后运行。

#### 实现 `CommandLineRunner` 接口

与 `ApplicationRunner` 类似，但允许访问命令行参数。

```java
@Component
@Slf4j
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        log.info("Spring容器启动完成，自动运行代码");
    }
}
```

运行结果：

```shell
: Initializing Spring embedded WebApplicationContext
: Root WebApplicationContext: initialization completed in 526 ms
: Tomcat started on port(s): 8080 (http) with context path ''
: Started SpringAutoRunDemoApplication in 0.991 seconds (process running for 1.315)
: Spring容器启动完成，自动运行代码
```

#### 使用 `@EventListener` 监听 `ContextRefreshedEvent` 

可以在 Spring 容器刷新后执行代码。

```java
@Component
@Slf4j
public class MyEventListener {

    @EventListener
    public void onApplicationEvent(ContextRefreshedEvent event) {
        log.info("Spring容器启动完成，自动运行代码");
    }
}
```

运行结果：

```shell
: Initializing Spring embedded WebApplicationContext
: Root WebApplicationContext: initialization completed in 528 ms
: Tomcat started on port(s): 8080 (http) with context path ''
: Spring容器启动完成，自动运行代码
: Started SpringAutoRunDemoApplication in 1.012 seconds (process running for 1.347)
```

在 Tomcat 容器启动完成后运行。

#### 使用 `@Bean` 方法

在**配置类**中，通过 @Bean 定义一个 Bean，并在方法中执行需要的代码。

```java
@Configuration
@Slf4j
public class MyConfiguration {

    @Bean
    public void init() {
        log.info("Spring容器启动完成，自动运行代码");
    }

}
```

运行结果：

```shell
: Initializing Spring embedded WebApplicationContext
: Root WebApplicationContext: initialization completed in 540 ms
: Spring容器启动完成，自动运行代码
: Tomcat started on port(s): 8080 (http) with context path ''
: Started SpringAutoRunDemoApplication in 1.041 seconds (process running for 1.44)
```

同样是在 Spring 容器初始化完成之后运行。



