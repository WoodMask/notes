# Nginx 启动过程

### 源码阅读tips
#### tips 1
nginx源码中的初始化逻辑，一般都需要create完再进行init，create是为了先分配空间，而且往往会先根据之前的“经验”（old_cycle）先预分配空间，来避免后续空间不够而进一步扩容。
#### tips 2
cycle在nginx中指的是一次启动周期，nginx使用cycle模型来实现平滑启动的功能，以达到热更新配置而不停止服务的功能。



### 相关文章
> http://tengine.taobao.org/book/chapter_11.html