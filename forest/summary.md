你还在用HttpClient或OkHttp来发送http请求吗？甚至是用Java原生的java.net包?

不用再那么麻烦了，你现在可以像调用本地方法一样去调用远程rest接口了。

[Forest](https://github.com/mySingleLive/forest)通过动态代理模式将http请求代码业务接口代码，用标注申明http相关信息，这样就能带来很多好处：<br>
1、首先就是解耦。这不用多说了吧，将http请求相关信息（如用POST方法、加什么header之类的信息）和业务代码解耦后好处多多。从此业务代码就不必再关心此接口是如何发送请求的，只需要知道3件事（做什么的；给它什么；给我什么）。<br>
2、然后就是http请求集中化管理。以前项目中http请求代码散乱在各个角落里，同一个项目可能会有两三个不同的http框架同时存在，还有自个对http请求代码的封装。日积月累后代码自然难以管理。而Forest将所有http请求都集中在了接口文件中，而所有请求信息都如清单一般详细罗列在其中。如果调用接口的人用的是Eclipse或Intellij IDEA能很容易跳转到接口方法申明的地方，一眼就能看到HTTP的详细信息，这也是申明式编程的好处。
<br>

GitHub地址：[https://github.com/mySingleLive/forest](https://github.com/mySingleLive/forest)

码云地址：[https://gitee.com/dt_flys/forest](https://gitee.com/dt_flys/forest)

### 方法：

1. 添加Maven依赖
```xml
<dependency>
    <groupId>com.dtflys.forest</groupId>
    <artifactId>forest-core</artifactId>
    <version>1.0.0</version>
</dependency>

<dependency>
    <groupId>com.dtflys.forest</groupId>
    <artifactId>forest-spring</artifactId>
    <version>1.0.0</version>
</dependency>
```

2. 添加一个Interface: <br>
  com.mytest.client.MyClient
```java
package com.mytest.client;

import com.dtflys.forest.annotation.Request;
import com.dtflys.forest.annotation.DataParam;

public interface MyClient {
    /**
     * 百度短链接API
     * @param url
     * @return
     */
    @Request(
        url = "http://dwz.cn/create.php",
        type = "post",
        dataType = "json"
    )
    Map getShortUrl(@DataParam("url") String url);
}

```

   此接口会被作为Http请求调用的接口。方法上加上@Requst后才会被视为发送请求的方法。
@Requst中的参数有很多，其中有url（请求地址），type（http请求方法类型），dataType（接受的数据类型）等等。

2. 在resources目录下添加配置文件forest.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:forest="http://www.dtflys.com/schema/forest"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.dtflys.com/schema/forest
       http://www.dtflys.com/schema/forest/forest-spring.xsd">

   <forest:configuration
            id="forestConfiguration"
            timeout="30000"
            retryCount="3"
            connectTimeout="10000"
            maxConnections="500"
            maxRouteConnections="500">
    </forest:configuration>

    <forest:scan configuration="forestConfiguration"
                 base-package="com.mytest.client"/>
</beans>
```

在spring配置文件中加入import标签，把forest.xml导入到spring主配置。当然你直接把上面forest配置直接写在外面的spring配置中也是完全没有问题的，不过别忘了要把上面的xml schema贴过来，不然forest专用的xml标签将无法解析。

```xml
  <import resource="classpath:/forest.xml" />
```

3. 在业务的java Bean中注入MyClient类对象

```java
package com.mytest.service.impl

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.mytest.client.MyClient;

public class MyServiceImpl implements MyService {
    
    @Autowired
    private MyClient myClient;
    
}
```

4、然后就直接调用咯

```java

    public void doTest() {
        Map result = myClient.getShortUrl("https://gitee.com/dt_flys/forest");
        System.out.println(result);
    }

```

好了，是不是很简单，祝您码代码愉快 ;-)