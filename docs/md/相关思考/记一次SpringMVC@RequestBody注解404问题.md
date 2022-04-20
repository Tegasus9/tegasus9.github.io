

# 记一次SpringMVC@RequestBody注解404问题。

本人在springboot经常使用@RequestBody注解，方便校验和保持代码整洁，也方便日后对入参进行统一管理。然而将这一套搬上SpringMVC却提示404。

## 1. 路径访问阶段

代码如下：

```java
public String showUserCustomInfo(@Valid @RequestBody UserCustomerInfoReq req) throws Exception
```

一开始以为是路径问题，但后来将@RequestBody去掉改用最原始的String。代码如下：

```java
public String showUserCustomInfo(String phone) throws Exception
```

这时请求便可以正常发送和映射。

这时便去百度相关资料，做了如下操作：

1. 在`spring-servlet.xml`中添加如下配置：

`<mvc:annotation-driven />`

2. 导入相关JAR包依赖：

    ```java
     		<dependency>
                <groupId>org.codehaus.jackson</groupId>
                <artifactId>jackson-core-asl</artifactId>
                <version>1.9.13</version>
            </dependency>
            <dependency>
                <groupId>org.codehaus.jackson</groupId>
                <artifactId>jackson-mapper-asl</artifactId>
                <version>1.9.13</version>
            </dependency>   
            <dependency>
                <groupId>org.codehaus.jackson</groupId>
                <artifactId>jackson-mapper-lgpl</artifactId>
                <version>1.9.6</version>
            </dependency>     
    ```

3. 再次测试相关请求。

![image-20220407151853185](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071518219.png)

可以正常访问，至此@RequestBody导致的404问题到此告一段落，**但是从返回报文中的乱码来看，新注解带来的一系列问题远没有到此结束。**

## 2. 返回值乱码阶段。



遇到返回值乱码时，首先看看对方返回的报文是否存在乱码。

进入debug：

![image-20220407152432953](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071524993.png)

可以看到是RestTemplate在返回response时返回了一串乱码。

![image-20220407152521912](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071525946.png)

一般乱码有两种情况。一是对方返回的报文本身就是乱码，二是在框架在解析报文时发生了乱码。一般情况下，只要接口提供方对暴露接口进行过相关测试，返回报文不会存在乱码问题。于是考虑是RestTemplate相关配置问题。

进行相关资料查找并查看RestTemplate相关消息转换器。

![image-20220407152834298](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071528333.png)

这里明显看到`StringHttpMessageConverter`默认编码是`"ISO-8859-1"`

问题就在这里。大概率考虑是项目没有对RestTemplate进行相关编码配置。于是查看配置项里相关Bean.

![image-20220407153047226](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071530273.png)

如此简陋，以至于没有更改任何有关消息转换器相关配置。

于是我们自定义消息转换器进行替换：

如下。

```java
restTemplate.getMessageConverters().set(1,new StringHttpMessageConverter(StandardCharsets.UTF_8));
```

![image-20220407153248312](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071532372.png)

这下应该不会有乱码问题。

再次调用接口进行测试。

![image-20220407153358348](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071533381.png)

可以看到乱码已经消失。本以为这次debug可以告一段落。

**然而当我们查看接口调用工具返回值一栏时可以发现**：

![image-20220407153503098](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071535129.png)

## **3. ？？？？阶段**

我的心情当时就和他显示的字符串一样感到`？？？？`

再次进行相关资料查找，并查看响应头。

![image-20220407153642195](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071536225.png)

已经变成熟悉的`ISO-8859-1`

怀疑是`<mvc:annotation-driven />`注解覆盖了某些设置，导致响应头变回了最原始的样子

于是我们注释这个注解并调用其他原有接口查看。

![image-20220407154207894](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071542932.png)

可以看到响应头中的charset已经变回我们想要的`UTF-8`

至此可以断定是`<mvc:annotation-driven />`覆盖了一些原有的设置。

于是我们再次查找相关资料

并将`spring-servlet.xml`中的原有配置进行修改。

原有配置：

```java
    <bean
            class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean
                        class="org.springframework.http.converter.StringHttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>text/html;charset=UTF-8</value>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>
            </list>
        </property>
    </bean>
    <mvc:annotation-driven/>
```

更改后的配置：

```java
    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

再次启动项目进行接口调用测试。

现有@RequestBody接口：

![image-20220407151505406](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071548912.png)

查看响应头：`UTF-8`

![image-20220407151536594](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071515619.png)

原有无@RequestBody接口：

![image-20220407155326364](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204071553395.png)

响应正常。且响应头也为`UTF-8`

不过怀疑之前所做相关`jackson`包导入是多余动作，因为公司有导入jackson相关包，于是进行尝试。

在删除相关依赖后进行项目接口测试。仍旧可以正常访问。

## 问题解决 & 相关思考

**至此，@RequestBody接口访问404问题得到解决。**

纵观最后的解决办法，无非就只有一步：**更改`spring-servlet.xml`配置项。**

但实际查资料和探索所花费的时间却远远不止更改配置项这么点。debug结束后仍然还有许多疑问等着未来的自己进行揭发，比如

`<mvc:annotation-driven/>`**到底做了些什么事？**

**该注解驱动在何处影响返回头？**

 **为什么不启用该配置项时会无法使用@RequestBody注解并导致404？**



# 2022年4月18日 勘误后记

**然而，事实的真相远非如此。相信一些经验丰厚的大佬们能从前篇文章中看出端倪。**

随着知识的增进，我将这篇文章转给一位大佬请他帮忙观摩思路不足之处，他立刻就表示在方法上添加注解一般不会导致404，于是坐在电脑前同我一起复现问题的存在。

经过一下午的调试。（实际上大部分时间都浪费在了项目启动上，每修改一个配置，启动时长短则50秒，长则一两分钟，很是折磨。）

最终发现问题的根源来源于项目中没有配置`MappingJackson2HttpMessageConverter`导致返回的数据无法被解析成json数据，而是变成访问`ip:port/context/returnMsg`

最终在访问路径的时候，由于返回的路径都是JSON字符串，所以当然报错404而无法访问了，实际上controller中的方法是可以访问的。只是自己并没有意识到问题所在而已，如果当初发现问题的时候能及时查看被调用的方法是否进入，也许就能更早一步发现问题存在。



