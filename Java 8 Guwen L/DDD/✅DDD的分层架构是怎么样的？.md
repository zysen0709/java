# 典型回答

DDD的分层架构是一个四层架构，从上到下依次是：用户接口层、应用层、领域层和基础层。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/5378072/1685861039102-25f6a7a4-2c2a-4f05-a775-99079935332e.png#averageHue=%23e1d9cf&clientId=ud6673f08-b3e0-4&from=paste&id=u0d0a7268&originHeight=996&originWidth=1106&originalType=url&ratio=1&rotation=0&showTitle=false&size=238711&status=done&style=none&taskId=u0fbf375e-e66b-426e-97b1-a8877b79c06&title=)

层次之间的调用关系是上层可以调用下层，即用户接口层可以调用应用层、领域层及基础层。应用层可以调用领域层和基础层，领域层可以调用基础层。

但是不能从下往上反向调用，各个层级之间是严格的单向调用的依赖关系。

除了这种简单的四层架构以外，DDD中还有比较典型的洋葱架构和六边形架构

洋葱架构，就是像洋葱一样的一层一层，从外到内的架构形式，如下图：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/5378072/1685861299754-4524e227-7f8d-4f7f-b466-2d20fdea61a1.png#averageHue=%23f3e2cd&clientId=ud6673f08-b3e0-4&from=paste&id=u0ddcbb5d&originHeight=767&originWidth=1123&originalType=url&ratio=1&rotation=0&showTitle=false&size=129108&status=done&style=none&taskId=u3b4b77aa-5e8a-4068-9ac7-7539ba79ece&title=)

他的依赖关系是从外到内的。

六边形架构和洋葱架构有点像，只不过不是圆形，而是六边形的：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/5378072/1685861346529-f22178f9-944f-4b3b-a9c3-4fc2ed9577ea.png#averageHue=%23f7ece2&clientId=ud6673f08-b3e0-4&from=paste&id=uc6679134&originHeight=854&originWidth=1142&originalType=url&ratio=1&rotation=0&showTitle=false&size=101949&status=done&style=none&taskId=ud34c4bae-e6db-4968-b934-2fa67eaf8d4&title=)

虽然 DDD 分层架构、整洁架构、六边形架构的架构模型表现形式不一样，但是这三种架构模型的设计思想都是微服务架构高内聚低耦合原则的完美体现，**都是以领域模型为中心的设计思想**。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/5378072/1685861394105-ef041ab6-8d33-416b-9617-44a263ef1d62.png#averageHue=%23f6eadf&clientId=ud6673f08-b3e0-4&from=paste&id=u0b86cd6d&originHeight=971&originWidth=1310&originalType=url&ratio=1&rotation=0&showTitle=false&size=197879&status=done&style=none&taskId=u0629a59b-f529-47c5-9eb5-024c2644e1e&title=)


本文图片参考自：《极客时间——DDD实战课》
