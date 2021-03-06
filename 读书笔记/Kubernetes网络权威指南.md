Service Mesh解决微服务发现和治理问题：微服务之间的调用关系，服务异常后的熔断、降级，调用链的跟踪、分析；企业只需在云原生和微服务架构的实践道路上根据自己的业务做适当的微服务拆分，无须过多关注底层服务发现和治理的复杂度；

Istio提供了真正可供操作、非侵入式的方案，相对于Spring Cloud、Dubbo这些SDK方式让人有种耳目一新的感觉。

> 屏蔽底层复杂度，专注于开发。

软件设计中的sidecar模式通过给应用服务加装一个“边车”达到控制和逻辑分离的目的。该设计模式通过给应用程序加上一个“边车”的方式拓展应用程序现有的功能，例如日志记录、监控、流量控制、服务注册、服务发现、服务限流、服务熔断等在业务服务中不需要实现的控制面功能，可以交给“边车”，业务服务只需要专注于实现业务逻辑即可。

sidecar模式一般有两种实现方式：

- 通过SDK的形式，在开发时引入该软件包依赖，使其与业务服务集成起来。这种方法可以与应用密切集成，提高资源利用率并且提高应用性能，但也对代码有侵入，受到编程语言和软件开发人员水平的限制；
- agent形式。服务所有的通信都是通过这个agent代理的，这个agent同服务一起部署，和服务一起有着相同的生命周期创建。这种方式对应用服务没有侵入性，不受编程语言和开发人员水平的限制，做到了控制与逻辑分开部署。但是会增加应用延迟，并且管理和部署的复杂度会增加；

Service Mesh将底层那些难以控制的网络通信统一管理，诸如流量管控、丢包重试、访问控制等。而上层的应用层协议只须关心业务逻辑。Service Mesh是一个用于处理服务间通信的基础设施层，它负责为构建复杂的云原生应用传递可靠的网络请求。

Kubernetes集群的每个节点都部署了一个Kube-proxy组件，该组件会与Kubernetes API Server通信，获取集群中的Service信息，然后设置iptables/ipvs规则，直接将对某个Service的请求发送到对应的后端Pod上。

Istio Service Mesh把Kubernetes看作服务注册机构，通过控制平面生成数据平面的配置，数据平面的透明代理以sidecar容器的方式部署在每个应用服务的Pod中。之所以说是透明代理，是因为应用程序容器完全无感知代理的存在。区别在于Kube-proxy拦截的是进出Kubernetes节点的流量，而Istio sidecar拦截的是进出该Pod的流量。