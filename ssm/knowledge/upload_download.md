## 上传文件
>方法：POST（必须）
>
>编码：form/data

- MultipartFile对象
>Spring MVC会将上传的文件绑定到MultipartFile对象中。MultipartFile提供了获取上传文件内容、文件名等方法。
>通过transferTo()方法还可以将文件存储到硬件中，MultipartFile对象中的常用方法如下：
>
> byte[] getBytes()：获取文件数据
>
> String getContentType[]：获取文件MIME类型，如image/jpeg等
>
> InputStream getInputStream()：获取文件流
>
> String getName()：获取表单中文件组件的名字
>
>String getOriginalFilename()：获取上传文件的原名
>
>Long getSize()：获取文件的字节大小，单位为byte
>
>boolean isEmpty()：是否有上传文件
>
>void transferTo(File dest)：将上传文件保存到一个目录文件中

- spring-mvc.xml中的配置
```xml
<!-- 视图解析器  -->
     <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <!-- 前缀 -->
        <property name="prefix">
            <value>/WEB-INF/content/</value>
        </property>
        <!-- 后缀 -->
        <property name="suffix">
            <value>.jsp</value>
        </property>
    </bean>

   <bean id="multipartResolver"  
        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">  
        <!-- 上传文件大小上限，单位为字节（10MB） -->
        <property name="maxUploadSize">  
            <value>10485760</value>  
        </property>  
        <!-- 请求的编码格式，必须和jSP的pageEncoding属性一致，以便正确读取表单的内容，默认为ISO-8859-1 -->
        <property name="defaultEncoding">
            <value>UTF-8</value>
        </property>
    </bean>
```
 ## 下载文件
 > SpringMVC提供了一个ResponseEntity类型，使用它可以很方便地定义返回的HttpHeaders和HttpStatus
 
 - 具体代码可见ssmTest
