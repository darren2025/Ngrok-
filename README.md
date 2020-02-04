# Ngrok一件安装脚本-使用说明

## 安装(先看注意第一条与第二条)

```shell
wget https://raw.githubusercontent.com/darren2025/Ngrok-/master/ngrok.sh
chmod 777 ngrok.sh
sh ngrok.sh  #装2-5
git version  #查看是否安装成功
go version   #查看是都安装成功
```

## 注意:

### 一、建议不要用全新安装,从2-5自己手工一个个安装,如果全新安装如果其中一个安装失败后期报错你们不知道

### 二、安装第五个的时候,请先解析一个域名到当前的存在公网的域名到服务器,可以二级也可以三级,要泛解析，第五的时候会让你输入域名，就用刚才解析的域名

#### 如图：

![1580811799248](/src/1580811799248.png)



### 三、如果后期域名换过需要重新安装5，因为需要重新生成签名

### 四、6 生成客户端的时候，如果出现ERROR: Cannot find /root/go1.4/bin/go. 请不用管 ，生成好的，文件会存在/usr/local/ngrok/bin下，生成后通过sftp或者sz下载到本地即可

### 五、卸载的时候，是卸载全部，包括git、go、ngrok等！

### 六、启动服务的时候，如果是有nginx的需要配置一下，不然会跟nginx抢占80端口，不然就共存，不然停止ngin,wo 这里启动的时候配置为800,后期通过nginx反代过去

#### 附上nginx共存的配置

```nginx
server
    {
        listen 80;
        server_name *.xxxx.com ;  #域名
        index index.html ;
        root  /home/wwwroot/xxx.com;
        location / {
            proxy_pass http://ngrok.xxxx.com:800; #此处二级域名可以随意填写
            proxy_set_header Host $host:800; #这个是重点，$host 指的是与server_name相同的域名
            proxy_redirect off;
            client_max_body_size 10m;
            client_body_buffer_size 128k;
            proxy_connect_timeout 90;
            proxy_read_timeout 90;
            proxy_buffer_size 4k;
            proxy_buffers 6 128k;
            proxy_busy_buffers_size 256k;
            proxy_temp_file_write_size 256k;

        }
 
#解决配置反向代理后js css文件无法加载问题
      location ~ .*\.(js|css)$ {            
            proxy_pass http://ngrok.xxxx.com:800; #此处二级域名可以随意填写
            proxy_set_header Host $host:800;#这个是重点，$host 指的是与server_name相同的域名
         }


}
```

### 七、没有启动成功或者没有隧道成功或没有安装成功，请看如下

###### 1、首先端口是否开放以及转发4443以及80端口

###### 2、域名的二级域名和三级域名是否都转发到服务器的公网ip地址上

###### 比如ngrok.321dz.com和*.ngrok.321dz.com都需要转发

###### 3、证书生成后是否替换成功，证书生成时，域名填写是否正确？

###### 4、启动服务端时的命令是否写错，导致域名错误

###### 5、客户端启动后，如果没连接成功，不要着急。先查看log日志，查看是证书错误还是说是连接不上服务端。上面的错误一般都包含了，因此我们在搭建的时候一定要小心，一步错，步步错。小心使得万年船。

## 使用流程

**下载脚本**->**安装2-5**->**生成客户端,将客户端覆盖client,如果是win64,就不用生成了**->**启动服务**->**查看配置文件复制配置文件东西到ngrok.cfg,即可**->**运行客户端start即可**



# 以下内容需要就用

## 设置开机自启

在ngrok目录下创建 start.sh 文件

```shell
cd /usr/local/ngrok
vim start.sh  ##复制下面内容到里面,保存退出
/usr/local/ngrok/bin/ngrokd -domain="xxx.com" -httpAddr=":800" &
```

在/etc/rc.d/init.d/下新建ngrok文件

```shell
vim /etc/rc.d/init.d/ngrok
```

复制一下内容

```shell
#!/bin/sh  
 
ngrok_path=~/ngrok
case "$1" in
    start)
        echo "start ngrok service.."  
        sh ${ngrok_path}/start.sh
        ;;
    *)
    exit 1
    ;;
esac
```

最后执行一下东西,给权限,开启自启

```shell
chmod 755 ngrok  #修改ngrok文件权限
chkconfig --add  ngrok  #注册自启动
service ngrok start #运行ngrok，关闭远程服务器后ngrok还是在运行着
```

