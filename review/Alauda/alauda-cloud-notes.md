# 灵雀云云原生相关笔记

## 基本技术栈
基于K8s、Kube-OVN、Ingress、Helm

## Alauda Load Balancer
ALB是灵雀云自带的负载均衡方案，支持典型的四层和七层负载均衡，支持多种复杂规则的灰度发布。

## Operator Hub
通过Operator Hub，展示用作部署和管理Kubernetes 原生应用程序的Operator。
### 什么是Operator
+ 首先要了解什么是Helm，大家应该听过helm，它是一个命令行工具，帮你同时编排多个K8S资源（比如：Deployment+Service+Ingress这样的经典组合），统一发布到K8S。   
它有一个优势就是模板化，比如：所有的PHP应用除了image不同之外的其他部署结构都相同，那么就可以共用一套Helm模板，通过传参image参数来替换YAML细节即可。  
Helm发布的底层原理就是调用K8S的apiserver，逐个YAML推送给K8S，和我们手动去做没多少差别，所以其弊端就是缺乏对资源的全生命期监控。
+ 其次是CRD，全拼CustomResourceDefinitions，也就是自定义K8S资源类型。通过CRD，我们可以在K8S中自定义一种新的资源类型，并且配套实现它在K8S中的底层逻辑（比如通过实现k8s中的Controller，来实现对应的逻辑），以便根据我们的需求实现定制化的资源类型。
+ 而Operator就是为了实现上述目的的一个脚手架项目，能够让我们把一系列的服务和应用打包成一个镜像，让我们只需填写模板就能实现一系列自动化编排。

### reference
> https://yuerblog.cc/2019/08/13/k8s-%E5%85%89%E9%80%9F%E7%90%86%E8%A7%A3operator/


### 支持开箱即用DB服务和MQ服务
通过上述Operator，实现了开箱即用的DB服务和MQ服务，比如MYSQL、Redis、Kafka、RabbitMQ等，并提供了监控和运维。