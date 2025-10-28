Q:hpa的扩展时间间隔如何设置  
A:Kubernetes 将水平 Pod 自动扩缩实现为一个间歇运行的控制回路（它不是一个连续的过程）。间隔由 kube-controller-manager 的 --horizontal-pod-autoscaler-sync-period 参数设置（默认间隔为 15 秒）    
> https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/  

Q:hpa v2版本支持的功能有哪些    
>　https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/horizontal-pod-autoscaler-v2/#HorizontalPodAutoscalerSpec    

Q:hpa支持的可扩容对象类型有哪些    
A：官方文档和手册均无详细的列表清单，只有horizontalpodautoscalerspec里面定义是一个CrossVersionObjectReference类型的引用，可以任意指定其Kind, 应该所有能够进行scaling的资源都可以作为引用对象传入。
> https://cloud.tencent.com/developer/inventory/10387/article/1096512
> https://sourcegraph.com/github.com/AliyunContainerService/kubernetes-cronhpa-controller/-/blob/pkg/controller/cronjob.go?L99:23-99:26
> https://sourcegraph.com/github.com/AliyunContainerService/kubernetes-cronhpa-controller/-/blob/pkg/controller/cronjob.go
> https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#horizontalpodautoscaler-v2-autoscaling
> https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#horizontalpodautoscalerspec-v2-autoscaling

Q:hpa对于custom.metrics.k8s.io版本的支持关系    
A:虽然官网没有明确说，但是目前ocp3.11（也就是k8s v1.11)的环境测试看来，hpa v1和v2beta1使用的都是custom.metrics.k8s.io/v1beta1版本，而比较新的prometheus-adapter(测试使用的是0.9.0)内置安装的是custom.metrics.k8s.io/v1beta2，这会导致hpa获取不到对应的metrics，不过prometheus-adapter向上兼容custom.metrics.k8s.io/v1beta1，重新安装即可。

Q:vpa支持的版本列表是什么    
> https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#compatibility

| VPA version     | Kubernetes version |
|-----------------|--------------------|
| 1.0             | 1.25+              |
| 0.14            | 1.25+              |
| 0.13            | 1.25+              |
| 0.12            | 1.25+              |
| 0.11            | 1.22 - 1.24        |
| 0.10            | 1.22+              |
| 0.9             | 1.16+              |
| 0.8             | 1.13+              |
| 0.4 to 0.7      | 1.11+              |
| 0.3.X and lower | 1.7+               |     

Q:ca支持的版本列表是什么    
| Kubernetes Version  | CA Version   |
|--------|--------|
| 1.28.X | 1.28.X |
| 1.27.X | 1.27.X |
| 1.26.X | 1.26.X |
| 1.25.X | 1.25.X |
| 1.24.X | 1.24.X |
| 1.23.X | 1.23.X |
| 1.22.X | 1.22.X |
| 1.21.X | 1.21.X |
| 1.20.X | 1.20.X |
| 1.19.X | 1.19.X |
| 1.18.X | 1.18.X |
| 1.17.X | 1.17.X |
| 1.16.X | 1.16.X |
| 1.15.X | 1.15.X |
| 1.14.X | 1.14.X |
| 1.13.X | 1.13.X |
| 1.12.X | 1.12.X |
| 1.11.X | 1.3.X  |
| 1.10.X | 1.2.X  |
| 1.9.X  | 1.1.X  |
| 1.8.X  | 1.0.X  |
| 1.7.X  | 0.6.X  |
| 1.6.X  | 0.5.X, 0.6.X<sup>*</sup>  |
| 1.5.X  | 0.4.X  |
| 1.4.X  | 0.3.X  |    
