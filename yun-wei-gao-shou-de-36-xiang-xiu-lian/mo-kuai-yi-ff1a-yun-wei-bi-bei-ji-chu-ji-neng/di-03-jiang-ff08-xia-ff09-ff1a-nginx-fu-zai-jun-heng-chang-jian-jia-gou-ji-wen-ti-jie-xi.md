# 第03讲（下）：Nginx 负载均衡常见架构及问题解析

## Nginx 负载均衡常见问题

那么，Nginx 负载均衡的通常配置会出现哪些问题呢？这里列出几种比较常见的问题：

* 客户端 IP 地址获取问题

* 域名携带问题

* 负载均衡导致 session 丢失问题

* 动态负载均衡问题

* 真实的 Realserver 状态检测

接下来，我们就重点讲解下 Nginx 作为负载均衡的这几个问题，值得一提的是，这五个问题中的动态负载均衡问题将在下一课时重点讲解，所以本课时就不再讲解了。

## 客户端IP地址获取问题
第一个问题是客户端 IP 地址获取问题，为什么会存在客户端 IP 地址获取问题呢？

CgpOIF5U0tqAdFH3AAFLYg4Sm0c598.png

我们来看这样一张模拟图，图中请求从用户到 Nginx，再到后端服务。我们可以看到用户的 IP 地址是 100.100.100.100，Nginx 的地址是 192.168.0.1，通过 Nginx 负载均衡到三台后端服务，它们的 IP 分别是 192.168.1.1、192.168.1.2、192.168.1.3。



那么客户端的 IP 地址为什么会无法被后端服务获取呢？原因是我们获取方式在Nginx的加入负载均衡后出现了差异，具体如下：



我们了解后端服务获取IP的方式，第一种方式是由后端服务通过 4 层 TCP 协议获取源 IP 地址，你会发现，通过这种方式，后端服务只能获取 Nginx 的 IP 地址，而无法获取到 100.100.100.100 这个 IP 地址，原因在于 Nginx 做了一层反向代理，而反向代理修改了客户的源地址和源端口的包头，所以在后端服务，通过这种方式只能获取到 Nginx 的 IP，而无法获取到用户的 IP。



另外一种方式是后端服务通过 HTTP 标准请求头信息 X-forwarded-for 获取用户IP信息，但是通过代理这一层时，有可能把 X-forwarded-for 改写或丢失用户的请求地址，这样就有可能导致 X-forwarded-for 在后端无法获取用户信息。



X-forwarded-for 头的格式，它会一层一层往后添加信息，比如第一层是客户端的 IP 地址，然后是通过代理后的代理层IP，代理层后一层一层的追加信息。X-forwarded-for需要通过 Nginx 配置添加，从而使得标准协议将以 X-forwarded-for 头为载体，把用户的 IP 地址和代理层 IP 地址添加到这个载体中，这样后端才能通过X-forwarded-for 获取到详细的 IP 地址信息。



## 解决办法



知道了问题发生的原因，我们如何来解决呢？第一种方式的解决办法是在 Nginx 负载均衡的基础上添加了一个转发到后端的标准 head 信息，把用户的 IP 信息通过 X-Forwarded-For  方式传递过去。



第二种方式是就是添加一个 X-Real 的自定义头，自定义头我们可以随意的命名，一般情况下命名为 X-Real_IP，把用户的真实四层 IP地址赋值给它，如何做到呢？



remote_addr 是一个 Nginx 的内置变量，它获取到的是 Nginx 层前端的用户 IP 地址，这个地址是一个 4 层的 IP 地址，我们来看这张图，Nginx 的 IP 地址是 192.168.0.1，它的 remote_addr 是什么呢？它就是用户的 100.100.100.100 这个地址，也就是直接将 TCP/IP 协议的用户 IP 地址（源地址）以 remote_addr 变量的方式赋值给 X-Real_IP 的自定义变量，后端直接通过 X-Real 获取 X-Real IP 信息，便可获取到用户地址。



更深一层的分析，这两种解决方式虽然解决地址获取的问题，但各有优劣，X-Forwarded-For 是通过 Nginx 中的 $proxy_add_x_forwarded_for 内置变量获取的头信息，通过 X-Forwarded-For传递到后端。在 Nginx 通过 $proxy_add_x_forwarded_for 内置变量获取的 X-Forwarded-For 信息本身可能不准确，因为前端用户可以作篡改 HTTP 的请求头，所以如果被一些恶意用户篡改了请求的 X-Forwarded 信息就会导致信息影响获取真实的用户 IP 地址信息。



第二种方式相对于第一种方式而言更加准确，因为 remote_addr 是直接获取第一层代理的用户 IP 地址，如果直接把这个地址传递给 X-Real，这样就会更加准确。但是它有什么劣势呢？如果是多级代理的话，用户如果不是直接请求到最终的代理层，而是在中间通过了 n 层带来转发过来的话，此时 remote_addr 可能获取的不是用户的信息，而是 Nginx 最近一层代理过来的 IP 地址，此时同样没有获取到真实的用户 IP 地址信息。



可以看到，这两种方式各有优劣，通常你也可以把这两种配置都加进去，然后交给后端综合分析，这就是对于客户端 IP 地址携带通常的配置方式。

## 如何解决域名携带

接下来，我们来看第二个问题，也就是 Nginx 作为负载均衡是如何把请求域名信息携带到后端的？这样的问题是什么样的场景呢？首先我们来看一张示意图。

Cgq2xl5U0uGAe2_GAAFMYE51VWw660.png

用户的 IP 同样是 100.100.100.100，Nginx 对外 IP 地址是 200.200.200.200，Nginx 本身内部的地址是 192.168.0.1，同样分发给三个后端服务。当用户请求某一 Nginx 入口域名时，会携带一个头信息，叫作 Host 头信息。如果通过域名方式，当用户请求的是 http://www.jesonc.com，它的 Host 头就是  www.jesonc.com，以这样的方式请求到代理层时，由于 Nginx 没有将头信息携带到后端，而是把配置的头信息置为空，这样就导致后端服务想要获取不到用户请求的 Host 信息，导致服务不可用。这时需要 Nginx 将用户携带的 Host 头信息真实的传递给后端。



第二种情况是假设用户请求用的 IP 地址方式（非域名），此时会发现，用户请求的 IP 地址的 Host 头信息本身就是空的，所以如果后端需要用到 Host 头信息，这就需要在 Nginx 中把 Host 头改写为一个要求的头信息，我们来看下这个场景配置：


```
http {
        …
        upstream app_servers {
                 server ip1:port1;
                 server ip2:port2;
                 server ip3:port3;
        }
        server {
         …
              location / {
     proxy_set_header Host $host ;
     proxy set_header Host www.vipumi.com; 
                      proxy_pass http://app_servers; 
              }
        ….
        }
}
```
我们通过 proxy_set_header 配置方式加入 Host 头部字段，如果用户的请求携带了域名，就把这个域名的 head 头以标准的 HTTP 方式将头信息传递到后端，然后让后端获取 Host 头信息，这样访问就不会受影响。



然后，如果用户没有携带头信息，而后端又要求指定域名，那么我们就可以在 proxy_set_header 下，将 Host 头信息指定成一个固定的域名，保证满足后台对 Host 头信息的要求，这样就可以解决域名携带问题。

## 负载均衡导致session丢失问题
CgpOIF5U0tOAeMMQAAGg2smEGW8440.png

另外一个是 session 丢失问题，我们知道，session 是用户的一个会话信息，服务端和用户端通信需要一个访问认证，以及对鉴权处理的时候，给客户端分配一串 session 信息，这个 session 信息会在客户端以 cookie 的方式承载到服务端中进行校验。如果加入 Nginx 负载均衡，会默认开启一个轮询策略。假如这样的一种场景，用户请求到 Nginx，第一次请求会分发给 App server 1，第二次请求分发给 App server 2，但用户的 session 保留在 App1 上，此时这条请求再去请求 App2 的话，由于 App2 上没有 session 信息，就导致会话丢失，用户需要重新登录。我们看到 Nginx 作为负载均衡  + Java 后台应用中遇见的一种问题，我们该怎样解决呢？

解决 session 丢失的问题，通常有以下几种做法：

* 解决方案 1：Session 保持

第一种方式是做 Session 保持，就是把负载均衡策略基于原有轮询数的基础上，改用 ip_hash、URL_hash 来解决。ip_hash 就是基于用户 IP 来做 hash，一个用户的请求统一分发到一台机器上。URL_hash 用于用户请求固定页面时，将用户请求固定到具体 后端 上，就保证了 Session 不会被丢失。

* 解决方案 2：Session 复制