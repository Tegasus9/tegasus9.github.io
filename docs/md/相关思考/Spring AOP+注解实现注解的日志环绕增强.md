# Spring AOP+注解实现注解的日志环绕增强

## 实现效果

![image-20220402174857124](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204021748237.png)

## 遇到问题

### AOP切面不生效

在被切入的类上实现`@EnableAspectJAutoProxy`注解

### 切面被切入两次

切面注解上加了`@Component`注解，同时Spring进行组件扫描，于是组件切面被添加两次。