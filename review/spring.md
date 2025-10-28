# Spring笔记
##  1 Spring Bean 生命周期
spring生命周期其实和jvm里面的对象生命周期很类似，基本浓缩成以下几个阶段：    
装载bean资源->调用构造器实例化->依赖注入->初始化->销毁    
这几个阶段是最为重要的阶段，其中在实例化的前后和初始化的前后，都会伴随着实现BeanProcessor接口的前置调用和后置调用。
## 2 Spring Bean的依赖注入
### 2.1 注入方式
Spring Bean大致有两种注入方式：   
构造器注入和Setter注入    
#### 2.1.1 构造器注入
构造器注入时，将会优先采用类型方式去获取依赖的bean，再根据注入的参数名字去其中获取最适配的依赖bean，如果依赖未创建，将会先创建依赖的bean后再注入。
#### 2.1.2 参数注入
参数注入也是同理。
### 2.2 解决循环依赖
Spring在注入时，如果出现A中依赖B,而创建B时又依赖A的情况，也就是循环依赖的情况，将会采用以下解决方式：  
1.如果是构造器注入发现了循环依赖，那么将会在运行时抛出异常，提醒存在循环依赖。  
2.如果是setter注入的方式，Spring将会智能的去解决这个问题，解决方案是，在创建A时如果检查到需要创建其依赖B，那么在进入B之前，将会把A（或者其代理）做一次缓存，那么在创建B时直接用A的缓存来生成A的实例注入B，最后再回去A完成整个的依赖注入。

## 引用
> https://www.jianshu.com/p/1dec08d290c1
> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-type-determination