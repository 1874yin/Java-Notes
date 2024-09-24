## SpringSecurity

认证 + 鉴权

认证：是否该系统的用户，即登录功能。

鉴权：是否具有访问某个资源的权限。

### 引入

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
```

在 `pom.xml` 文件中引入依赖，便会自动应用到项目中，此时访问项目页面，需要登录。

### 认证

#### 流程

`AbstractAuthenticationProcessingFilter`  实现类`UsernamePasswordAuthenticationFilter`  

提交用户名密码，封装 `Authentication` 对象（还没有权限），调用 `authenticate` 方法认证。

``AuthenticationManager` 实现类 `ProviderManager` 

调用 `DaoAuthenticationProvider` 的 `authenticate` 方法

`AbstractUserDetailsAuthenticationProvider` 实现类 `DaoAuthenticationProvider` 

调用 `loadUserByUsername` 方法查询用户

`UserDetailsService` 实现类 `InMemoryUserDetailsManager` 

根据用户名去查询对应的用户，及这个用户对应的权限信息。`InMemoryUserDetailsManager` 是在内存中查找。

把对应的用户信息包括权限信息封装成 `UserDetails` 对象，返回 `UserDetails` 对象。

通过 `PasswordEncoder` 对比 `UserDetails` 中的密码和 `Authentication` 的密码是否一致。

如果一致，则把 `UserDetails` 中的权限信息设置到 `Authentication` 对象中。返回 `Authenticaion` 对象。

如果上一步返回了 `Authentication` 对象，就用 `SecurityContextHolder.getContext().setAuthentication()` 方法存储该对象。其他过滤器会通过 `SecurityContextHolder` 来获取当前用户信息。

##### JWT 相关

利用 JWT，让前端携带 token 访问，即可避免每次都需要登录的问题。

认证通过后，生成一个 JWT 发给前端。前端在每次请求的时候带上该 JWT，后端接收到请求后，JWT认证过滤器会解析该 token，获取 userId，再获取 User 信息，封装 `Authentication` 对象存入 `SecurityContextHolder`。

#### 思路

登录：

1. 自定义登录接口
   - 调用 `ProviderManager` 的方法进行认证，认证成功生成 JWT
   - 把用户信息存入 redis 中
2. 自定义 `UserDetailsService` 
   - 在这个实现类中查询数据库，代替内存查找用户

校验：

1. 定义 JWT 认证过滤器
   - 获取 token
   - 解析 token，获取 userid
   - 从 redis 中获取用户信息
   - 封装 `Authentication` 对象，放入 `SecurityContextHolder`

#### 实现

##### 数据库校验用户



##### 密码加密存储



##### 登录接口



##### 认证过滤器



##### 退出登录



### 授权

#### 作用



#### 流程



#### 实现







