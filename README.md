# DDD
DDD规范
整个工程代码量较大，规范化不是一蹴而就的，我们需要做的是统一代码风格，确定规约，确保后续新增的代码是规范的，并基于已有的规约逐步将已有不符合规范的代码进行重构、改写。
问题1: 明确当前系统每一模块的作用以及明确调用关系

![image](https://github.com/woshidaniu-com/DDD/assets/8299602/8e787b70-b98e-43e3-9727-abc3be0bfa9a)

注意事项：
1）domain层对外部的依赖尽可能少，domain层不会去依赖adapter层和application层
2) adapter层可以访问所有层
3）application层不能访问adapter层，可以访问domain层

问题2: 各模块职能不清晰，导致引入的包出现在不应该出现的模块、功能代码实现出现在不合适的位置

目前工程结构大体分成四部分: adapter适配层、application应用层、domain领域层、common公共层，各层职责如下：
### adapter层
#### adapter-acl层
 该层作为防腐层，主要职责是通俗讲适配外部接口，因而1) 所有涉及到外部rpc接口调用的逻辑都应该放到adapter-acl层 2) 业务逻辑不应该在adapter-acl中实现，而应该调用domain层或者application层。
 
#### adapter-persistence层
 adpter-persistence中应该只包含最纯粹的数据库操作，包含数据库增删改查、DO定义、mapper、repo实现类以及DO与领域模型的converter。
 Q1: mapper和repository(资源库)需要进行区分
 首先明确下mapper定义在adapter-persistence层，repository资源库的接口定义在domain-model中，reposiroty的实现类定义在adapter-persistence中。另外所有数据库的操作应该通过调用repository完成，不可以单独在某个domainservice、application中引用mapper进行数据库操作
Q2: 聚合根与repository的定义应该一一对应, 非聚合根对象数据库操作应该在相应的聚合根中实现
 这里涉及到“聚合根”的概念，聚合根就是软件模型中那些最重要的以名词形式存在的领域对象。关于聚合根的定义没有一个特别精确的定义，符合某种特征的就一定要定义成聚合根，更多的还是基于领域知识的一种直觉。前面说了实体与repository应该一一映射，而且操作数据库只能通过repository进行操作。针对非聚合根的实体进行数据库操作时，是去操作实体对应所在的聚合根对应的repository实现的
 
#### adapter-rest层
 主要承载rest接口的编写和converter转换的功能，在adapter-rest中新增代码，业务逻辑不应该在该模块中实现。
 
#### adapter-rpc层
 主要用于实现暴露给外部调用的rpc接口的实现类，业务逻辑不应该在该模块中实现，而是通过调用domain层或者application层实现。
 
### application层
UML中有用例(Use Case)的概念，表示的是软件向外提供业务功能的基本逻辑单元。在DDD中，由于业务被提到了第一优先级，那么自然地我们希望对业务的处理能够显现出来，为了达到这样的目的，DDD专门提供了一个名为应用服务(ApplicationService)的抽象层。

ApplicationService采用了门面模式，作为领域模型向外提供业务功能的总出入口，就像酒店的前台处理客户的不同需求一样。

applicationService需要遵循以下原则：

•	业务方法与业务用例一一对应

•	业务方法与事务一一对应：也即每一个业务方法均构成了独立的事务边界，不过我们现在应用系统架构是微服务架构，要保证分布式一致性很难，不过我们在日常设计中幂等性是最基本的保障。

•	本身不应该包含业务逻辑：业务逻辑应该放在领域模型中实现，更准确的说是放在聚合根中实现。

 ![image](https://github.com/woshidaniu-com/DDD/assets/8299602/d31d501b-e60e-43e0-aa52-1be3fc1dab13)

业务逻辑应该放到domain-model中或者domain-service中。几乎每一层的逻辑都是希望做到尽可能的薄，尽量把逻辑收敛到domain层，这也确实是我们的初衷。

### common层
#### common-lang
 存放公共类，比如工具类、异常码等
#### common-facade
  对外提供的rpc接口放到这个包内，另外还包含返回数据类型XXXDTO，查询模型XXXQuery, 操作模型XXXCommand等。
#### domain层
 domain层是我们整个应用最核心、最稳定的层，最核心是因为领域相关的逻辑都应该在domain层，最稳定是因为领域模型应该是稳定的，不受外部接口、系统的变更而变更的，应该高度是内聚的。
 
 聚合根是业务逻辑的主要载体，也就是说业务逻辑的实现代码应该尽量地放在聚合根或者聚合根的边界之内。但有时，有些业务逻辑并不适合于放在聚合根上，在这种“迫不得已”的情况下，我们引入领域服务，因此程序中的DomainService应该越少越好。
 
Q1: domain层应该高度内聚，只有可能依赖common层，不应该有其他层的依赖

这些对第三方系统的依赖都应该在adapter-acl中实现，而不是放到domain中，需要集中进行清理，将对应逻辑移动到对应的位置。

Q2: domain service过重，很多逻辑应该收敛到领域模型中

不是说什么逻辑都应该塞到领域模型中的，需要判断逻辑是不是领域内的逻辑。
