swift 命令及使用方法
============

2013年7月  


使用swift有2种命令方式  
一种是使用swift提供的API：  
>     swift blablabla  

另一种是url方式调用命令（这得益于swift采用了REST接口） 
>     curl blablabla    
  *这里的curl是一个linux下传送url的工具，也可以使用st等其他工具*

**为简化后面的内容，下文以swift方式代表API调用，curl方式代表url调用**

curl方式
--------

### curl命令的口令（token）
对于curl方式，所有的命令都必须提供认证口令（auth token）  

可以通过以下命令获取口令（token）：  
>     curl -v -H 'X-Storage-User: < your-account-name >:< your-user-name >' -H 'X-Storage-Pass: < your-password >' http://< your-ip-address >:< your-port >/auth/v1.0  

这是官方文档给出的例子，这里account是test，user是tester，password是testing：
>     curl -v -H 'X-Storage-User: test:tester' -H 'X-Storage-Pass: testing' http://127.0.0.1:8080/auth/v1.0  
   **有些文档说X-Storage-User:和< your-account-name >之间（同样还有X-Storage-Pass:和< your-password >之间）的空格不能省略，否则报错。但是本人验证后发现即使没有空格也可以正常使用**  

PS: 如果没有提供token则会报这个错误  
> This server could not verify that you are authorized to access the document you requested.  

举个例子，获取一个token（account、user、password都是admin）：  
>      curl -v -H 'X-Storage-User: admin:admin' -H 'X-Storage-Pass: admin' http://127.0.0.1:8080/auth/v1.0  

这个命令将生成一份account为admin，用户为admin，密码为admin的认证口令。系统返回结果如下：  

     * About to connect() to 127.0.0.1 port 8080 (#0)
     * Trying 127.0.0.1... connected
     * Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
     > GET /auth/v1.0 HTTP/1.1
     > User-Agent: curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.14.0.0 zlib/1.2.3 libidn/1.18 libssh2/1.4.2
     > Host: 127.0.0.1:8080
     > Accept: */*
     > X-Storage-User: admin:admin
     > X-Storage-Pass: admin
     > 
     < HTTP/1.1 200 OK
     < X-Storage-Url: http://127.0.0.1:8080/v1/AUTH_admin
     < X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65
     < Content-Type: text/html; charset=UTF-8
     < X-Storage-Token: AUTH_tk5a69111926e34d3da0864f144b167c65
     < Content-Length: 0
     < Date: Fri, 05 Jul 2013 01:52:02 GMT
     < 
     * Connection #0 to host 127.0.0.1 left intact
     * Closing connection #0  

注意结果中的X-Storage-Url和X-Auth-Token，curl命令需要这两个值。这里二者分别为`http://127.0.0.1:8080/v1/AUTH_admin`和`AUTH_tk5a69111926e34d3da0864f144b167c65`  

PS: X-Storage-Url的值是有规律的，可以直接推算出来所以不用专门记录，只需要把X-Auth-Token记录下来就行了  
PS: 如果忘了X-Auth-Token的值，那也不必担心，只需要再执行一遍上面这条命令就行了，只要参数一样，X-Auth-Token不会变。因此X-Auth-Token也不用记录  
PS: 你可能也注意到了还有一个token，叫X-Storage-Token，而且X-Auth-Token和X-Storage-Token这两个值时一样的  
  

### curl的常用命令格式

curl命令的通用格式如下：  
>     curl [-X <command>] [-T <file>] -H 'X-Auth-Token: <your-auth-token>' <url>  

+ -X代表request command，有PUT、GET等  
+ -H代表request header，也就是之后单引号包围的部分  
+ -T代表upload file，当向swift上传文件的时候需要带这个参数  
+ <url>代表发送命令的目标地址，通常是X-Storage-Url地址加上一些后缀  
+ curl还支持-o参数，代表另存为的意思，不过这个参数和swift无关，只是curl的功能，就不介绍了  

看看下面的例子就很好理解了  


### 一些例子

**注意：下面例子中的token和url在不同的机器上会有变化** 

创建mycontainer容器    
>     curl -X PUT -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer  

列出所有容器  
>     curl -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/  

上传file1文件到mycontainer容器    
>     curl -X PUT -T ./file1 -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1.0/AUTH_admin/mycontainer  
>     注意：这里有点问题，我用curl命令后会返回Accepted. The request is accepted for processing.但是list的时候显示未上传。后来发现只要出现这句话就代表你执行的命令啥用也没有。。。

列出mycontainer容器内的文件    
>     curl -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer/  

列出admin容器内所有以fi开头的文件    
>     curl -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer?prefix=fi  

从mycontainer容器中“下载”file1文件    
>     curl -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer/file1  

PS：这个命令实际上并不是下载file1文件，而是将该文件标准输出到屏幕上，相当于cat命令。如果对一个二进制文件执行这个命令，得到的就是一堆乱码。可以配合管道或重定向实现下载功能。    

下载file1文件并且另存为localfile1    
>     curl -o localfile1 -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer/file1  

PS：这才是真正的下载命令，其实是curl的功能，因此和上一条命令本质上一样   

删除mycontainer容器  
>     curl -X DELETE -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer  

从mycontainer容器中删除file1文件    
>     curl -X DELETE -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer/file1  

获取admin账户的metadata  
>     curl -v -X HEAD -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/  

获取container容器的metadata    
>     curl -v -X HEAD -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/mycontainer/  

PS：这个命令如果不带-v就什么也返回不了  

可以要求swift以xml格式或json格式返回结果，例如    
>     curl -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/?format=json  
>     curl -H 'X-Auth-Token: AUTH_tk5a69111926e34d3da0864f144b167c65' http://127.0.0.1:8080/v1/AUTH_admin/?format=xml  


swift方式
---------

swift命令方式，就是调用swift封装好的API。官方文档说可以通过swift --help来学习，强烈建议看一遍，但只看help肯定不够。。。  


swift API具有6种行为，和curl类似：  

  - stat 获取account、container或object的基本信息  
  - list 获取account里有多少个container，container中包含多少个object信息  
  - upload 上传文件  
  - post 更新account、container或object的metadata  
  - download 下载文件  
  - delete 删除container或object  

### swift方式常用的命令格式  

>     swift -A <your-auth-url> -U <your-account-name>:<your-user-name> -K <your-password> <command>  

例如，沿用之前讲解curl方式时后的用户，如果我想查看该用户的使用情况，可以用以下命令：  
>     swift -A http://127.0.0.1:8080/auth/v1.0/ -U admin:admin -K admin stat

还是继续通过例子学习吧

### 一些例子

**注意：下面例子中的token和url在不同的机器上会有变化** 

创建mycontainer容器    
>     没有对应API

列出所有容器  
>     没有对应API，类似的命令是列出某个用户的状态  
>     swift -A http://127.0.0.1:8080/auth/v1.0 -U admin:admin -K admin stat admin

上传file1文件到mycontainer容器    
>     swift -A http://127.0.0.1:8080/auth/v1.0/ -U admin:admin -K admin upload mycontainer ./file1  

PS：上传的文件会包含路径名，如果包含了路径，那么下载的时候也会在当前目录下生成相应的路径文件夹。。。  

列出mycontainer容器内的文件    
>     swift -A http://127.0.0.1:8080/auth/v1.0/ -U admin:admin -K admin list mycontainer  

列出admin容器内所有以fi开头的文件    
>     swift没有对应API

下载file1文件  
>     swift -A http://127.0.0.1:8080/auth/v1.0/ -U admin:admin -K admin download mycontainer file1  
PS: 这条命令会把文件下载到当前路径下，当然也支持-o选项  

删除mycontainer容器  
>     swift没有对应命令  

从mycontainer容器中删除file1文件    
>     swift -A http://127.0.0.1:8080/auth/v1.0/ -U admin:admin -K admin delete mycontainer file1  

获取admin账户的metadata  
>     swift没有对应API  

获取container容器的metadata    
>     swift没有对应API  

以xml格式或json格式返回结果，例如    
>     swift没有对应API


curl方式和swift方式对比
------------------------
 
相比于curl方式，swift命令方式有以下好处：  

  - 不需要每次都输入冗长的token了。
  - swift API效率更高。curl命令每次都会新建http连接（即一个http连接仅包含一个命令），而swift API经过优化（即可以实现一个http连接包括多个命令）

但是，swift命令的功能有限，例如：  

  - 不能创建account、container  
  - 不能删除container  
  - 不能格式化输出成xml或json格式文件  
  - 
  
by 李舜阳(southern9cross@gmail.com)
