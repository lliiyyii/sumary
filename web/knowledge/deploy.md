## windows构建java-springboot-mysql-activiti-redis项目部署到服务器的问题

> ## 写在前面的话
> 这几天老师要谈一个项目，就要用一个网上的项目做简单更改做一个demo，在构建系统，部署到都服务器，修改项目上都遇到很多问题，我觉得好些问题都是对linux系统和其他东西不了解导致的，所以总结一下遇到的bug，对以后应该很用


### 连接redis的为问题
> 这个问题只改了bug，没仔细看怎么回事，就说说遇到的问题和处理方法

#### 问题描述
开启的redis服务和项目里面的配置都是没有密码，但是一直报错，解决方法就是都加了密码

#### 错误提示
```
(error) ERR Client sent AUTH, but no password is set
```

#### 修改方法
1.在redis客户端设置密码123456
```
  CONFIG SET requirepass 123456
```
2.查看修改是否成功
```
 auth 123456
```
3.在配置文件application.yml也修改密码
```yml
  redis:
        database: 0
        hostName: 127.0.0.1
        port: 6379
        password: 123456 # 密码（默认为空）
        timeout: 6000  # 连接超时时长（毫秒）
        pool:
            max-active: 1000  # 连接池最大连接数（使用负值表示没有限制）
            max-wait: -1      # 连接池最大阻塞等待时间（使用负值表示没有限制）
            max-idle: 10      # 连接池中的最大空闲连接
            min-idle: 5       # 连接池中的最小空闲连接
```


### 项目部署到服务器后数据库处理的各种问题

#### 问题描述
在windows中没有任何问题，在服务器上导入或者复制数据表以后服务器还一直显示找不到数据表，后面发现刷新时增加了一些同样名字的但是全部变为大写的数据表，后面发现原来windows上mysql是不区分大小写的，但是Linux上严格区分大小写，ps.mac上也不区分大小写。解决时使用网上说的更改命令都对的，但是说的没前没尾，浪费很多时间，下面是正确的解决方法

#### 解决方法
1.更改/etc/mysql/my.cnf，在最后添加下面代码
```cnf
[mysqld]
lower_case_table_names=1
```
修改后我的my.cnf文件变为
```cnf
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
# 
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

#
# * IMPORTANT: Additional settings that can override those from this file!
#   The files must end with '.cnf', otherwise they'll be ignored.
#

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
[mysqld]
lower_case_table_names=1
                           
```
2.重启数据库
```
sudo /etc/init.d/mysql restart
```

3.查看大小写情况
```
mysql -u root -p
```
```
mysql> show variables like '%case_table%';
```
显示为1则修改成功

这样就不区分大小写了，啦啦啦


### 部署到服务器上找不到controller之类的各种东西
> 我觉得有一种可能的原因是扫描不到包，不过改了也没有解决问题，还是写一下方法
> 1.注解的两种方式
> @ComponentScan
>  @Controller
> 或者
> @RestController
> 2.在application.java上加上扫描包注解
> @ComponentScan({"com.hxy.modules.activiti.org.activiti", "com.hxy"})

下面是最终的解决方法

#### 问题描述
其实是这个项目需要用到计算机的计算机名，例如我的，本机IP127.0.0.1，也叫localhost，计算机名是DESKTOP-ES2RSHS23DKDL56这样的，这个程序会自动识别，但是我的linux服务器不知道它的主机名，就是```root@iW0788ypdf4lfaZ:/etc/mysql# ```里面那一串，也可以用命令```hostname```得到，解决方法就是把linux的主机名添加到配置文件里就可以了

#### 解决方法
1.打开文件/etc/hosts
2.可以看到第一行是
```
127.0.0.1       localhost 
```
3. 把你的主机名加进去,修改为
```
127.0.0.1      localhost iW0788ypdf4lfaZ
```
4. 重新运行项目就可以了，啦啦啦啦


### 引入activiti后无法显示以及无法保存的问题
> 这两个问题应该是这个项目开发者的错误，没有什么系统的问题，就说说我的问题和解决方法啦

#### activiti无法显示问题描述
> activiti是一个画流程图的工具，感觉很好用，不过没有使用过，就看了一下教程，疯狂看了教程代码和网络日志。顺便记一下看的时候很有用的教程https://www.jianshu.com/p/cf766a713a86 提供的项目代码也能用
在应该出现流程图画板的地方什么都没有出现，chrome的network显示```http://localhost:8080/myFrame/model/1/json 404```

#### activiti无法显示解决方法
1.加入必要的静态工具包，详见链接
2.不知道为什么，这个项目的activiti静态包的设置```editor-app/app-cfg.js```里项目路径就是不能和项目路径一样，就直接去了，显示成功
3.具体修改
editor-app/app-cfg.js
```javascript
ACTIVITI.CONFIG = {
    'contextRoot' : '/service',
};
```
application.yml中注释项目路径的配置
```yml
#    context-path: /
```

#### 无法保存的问题描述
点击activiti的保存以后没法保存，检查activiti中的保存java类ModelSaveRestResource没有问题

#### 无法保存问题的错误提示
catch报错日志
```
org.apache.batik.transcoder.TranscoderException: null 
Enclosed Exception: 
Content is not allowed in 
```
error报错日志
 ```
 Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.activiti.engine.ActivitiException: Error saving model] with root cause
 ```
 前端错误500
 
 #### 解决方法
 1.activiti中有一个文件处理的java文件 JsonpCallbackFilter
 
```java
@WebFilter("/service/*")
public class JsonpCallbackFilter implements Filter {

    private static Logger log = LoggerFactory.getLogger(JsonpCallbackFilter.class);

    public void init(FilterConfig fConfig) throws ServletException {
    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        Map<String, String[]> parms = httpRequest.getParameterMap();

        if (parms.containsKey("callback")) {
            if (log.isDebugEnabled())
                log.debug("Wrapping response with JSONP callback '" + parms.get("callback")[0] + "'");

            OutputStream out = httpResponse.getOutputStream();

            GenericResponseWrapper wrapper = new GenericResponseWrapper(httpResponse);

            chain.doFilter(request, wrapper);

            //handles the content-size truncation
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            outputStream.write(new String(parms.get("callback")[0] + "(").getBytes());
            outputStream.write(wrapper.getData());
            outputStream.write(new String(");").getBytes());
            byte jsonpResponse[] = outputStream.toByteArray();

            wrapper.setContentType("text/javascript;charset=UTF-8");
            wrapper.setContentLength(jsonpResponse.length);

            out.write(jsonpResponse);

            out.close();

        } else {
            chain.doFilter(request, response);
        }
    }

    public void destroy() {
    }
 }
 ```
项目中又有一个文件处理的文件XssFilter并进行了bean的配置
```java
public class XssFilter implements Filter {

	@Override
	public void init(FilterConfig config) throws ServletException {
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
		XssHttpServletRequestWrapper xssRequest = new XssHttpServletRequestWrapper(
				(HttpServletRequest) request);
		chain.doFilter(xssRequest, response);
	}

	@Override
	public void destroy() {
	}
}
```
```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean shiroFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new DelegatingFilterProxy("shiroFilter"));
        //该值缺省为false，表示生命周期由SpringApplicationContext管理，设置为true则表示由ServletContainer管理
        registration.addInitParameter("targetFilterLifecycle", "true");
        registration.setEnabled(true);
        registration.setOrder(Integer.MAX_VALUE - 1);
        registration.addUrlPatterns("/*");
        return registration;
    }

    @Bean
    public FilterRegistrationBean xssFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setDispatcherTypes(DispatcherType.REQUEST);
        registration.setFilter(new XssFilter());
        registration.addUrlPatterns("/*");
        registration.setName("xssFilter");
        registration.setOrder(Integer.MAX_VALUE);
        return registration;
    }

    @Bean
    public JsonpCallbackFilter filter(){
        return new JsonpCallbackFilter();
    }

}
```
2.我暂时直接删去了xssFilter的内容并注释了FilterConfig.java中的xssFilterRegistration函数，可以保存啦

3.方法是在太暴力，没有研究两个函数的作用，后面项目处理文件的bug留坑


### xml方式使用mybatis的错误No typehandler found for property XXX
#### 错误描述
   >这个部分是项目新加的功能 
   前面是各种包的错误，最后都是一样，指向一个xml文件的错误，其实很简单，只是我太智障了，我用谷歌翻译后是“找不到属性文件的类型处理程序”结果，其实file是我的变量名，囧，所以意识是file变量的java类型和数据库类型不匹配

#### 报错信息
```
No typehandler found for property file
```
其实是
```
...... No typehandler found for property XXX
```
#### 解决方法
将file变量的数据库类型和java类型对应，我的是把原来的MultipleFile改成String，数据库依然是varchar，就可以了。


### ajax使用时报错
#### 问题描述
使用ajax上传文件时报错

#### 报错信息
```
Uncaught TypeError: Illegal invocation
```

#### 解决方法
1.查看data里的数据是否真的有效，这里有两种file上传的方法
```javascript
 var  formData = new FormData();
formData.append("file",$("#file")[0]);
```
```javascript
 var  formData = new FormData();
formData.append("file",$("#file")[0].files[0]);
```
2.在ajax的属性里添加```processData: false,```

3.我最后的请求为****后面一个问题这个代码有问题后又有更改*****
```javascript
  $("#btnFile").click(function () {
        var  formData = new FormData();
        formData.append("file",$("#file")[0]);
        $.ajax({
            cache: true,
            type: "POST",
            dataType: 'json',
            contentType: "form-data",
            processData: false,
            async: false,
            url: "${webRoot}/demo/approved/upload",
            data: formData,
            beforeSend:function(){
                console.log("正在进行，请稍候");
            },

            success: function(responseStr) {
                if(responseStr.status===0){
                    console.log("成功"+responseStr);
                }else{
                    console.log("失败");
                }
            },
            error: alert("Error")
        });
    })
 ```
    
    
### 解决上传文件时400，required request part file is not present的问题
#### 错误描述
spring boot使用MultipartFile类型做上传文件是，方法同原来写的spring boot中的方法一样，但是总是出问题，解决了四天还没有解决，现在解决了。

#### 报错信息
```
{
    "timestamp": 1517142211343,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.web.multipart.support.MissingServletRequestPartException",
    "message": "Required request part 'file' is not present",
    "path": "/demo/approved/upload"
}
```

#### 解决方法
1.应该是配置的问题，所以增加配置，尝试多个以后发现这个有用
```java
@Bean
public CommonsMultipartResolver multipartResolver() {
	CommonsMultipartResolver multipart = new CommonsMultipartResolver();
	multipart.setMaxUploadSize(3 * 1024 * 1024);
	return multipart;
}

	@Bean
	@Order(0)
	public MultipartFilter multipartFilter() {
		MultipartFilter multipartFilter = new MultipartFilter();
		multipartFilter.setMultipartResolverBeanName("multipartResolver");
		return multipartFilter;
	}
```
2.在返回的函数名前添加```@ResponseBody```，我更改后的上传文件的函数变为
```java
 @PostMapping(value = "upload",consumes = "multipart/form-data")
    @RequiresPermissions("act:model:all")
    public @ResponseBody BaseApi myupload (HttpServletRequest request,
                           @RequestParam(value = "myfile") MultipartFile file) throws Exception{

//        log.info("进入upload"+tag);
        try{
            String filename = file.getOriginalFilename();
            String path = request.getServletContext().getRealPath("/files/");
            log.info("文件名字是 "+filename+" 文件路径是 "+path);
            File filepath = new File(path,filename);
            //判断路径是否存在，如果不存在就创建一个
            if (!filepath.getParentFile().exists()) {
                filepath.getParentFile().mkdirs();
            }
            //将上传文件保存到一个目标文件当中
            file.transferTo(new File(path + File.separator + filename));
            //返回文件的路径
            JSONObject jsonObject = new JSONObject();

            jsonObject.put("path",path+File.separator+filename);
            return new BaseApi("ok",0,jsonObject);
        }catch (Exception e){
            log.info("出错");
            e.printStackTrace();
            return new BaseApi("error",1,null);
        }

    }
```
3.不知道有没有用，但是我增加了一个包
```yml
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
```

4.贴上解决这个问题的网址：https://stackoverflow.com/questions/43936372/upload-file-springboot-required-request-part-file-is-not-present

#### 这个问题前端失败的问题已经解决了
##### 问题描述
报错和后端的一样，原因是前端传文件的时候取的值不是文件
##### 问题解决
1.更改js就可以了，贴上代码
```javascript
 var fd = new FormData();
        fd.append( "myfile", $("#myfile")[0].files[0]);

        $.ajax({
            type: "POST",
            url: "${webRoot}/demo/approved/upload",
            data: fd,
            contentType: false,
            processData: false,
            cache: false,
            /*beforeSend: function(xhr, settings) {
                xhr.setRequestHeader("Content-Type", "multipart/form-data;boundary=gc0p4Jq0M2Yt08jU534c0p");
                settings.data = {name: "file", file: inputElement.files[0]};
            },*/
            success: function (result) {
                if ( result.reseponseInfo == "SUCCESS" ) {

                } else {

                }
            },
            error: function (result) {
                console.log(result.responseText);
            }
        });
```


> ### 写在最后的话
> 老师还要改前端的字，好像好多字是动态生成的，解决了继续写，呀呀呀















