# k8s笔记
## 概述
Kubernetes(k8s)是一项服务编排的技术工具，为容器化的应用提供更加强大高效且自动化的部署、规划、更新、维护等操作。

## 重点知识
+ k8s的工作原理
+ APIs的设计理念
+ Pod的设计理念
+ k8s的网络管理

## 随笔
+ 为什么需要Pod  
Pod本身属于一个逻辑概念，首先要明确的是，pod中是可以起多个容器(container)，一个容器中是可以启用多个进程。但是，官方设计模型的时候，还是以pod为最小调度单位，在官方的理念中，Pod更像是应用所存在的环境。如果k8s直接管理应用进程，对于应用产生的一些子进程或者日志输出之类的复杂度将会大幅度上升，很有可能产生应用进程结束，但是还是得由容器去一一关闭其附带的子进程这种过程，于是为了屏蔽复杂度，k8s抽象出了一个Pod模型，Pod中将运行对应的容器，一个容器对应一个应用（理想状态，也是官方推荐最佳实践），多个需要共享资源还有上下文的关系密切的容器组成一个pod（例如，应用容器和其相关的日志输出容器），而k8s以pod为维度来操作，则能够大大减少管理复杂度，如果需要关闭应用，则将其整个“环境”pod关闭即可，并且在调度的过程中，能够避免归属于同一个应用的进程，被分散到不同的节点上这种情况。
+ Tips  
1、kubectl explain可以查看yaml对应的层级  
2、kubectl port-forward可以在不通过service的情况下和pod通信  
3、kubectl get pod -L {tag} 可以查看pod对应标签的情况   
4、kubectl logs mypod --previous 可以查看关闭的前一个容器的日志  
5、退出代码137表示进程被外部信号终止， 退出代码为128+9 (SIGKILL)。
同样， 退出代码143对应于128+15 (SIGTERM)。  
6、存活探针(liveness probe)的选取一般是采用应用中的一个自定义的简单返回json信息的/health接口，而不是采用业务接口，原因大致有两个，一个是因为探针的检查是频繁的，所以探针接口最好是轻量的，否则会大大拖慢容器的性能。第二个原因是最好这个接口能排除程序内部的影响，否则一个类似于数据库连接失败的错误导致后端容器不可用，让后端容器不断重启并不会对修复错误有很好的帮助。  
7、ReplicaSet相比ReplicationController的一个重大优势，在于对于标签选择器提供更加强有力的模糊匹配以及标签管理功能。  
8、DaemonSet类型是设计出来为了能在指定node上运行特定的pod，所以它创建的pod是不受调度器控制的。它仍可将pod创建到标记为“不可调度”的节点。  
9、双横杠(--)代表着kubectl命令项的结束。在两个横杠之后的内容是指
在pod内部需要执行的命令。例如下列语句代表在kubia-7nogl这个pod中执行语句curl -s http://10.111.249.153

  kubectl exec kubia-7nogl -- curl -s http://10.111.249.153  
10、由于服务现在已暴露在外， 因此可以尝试使用网络浏览器访问它。但是会看到一些可能觉得奇怪的东西——每次浏览器都会碰到同一个pod 。此时服务的会话亲和性是否发生变化？使用kubectl explain, 可以再次检查服务的会话亲缘性是否仍然设置为None, 那么为什么不同的浏览器请求不会碰到不同的pod, 就像使用curl时那样？现在阐述为什么会这样。浏览器使用keep-alive连接， 并通过单个连接发送所有请求， 而curl每次都会打开一个新连接。服务在连接级别工作， 所以当首次打开与服务的连接时， 会选择一个随机集群， 然后将属于该连接的所有网络数据包全部发送到单个集群。即使会话亲和性设置为None, 用户也会始终使用相同的pod (直到连接关闭）。  
11、SVC中的LoadBalancer模式其实是NodePort模式的扩展，它本质上是集群上的一个服务，通过负载均衡算法，让所有访问这个服务的请求选择一个节点的pod路由过去，其实本质上还是NodePort。而，NodePort的最大一个特点就是，一旦指定了节点对外范围的port，那么集群中的所有节点都会监听住这个port。  
12、谨记：排查问题的时候，ping clusterIp是没有作用的，因为是一个虚拟的ip  
13、Ingress 控制器不会将请求转发给该服务，只用它来选择一个pod 。
大多数（即使不是全部）控制器都是这样工作的。  
14、Dockerfile中的两种指令分别定义命令与参数这两个部分：  
• ENTRYPOINT定义容器启动时被调用的可执行程序。  
• CMD指定传递给ENTRYPOINT的参数。  
尽管可以直接使用CMD指令指定镜像运行时想要执行的命令， 正确的做法依旧是借助ENTRYPOINT指令， 仅仅用CMD指定所需的默认参数。这样， 镜像可以直接运行， 无须添加任何参数。  
15、ENTRYPOINT指令提供两种形式：  
• shell形式一如 ENTRYPOINT node app.js。  
• exec形式一如 ENTRYPOINT ["node", "app. js "]。  
两者的区别在于指定的命令是否是在shell中被调用。shell进程往往是多余的， 因此通常可以直接采用exec形式的ENTRYPOINT指令。  
16、可以采用$ (VAR)语法在环境变量值中引用其他的环境变量。  
17、ConfMap 中的键名必须是一个合法的DNS子域，仅包含数字字母、破
折号、下画线以及圆点。首位的圆点符号是可选的。  
18、将ConfigMap暴露为卷可以达到配置热更新的效果， 无须重
新创建pod或者重启容器。如果挂载的是容器中的单个文件而不是完整的卷， ConfigMap更新之后对应的文件不会被更新。  
19、每个namespace都会默认的创建几个secret，一般是做pod里面安全访问kubernetes api服务所用，一般这些secret会以卷的形式挂载在pod中的/var/run/secrets/kubernetes.io路径下。当在pod里边要对外访问kubernetes api服务时，可以通过如下方式将default-token这个secret输出到环境变量TOKEN中，再利用curl调用kubernetes api时即可安全认证通过。    
```bash
#将token输出到环境变量中
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
#利用curl，将token设置进请求头header中，即可安全访问k8s api服务
curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```
