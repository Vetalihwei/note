##  1.安装

<details>
<summary> </summary>

- 官网获取安装包解压后进入文件，找到configure文件`./configure`执行
- configure需要很多依赖，下面是部分可能所需:
  - `yum install -y pcre pcre-devel`
  - `yum install -y zlib zlib-devel`
  - `yum install -y gcc`
  - `yum install -y GeoIP-devel.x86_64  gd gd-devel.x86_64`
- 检查完依赖后执行`make install`
- 创建用户，不创建可能报错`nginx: [emerg] getpwnam("nginx") failed`  
  `useradd nginx`
- 进入安装好的目录`/usr/local/nginx/sbin`,可发现目录下有nginx文件
  ```
  ./nginx 启动
  ./nginx -s stop 快速停止
  ./nginx -s quit 优雅关闭，在退出钱完成已经接收的连接请求
  ./nginx -s reload 重新加载配置
  ```
</details>

---

## 2.配置

<details>
<summary> </summary>
`/nginx/conf/nginx.conf`

### 配置虚拟主机
```
server {
    #端口
    listen       80;
    #域名、主机名
    server_name  localhost;

    location / {
        #根目录
        root   html;
        index  index.html index.htm;
    }
    #错误时显示的网页
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```


</details>

---

##  3.配置虚拟主机

<details>
<summary> </summary>

`/nginx/conf/nginx.conf`下

### 简单案例

```
server {
    #端口
    listen       80;
    #域名、主机名
    server_name  localhost;

    location / {
        #根目录
        root   html;
        index  index.html index.htm;
    }
    #错误时显示的网页
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

### ServerName匹配规则
- 同一servername匹配多个域名
  - 完整匹配
    - 可以在server_name后直接写多个域名  
      `server_name  localhost www.lsls.com;`
  - 通配符匹配
    - 利用通配符`*`,conf中谁先定义优先匹配谁
  - 正则匹配
    - 支持正则表示式



</details>

---

## 4.反向代理

<details>
<summary> </summary>

### 概念
![](/img/Nginx/reverse-proxy.png)
[知乎大佬介绍代理模式](https://zhuanlan.zhihu.com/p/464965616)

**概括**  
- 在访问一个网站过程中，浏览器需要通过代理服务器根据域名查到对应ip才能访问网站，而浏览器不知道哪里有浏览器
- 对于正向代理，相当于要配置代理服务器
- 对于反向代理，相当于不需要配置代理服务器

**作用**
- 作为内容服务器的替身，真实的web服务器被包含，在外网只能看到反向代理服务器，但反向代理服务器上没有真实数据，可以保护web服务器的资源安全
- 作为内容服务器的负载均衡器，可以在一个组织内使用多个代理服务器来平衡各Web服务器间的网络负载，提高网络访问效率

### 反向代理简单模拟

#### 利用docker创建3个容器
- 创建挂载目录
```
mkdir -p /root/nginx/nginx1/conf \ &
mkdir -p /root/nginx/nginx1/log \ &
mkdir -p /root/nginx/nginx1/html \ &
mkdir -p /root/nginx/nginx2/conf \ &
mkdir -p /root/nginx/nginx2/log \ &
mkdir -p /root/nginx/nginx2/html \ &
mkdir -p /root/nginx/nginx3/conf \ &
mkdir -p /root/nginx/nginx3/log \ &
mkdir -p /root/nginx/nginx3/html
```
- 将nginx.conf与html文件复制到对应位置,记得修改conf中的html root目录为容器内目录`/usr/share/nginx/html`以及监听端口
```
cp /usr/local/nginx/conf/nginx.conf /root/nginx/nginx1/conf/nginx.conf / &
cp -r /usr/local/nginx/html /root/nginx/nginx1/ / &
cp /usr/local/nginx/conf/nginx.conf /root/nginx/nginx2/conf/nginx.conf / &
cp -r /usr/local/nginx/html /root/nginx/nginx2/ / &
cp /usr/local/nginx/conf/nginx.conf /root/nginx/nginx3/conf/nginx.conf / &
cp -r /usr/local/nginx/html /root/nginx/nginx3/
```
- 可用适当修改index.html以区分3个服务器的区别
- 启动容器
```
docker run -d \
-p 8081:8081 \
--name nginx1 \
-v /root/nginx/nginx1/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /root/nginx/nginx1/log:/var/log/nginx \
-v /root/nginx/nginx1/html:/usr/share/nginx/html \
--network host \
nginx

```

#### 代理设置
- 修改nginx.conf location部分(以修改nginx1为例)
```
location / {
    proxy_pass http://www.baidu.com; #代理地址,注意分号
    # root   /usr/share/nginx/html; #因为会直接跳转到代理地址，所以不需要根目录html
    #index  index.html index.htm;
}

```
- 访问ip:8081 我们会发现跳转到了百度
- proxy_pass 修改为其他服务器如nginx2,跳转时我们会发现浏览器地址栏地址是nginx1的，但跳转到了nginx2页面

</details>

---

## 5.负载均衡

<details>
<summary> </summary>

### 概念
- 负载均衡实际上就是将用户请求，通过某种算法，分发到集群中的节点
- 负载均衡的目标是尽力将网络流量平均分发到多个服务器上，以提高系统整体的响应速度和可用性
- 主要作用
  - 高并发：负载均衡通过算法调整负载，尽力均匀分配应用集群中各节点工作量，来提高应用集群的并发处理
  - 伸缩性：添加或减少服务器数量，然后由负载均衡进行分发控制。这使得应用集群具有伸缩性
  - 高可用：负载均衡器可以监控候选服务器，当服务器不可用时，自动跳过，将请求分发给可用的服务器。这使得应用集群具备高可用的特性
  - 安全防护：有些负载均衡软件或硬件提供了安全性功能，如：黑白名单处理、防火墙，防 DDos 攻击等

### 配置
- 继使用4中搭建的3个服务器
- 以nginx1来做负载均衡器，修改nginx.conf
  ```yml
  events {
      worker_connections  1024;
  }
  http {
      include       mime.types;
      default_type  application/octet-stream;
      sendfile        on;
      keepalive_timeout  65;
      upstream httpds{ #定义代理地址组
        server 192.168.52.129:8082;
        server 192.168.52.129:8083;
      }
      server {
          listen       0.0.0.0:8081;
          server_name  localhost;
          location / {
              proxy_pass http://httpds; #代理到代理组
          }
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
      }
  }
  ```
- 访问192.168.52.129:8081可以发现轮流访问8082,8083(轮询)

### 负载均衡策略

#### 轮询
- 默认情况使用轮询策略，逐一转发，这种方式适用于无状态请求  

**权重**  
- 可通过配置weight来调节服务器请求分配规则,weight越大服务器分到的请求越多
  ```
  upstream httpds{ #定义代理地址组
    server 192.168.52.129:8082 weight=4;
    server 192.168.52.129:8083 weight=2;
  }
  ```
- down 用于关闭服务器
  ```
  server 192.168.52.129:8082 weight=4 down;
  ```
- backup 设定当前服务器为备用服务器，当无服务器可用时才启用
  ```
  server 192.168.52.129:8082 weight=4 backup;
  ```
#### 其他少用策略
- ip_hash
  - 根据客户端的ip地址转发同一台服务器，可用保持会话
- least_conn
  - 最少连接访问
- fair
  - 根据后端服务器响应时间转发请求
  - 但会受网络延迟影响，有流量倾斜风险
- url_hash
  - 定向流量转发，适用于访问固定资源(不在同一服务器)

</details>


---

## 6.动静分离

<details>
<summary> </summary>

### 6.1 原理
- 将静态的资源如js、css、img放在Nginx服务器，动态请发送到后端服务器

### 6.2 配置动静分离
- 需要将静态资源放在nginx1的html中，但根据上文反向代理的知识，我们知道此时访问nginx1访问不到其下的html，会跳转，故需要添加多个location
```yml
server {
    listen       0.0.0.0:8081;
    server_name  localhost;

    location / {
        proxy_pass http://httpds;
    }
    # location /css { 简单配置静态资源路径，但一个location只能配一个
    #     root /usr/share/nginx/html;
    #     index index.html index.htm;
    # }
    location ~*/(js|img|css) { #通过正则表达式解决上文问题
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}

```

</details>

---

## 7.URLRetrite伪静态配置

<details>
<summary> </summary>

- proxy_pass上添加rewrite，隐藏真实地址，在转发前重写地址,支持正则表达式
  ```
  rewrite ^/([0-9]+).html$  /index.jsp?pageNum=$1 break;
  proxy_pass xxx;
  ```
- 实现访问number.html跳转到/index.jsp?pageNum=number
- `rewrite <regex> <replacement> [flag]`
- flag:
  - last：本条规则匹配完成后，继续向下匹配新的location URI规则
  - break：本条规则匹配完后终止
  - redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址
  - permanent：返回301永久重定向，浏览器地址会显示跳转后的URL地址


</details>

---

## 8.防盗链与http的referer

<details>
<summary> </summary>

- 防盗链目的是存在nginx服务器上的静态资源只有我们自己使用
- 浏览器在二次请求后会在请求头上添加referer，表示请求的来源
- 通过判断referer是否为nginx服务器地址可以实现简单防盗链

**实现**  
- 修改conf文件中location部分
  ```yml
  location ~*/(js|img|css) { 
      valid_referers 192.168.52.129:8081;
      if($invalid_referer){ #检测，无效的referer会返回403
        return 403;
      }
      root /usr/share/nginx/html;
      index index.html index.htm;
  }
  ```
- `valid_referers none | blocked | server_names | strings ..`
  - none，检测referer头域不存在的情况
  - blocked，检测referer头域的值被防火墙或者代理服务器删除或伪装的情况
  - server_names，设置一个或多个URL，检测referer头域的值是否是这些URL中的一个

</details>

---

## 9.高可用

<details>
<summary> </summary>

- 解决nginx宕机导致服务不可用问题
- 当一台nginx服务器宕机可用自动切换到另一台
- 通过keepalived实现

**原理** 
- keepalived通过生成一个虚拟ip，在nginx前部浮动，用户访问的是虚拟ip，再有虚拟ip指向nginx服务器
- 每台nginx服务器上都存在keepalived，他们之间能相互通讯，当主机宕机时可通知备用机
- 通过脚本定时检测nginx是否出错，出错则kill当前服务器的keepalived以告知其他keepalived
- 通过这种方式实现的高可用，不需要额外部署机器

### 9.1 keepalived
- 安装  
```
yum -y install keepalived
#docker容器内安装
#进入容器
docker exec -it nginx1 bash
apt-get update
apt-get install keepalived
apt-get install vim
```
- 配置文件地址
 ```
 vim /etc/keepalived/keepalived.conf
 ```
- 修改配置文件，主要修改vrrp_instance,router_id
  ![](/img/Nginx/vip.png)
- router_id：服务明早
- intefave：网卡名称，`ip adr` 查询网卡名称
- authenication：认证配置，同组保持一致即可
- virtual_ipaddress：虚拟ip，外部访问的ip


</details>

---

## 10.加密相关

<details>
<summary> </summary>

- http协议的缺点：明文，无状态
- 为了解决数据保密问题，需要对数据进行加密，以密文形式传输  

**对称加密**  
- 双方使用相同的加密算法，互相加密解密
- 缺点：密钥管理复杂、安全性风险高、扩展性差、密钥分发困难和无法保证数据完整性和身份验证

**非对称加密**  
- 加密解密不一致，一个公钥，服务端存有一个私钥
- 客户端访问服务端时，通过下载的公钥+算法进行加密，服务端通过私钥+算法进行解密
- 服务端返回数据时通过私钥+算法进行加密，客户端通过公钥+算法进行解密
- 公钥加密，公钥解不开
- 但存在拦截者伪装服务端情况，这时就引出CA机构，通过CA证书机制来保证客户端接收到的数据是来自服务端
![来源于尚硅谷](/img/Nginx/Asymmetric_encryption.png)


**CA机构**
- 在服务端，CA机构会通过服务端提供的资料，认证公钥，通过ca的私钥+算法生成一个证书，要求放在该域名下的某个目录下，ca机构会去访问这个证书，若访问不到说明该服务端不是正确的服务端
- 在客户端，会接收到证书，证书只能用内置的ca公钥进行解密
- 在证书传输给客户端过程中，存在拦截者拦截证书并解密获取明文可能，但修改后的数据会无法被客户端解密，因为拦截者不知道ca私钥，无法加密伪装服务端
- 这也是https原理之一
![来源于尚硅谷](/img/Nginx/CA.png)

> 图片来源于尚硅谷
</details>

---

## 11.

<details>
<summary> </summary>




</details>

---

## 12.

<details>
<summary> </summary>




</details>

---

## 10.

<details>
<summary> </summary>




</details>