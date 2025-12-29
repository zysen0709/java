# 典型回答

下图，是Dubbo的官网中给出的一张关于Dubbo的完整调用链：

![](https://cdn.nlark.com/yuque/0/2024/jpeg/5378072/1707629223479-ac4d4b09-b66a-4608-bc4c-564146a4950a.jpeg#averageHue=%23bdc8bd&clientId=u2f40da32-bc51-4&from=paste&id=ue4ffd253&originHeight=738&originWidth=800&originalType=url&ratio=1.5&rotation=0&showTitle=false&status=done&style=none&taskId=u815af6f1-1f87-41c2-90d4-bbf2d5c3c3c&title=)

有些人能看懂，有些人看不懂，看不懂没关系，我给大家简化一下就可以看得懂了：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/5378072/1707633108959-142d1d14-4d88-470d-a0e8-5b6bd4016145.png#averageHue=%23fae897&clientId=u2f40da32-bc51-4&from=paste&height=1067&id=u9856636a&originHeight=1067&originWidth=833&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=114429&status=done&style=none&taskId=u8cc443ec-b2f5-4f57-914a-14e0fa55a9b&title=&width=833)

首先，一次调用从Proxy开始，这里主要是借助JDK、javaassist等实现的动态代理，然后在开始执行之前，会经过一个filter，判断以下本地是否存在接口的缓存以及mock等，如果都没有，开始通过invoker进行服务调用。

服务调用者在启动时会从注册中心拉取和订阅对应的服务列表， Cluster会把拉取的服务列表聚合成一个Invoker列表。

在Invoker执行过程中，会先通过Directory获取所有可以调用的服务提供者的列表。接下来在根据一些负载均衡的规则进行负载均衡来选出一个具体的Invoker。

在开始调用Invoker的invoke方法前，需要再进行一些过滤规则，如上下文处理、限流、以及计数等。

接下来就通过Invoker进行调用，在调用时会使用具体的客户端进行网络通信，如Netty，在这个过程中当然还需要做协议封装、数据的序列化等动作。

同时，客户端在收到请求后，同样需要进行反序列化，然后把请求交给线程池来进行调度执行。

再根据请求查找对应的Exporter，然后开始执行Invoker，这里就要调用具体的服务实现方法了。

然后再按照原路把结果返回。
