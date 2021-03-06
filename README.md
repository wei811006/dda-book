
微服务是怎么被划分出来的？
===

我："你觉得我们这个架构用微服务怎么样？"

同事："你的问题是什么？"

我："没有问题，我就是感觉可以用微服务来做这个项目比较好，看了一些资料，微服务有 XXX 特点，感觉挺好的。"

同事："所以，你设计这样一个架构，解决的问题是什么呢？"

我陷入沉思。

早期的时候，每一次和架构师同事讨论问题，当我想要说出我的技术方案时，他总是用这个问题反问我。一开始我被弄得莫名其妙，后来慢慢习惯这种对话方式。其实这是一种问题驱动的思维方式，并且对架构师来说，至关重要。

微服务架构几乎成为互联网公司架构的标准形态，我们在讨论如何划分、设计微服务架构，甚至领域驱动设计（DDD）时，我们应该回归初心，当我们开始讨论怎么划分微服务时，我们应该能回答下面的问题：

- 需要解决的问题是什么？
- 什么是微服务，和 SOA（面向服务的架构） 的区别是什么？
- 为什么是微服务而不是其他架构方式（例如：SOA）？
- 带来的好处是什么？
- 微服务带来的成本有多大？
- 对系统造成的影响是什么？

## 需要解决的问题是什么？

一个几个人的小团队提供的服务一般不会太过于复杂，另外应用也不会特别大，这个时候一个单体应用对团队是非常友好的。不用考虑集成、部署等分布式系统的各种麻烦。实际上，大部分应用程序，还达不到使用微服务这样复杂架构的必要条件。

当团队开始变大，或者应用变得越来越复杂时，会产生几个痛点：

- 应用性能瓶颈，扩容困难
- 故障和系统弹性无法隔离
- 应用复杂到无法理解，模块耦合高，应用职责过多
- 过多的人工作到同一个代码库中
- 应用大到编译、部署时间长


其中，促使开发者从单体应用走向分布式系统最大的动机就是应用性能瓶颈，扩容困难的问题。实际上，当下的工程师们都是在开发广义的 "分布式系统"，因为真正的单体应用系统只存在于大型机、小型机时代了。客户端-服务器模式是一个最基本的分布式系统，无论是 C/S 模式还是 B/S 模式。

"客户端-服务器模式" 把需要单体主机需要全部完成的工作分离到了客户端实现了一次扩容，这个过程对软件行业进行了一次革命。但是随着需求量日益扩张，单纯的"客户端-服务器模式"已经不能满足需求。高性能、大容量的需求往往来自于面向 C（customer） 端的系统，这也解释了为什么分布式系统由面向消费者的公司驱动。Google 和 Salesforce 有海量的在线用户，他们比微软、Oracle 更为热衷分布系统的研发。

当单纯的 "客户端-服务器模式" 不能满足需求时，人们开始寻找新的拓展方式。其中一个方向是将 B/S 模式下的服务器工作下放到浏览器，这就是 SPA（Single Page Application） 富客户端，让前端单页应用承载用户交互和界面相关的工作，让服务器专注于处理业务逻辑和运算。另外一个方向是服务器角色分化：状态和运算分离。将数据库和应用分开部署，将应用中的 Session 入库，让应用无状态化，然后大规模部署应用服务器；剥离和业务无关的组件，例如邮件、文件、推送，演化出独立的服务，分布式文件系统应运而生。然而痛点在于，前面是根据技术进行水平切分，那么如何根据业务模块进行拆分，继续拓展呢？

其次的痛点是故障和系统弹性无法隔离。故障隔离的意思是某部分相对独立的业务中断，不应该影响整个系统。例如一个电商系统，因为某些原因造成无法支付，但是不应该影响用户浏览商品的业务。系统弹性是指应用扩容的需求，单体系统无法做到针对某一部分业务单独扩容，例如秒杀的场景，我们无法针对下单或者结算进行单独扩容。下单造成大流量的访问会导致整个系统不可用。

上面这两个痛点，是从最终用户体验的角度出发，这是真正推动技术进步的原因，也是技术变革产生业务价值的地方。另外三点都是从开发和运维的角度出发，这部分痛点来自开发者，但有可以通过各种方式在单体的应用下改善或克服。

当应用变得非常复杂时，程序变成了一个大泥球（A Big Ball of Mud）。模块之间的依赖关系变得极其复杂，程序开始变得混乱不堪，不过需要提前澄清的是单体或者分布式架构对这个问题无能为力。分布式架构的引入，会让大泥球的代码变成分布式大泥球，进一步增加系统熵 (Entropy，一种物理学概念，系统的混乱程度)。如何避免大泥球呢？后面会谈到可以通过良好的面向对象设计和重构完成。

其次，大的单体还会有编译时间长、部署时间长的问题。如果一个上百人的团队工作在一个代码库中，每天的工作除了在修复冲突之外，就是在等待编译。但这些都不是不可以克服的，Linux 内核就是一个超级单体系统，极长的编译时间和极多的参与人数，但 Linux 项目也能良好的运行。

所以这部分比较啰嗦，毛泽东思想告诉我们，分析问题的时候需要分清主要矛盾和次要矛盾。那么我们在分析系统问题的时候，不仅应该见微知著，也应该抓住关键。

所以问题的关键是系统的 "耦合"，"耦合" 的存在让我们难以拓展、故障隔离、开发困难、无法分开编译。

## 什么是微服务？

根据维基百科的定义，微服务是一种 SOA 的变体，特征是服务之间通过松耦合的方式集成。一般采用轻量级的传输协议（Http），以及不要求服务内采用同样的技术栈实现，通过暴露统一的 WEB API 实现相互通信。

业界对微服务的共识是：

- 服务根据业务能力划分，能提供一定范围内完整的业务价值
- 服务可以单独部署、运维
- 服务可以基于不同的编程语言、数据库等技术实现
- 服务之间使用轻量级的网络协议通讯，例如 HTTP 
- 服务有自己的独立的生命周期
- 服务之间松耦合

"服务根据业务能力划分"和"松耦合"需要特别注意，这是微服务和 SOA 最大的区别。

在解决扩容这个问题上，让我们看下 SOA 和微服务的区别。在解决拓展性的问题，有一个非常好的模型，叫做 "AKF扩展立方"。AKF可扩展立方 （Scalability Cube），来自于《可扩展的艺术》一书。

这个立方体中沿着三个坐标轴设置分别为：X、Y、Z。

![AKF拓展立方](./images/akf.png)


- X 轴扩展 —— 无差别的水平的数据和服务复制，具体的实践可以对应为加机器
- Y 轴扩展 —— 根据应用中职责的划分，具体的实践为对各个业务线剥离
- Z 轴扩展 —— 利用特殊属性划分数据集合，例如基于租户模型对用户进行切割

Z 轴拓展实际上非常常见，例如电信运营商基于地域对用户进行划分，北京和上海的用户使用不同的号码段进行管理。微服务出现之前，我们对系统的拓展更多的关注于 X 拓展，负载均衡、读写分离都是为了解决如何进行服务复制来承载更多的请求。SOA 应用进行了一部分的 Y 轴拓展，但是 SOA 和微服务的本质不同在于，SOA 拆分的不是独立的服务，而是组件，SOA 组件必须注册到企业总线或者其他机制中才能对外提供整体的服务。

一个典型的区别在于，SOA 往往将业务逻辑拆分成不同的 SOA 服务，但是数据依然是集中的。业界对微服务的共识在于数据应该被划分到各个微服务中，每个微服务可以独立演进、独立发布甚至独立运营，提供完整的服务能力，并通过松耦合的方式集成。这也是企业实现中台的基本能力。

一个典型的 SOA 架构像这样，又不同的服务，但是服务是作为系统的一部分组件存在的，在 Java EE 的生态下作为一个部署到容器中的 war 包存在的。
这些服务共用基础设施，尤其是数据库。

![一份简单的 SOA 架构图](./images/SOA.png)

而对比之下，微服务架构下，每个服务有自己的数据库，并且自己单独部署，提供 RESTful API 供其他服务。应用后者其他服务调用微服务时和调用第三方系统并没有两样。

![一份简单的微服务架构图](./images/micro-service.png)

参考下面的韦恩图，让我们对微服务架构的理解更进一步。

![分布式架构、SOA架构、微服务架构的关系](./images/microservcie-veen.png)

## 为什么我们需要微服务?

水平划分就够了，完全不需要垂直划分，那就不需要微服务。

前面我们谈到，主流微服务形态是各个微服务彼此独立，内部的实现不可见，能独立的提供服务。回到我们的需要解决的问题上，微服务的基础是建立在 AKF 的 Y 轴拓展上的。

![微服务在 AKF 拓展立方中的作用](./images/akf-example.png)

SOA 已经解决了微服务架构同样会遇到的 X 轴拓展和 Z 轴拓展，这些技术已经非常成熟：负载均衡、分库分表等。使用微服务其中一个原因就是 Y 轴拓展。实现 Y 轴的基础是服务解耦，并且随着解耦，给我们带了其他好处。

**应用复杂性的隔离**。分解巨大单体应用为多个微服务，降低了单体应用的复杂性问题。每个应用只需要关注自己的 API 内提供的能力即可，这样每个微服务比单体应用开发难度大大降低。在应用的复杂性上来讲，一加一大于二，所以架构师的工作需要把两个一隔离开，让大多数开发者在低复杂度的状态下开发。

**步速不一致的弹性**。云原生时代，一切都在云上。云，带来革命的优势就是弹性和伸缩，当我们用户量陡然提升的时候，可以快速地创建资源，用户数量减少也可以快速地销毁资源。但是如果我们是一个大的单体应用，每个模块的变化速率是不同的，换句话说就是弹性边界不同。通过服务化的拆分，能良好的实现弹性优化。

**更容易实现敏捷开发**。敏捷开发的理念是快速响应变化，要求频繁部署上线。因此快速编译、快速部署上线，减少服务中断时间，就成了敏捷开发的必要条件。服务化后的每个团队，把原来单体时代负责的模块当做一个独立的服务开发，内部的技术选型、具体实现灵活性更高。

当然，这些好处是对于足够庞大的单体系统良好服务化后而言的。如果你本身工作在一个非常小的团队，应用规模并不大，则很难体会到服务化的优点。（无论是SOA 还是微服务）。同时，如果一个耦合的单体应用，在不经过耦合的过程，被不合理的拆分成微服务，一样非常痛苦。另外，如果微服务被划分的非常小，以致于一个团队修改每一个功能都需要跨微服务操作，也是非常痛苦的。

最后，新技术往往即使令人激动人心也需要付出代价，微服务亦然，下面我们看下在实现微服务的过程中会有那些挑战。

## 微服务的成本和新问题

毋庸置疑，微服务是有成本的。在资源和基础设施有限的创业公司中，这种成本体验的尤为明显。微服务的成本来来自于架构、技术落地和协作等多个方面。

**架构成本**

如果需要将单体系统改造成微服务系统的第一步是解耦，只有当应用本身的耦合降低之后才能进行服务化改造，否则系统从"大泥球"的单体应用编程了分布式"大泥球"。

当服务化后，业务逻辑被分散到多个服务和代码仓库，不仅需要专业的架构师进行架构，也需要专人对架构进行守护和重构。然而，现实中架构并不是一成不变的，分布式架构对架构师的挑战大得多。

虽然服务化后，单个服务可以快速的响应业务需求的变化，但是就系统整体的架构来说，架构的重构是很艰难的。因此强大的架构能力，就成了微服务架构的必要条件。

**基础设施的挑战**

1996年，Gartner 就提出 SOA 的思想，前面我们讲到，微服务也可以算作 SOA 的一种形态。

> 对于复杂的企业IT系统，应按照不同的、可重用的力度划分，将功能相关的一组功能提供者组织在一起为消费者提供服务

为什么 SOA 并没有在业界流传开呢？因为，SOA 对一个企业的基础设施提出了巨大挑战，往往只有少数顶级的公司能做到，或者有必要实施 SOA 架构。随着云、开发运维一体化的兴起，服务化的成本开始降低，一些中型的公司也可以有能力进行服务化变革了。

国标草案 《通用微服务平台》讲微服务实施分为，给出了一个技术设施层能力的要求，部分如下：

- 自动部署
- 资源动态分配
- 认证鉴权
- 服务发现
- 服务注册
- 负载均衡 
- 服务契约
- 健康检查 
- 链路跟踪
- 分布式事务

**人员组织成本**

## 微服务的准入清单

在几年的实践中，被强行采用微服务的项目坑过的不少，有过同样体验的朋友不在少数。当服务内的业务逻辑非常单薄时，开发过程中的大部分工作都在处理服务间调用，尤其是服务内聚不够时，让这种情况雪上加霜。

下面是一份微服务的准入清单，帮助我们在技术选型时候，要不要使用微服务：




## 微服务划分方法论

微服务已经有非常多成熟的实践，大量开源的微服务框架可以供我们选择，Spring Cloud、Dubbo等框架让我们对微服务的实践难度和成本大大降低。有大量的技术书籍在讨论微服务、分布式架构等技术的具体实现方式，一个公司想要实践微服务已经非常容易了。

我在参与和实践了大量微服务项目过后，让我困惑的往往不是某一个具体技术的选型，服务发现、调用链追踪都有成熟的开源组件帮助我们完成。而让我苦思冥想的另一个问题是：如何把一个单体应用拆分出更合理、更有弹性的微服务架构？

当我查阅企业架构相关的书籍时，了解到IT系统架构不仅需要应对技术架构的挑战之外，还需要应对业务架构方面的挑战。如何从业务的角度出发设计出更加贴切企业战略和业务需求的架构，在适合业务发展和避免过度设计之间取得一个平衡？

在过去大量的实践中，我们的架构师关注在软件的基础设施上，云、数据库分库、数据库分表、集群、负载均衡、认证和授权等这些技术实践上。对于设计一个系统而言，做的第一件事就是关注数据库表、字段、关联。

我们往往忽略了一个非常重要的环节，就是领域建模或者叫做业务架构，这个环节被隐藏到架构师或者技术 Leader 的经验决策中了。在单体应用时期这个问题不是很明显，订单和商品需要设计多少个表、有什么字段、表之间的关联怎么样，富有经验的开发者都能做出合适的决策。

然而在微服务的时代，我们遇到的问题就变成了，订单和产品是否应该划分到两个微服务中，每个服务的职责和边界在哪里，服务之间的API怎么设计，服务之间的依赖关系是什么样子。这部分的架构设计看起来好像和我们做技术选型时考虑的角度不太一样，不再关注技术细节，而是关注于业务细节。相对于处理一些非功能需求，这部分工作对抽象能力、业务分析能力、面向对象能力有了更高的要求。

软件设计是一个复杂的系统问题。于是我们在架构设计中，有两类问题需要解决：同非功能需求相关的技术复杂度，以及和功能需求相关的业务复杂度。

非功能需求是指那些跨功能的通用需求，例如安全、性能、并发等，围绕着这些问题我们发展出了各种解决方案，例如OAuth、缓存技术、负载均衡等，但是这些复杂问题其实和业务是没有关系的。我们把解决这类复杂问题的架构思维叫做技术架构。

功能需求是指用户能操作的使用的需求，例如登录、下单、申请退款，这些复杂问题实际上和采用的技术关系并不大，从编程语言、框架、数据库上来说， 都能完成这些业务要求。无论是 Java、.Net、PHP 以及是否采用微服务，都不妨碍我们完成业务功能。

业务复杂度也是软件开发过程中非常重要的一部分， **我们暂且把解决这类问题的思维方式叫做业务架构**

技术架构大量依赖于实践和经验，在行业内具有相当的通用性，可以采用大量的开源方案。而业务架构解决的问题是如何分析业务逻辑，正确的对业务进行抽象，然后得到合理的软件架构。业务架构非常强的依赖于面向对象的思想和高度的抽象思维，是一线应用开发者主要思考的问题，同公司的产品相关，通用性非常弱。

| -              | 技术架构           | 业务架构           |
| -------------- | ------------------ | ------------------ |
| 实践           | 负载均衡、高可用   | 服务划分、应用解耦 |
| 对开发者的要求 | 主流解决方案的经验 | 抽象、逻辑分析能力 |
| 解决的问题     | 技术复杂度         | 业务复杂度         |

当然，这两种架构思维并不能彻底分开。讨论这两种架构的不同可以帮助我们换种思路去看待架构问题。

业务架构不应该限制于技术细节，在做业务架构的时候应该和我们的业务专家讨论，分析业务在逻辑上的可行性或者矛盾。技术架构则不应该和业务过多的捆绑，架构师讨论这些问题的时候，应该听取某个技术领域中的专家意见，例如更换一种缓存策略是否能大幅度提高性能问题。

另外一个方面，技术架构应该是服务于业务架构，而非凌驾于业务架构之上。尽可能的将最优的技术资源花在核心业务逻辑上，优秀的武器应该首先能打中靶子。

## 用 DDD 来设计微服务

被合理划分成多个微服务的分布式系统，在逻辑上和解耦良好的单体系统是一致的。大家可能都有这样的体会，因为单体应用某些模块已经被良好的解耦了，在划分成多个服务时显得非常自然。例如一个企业应用中，配置管理往往相对独立，一般作为单独的模块设计。在划分微服务的时候很容易划分处理，但是订单、商品、支付等部分往往依赖关系错综复杂，调用关系千丝万缕，做微服务划分时显得艰难。

所以，良好的微服务设计，很重要的一部分就是如何对业务的建模和分析，在逻辑上有一个清晰的关系。

DDD 基础
===

领域驱动设计（DDD） 是 Eric Evans 提出的一种软件设计方法和思想，主要解决业务系统的设计和建模。DDD 有大量难以理解的概念，尤其是翻译的原因，某些词汇非常生涩，例如：模型、限界上下文、聚合、实体、值对象等。

实际上 DDD 的概念和逻辑本身并不复杂，很多概念和名词是为了解决一些特定的问题才引入的，并和面向对象思想兼容，可以说 DDD 也是面向对象思想中的一个子集。如果遵从奥卡姆剃刀的原则，“如无必要，勿增实体”，我们先把 DDD 这些概念丢开，从一个案例出发，在必要的时候将这些概念引入。

## 从纸和笔思考 IT 系统的工作逻辑

让我真正对计算机软件和建模有了更深入的认识是在一家餐厅吃饭的时候。数年以前，我还在一家创业公司负责餐饮软件的服务器端的开发工作，因为工作的原因，外出就餐时常都会对餐厅的点餐系统仔细观察，以便于改进我们自己产品的设计。

一次偶然的情况，我们就餐的餐厅停电了，所幸是在白天，对我们的就餐并没有什么影响。我突然很好奇这家店，在收银系统无法工作的情况下怎么让业务继续运转，因此我饶有趣味的等待服务员来接受我们的点单。

故事的发展并没有超出预期，服务员拿了纸和笔，顺利的完成了点餐，并将复写纸复写的底单麻溜的撕下来交给了后厨。我这时候才回过神来。

**软件工程师并没有创造新的东西，只不过是数字世界的砖瓦工，计算机系统中合乎逻辑的过程，停电后人肉使用纸和笔一样合乎逻辑。**

合乎现实世界的逻辑和和规则，使用鼠标和键盘代替纸和笔，就是软件设计的基本逻辑。如果我们只是关注于对数据库的增、删、改、查（CRUD），实际上没有对业务进行正确的识别，这是导致代码组织混乱的根本原因。

会计、餐饮、购物、人员管理、仓储，这些都是各个领域实实在在发生的事情，分析业务逻辑，从中找出固定的模式，抽象成计算机系统中对象并存储。这就是 DDD 和面向对象思想中软件开发的一般过程。

你可能会想，我们平时不就是这样做的吗?

现实是，我们往往马上关注到数据库的设计上，想当然的设计出一些数据库表，然后着手于界面、网络请求、如何操作数据库上，业务逻辑被封装到一个叫做 Service 对象上，这个对象不承载任何状态，业务逻辑通过修改数据库实现。

![面向数据库的编程方法](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/face2database.png)

一般来说这种方法也没有大的问题，甚至工作的很好，Fowler 将这种方法称作为 **事务脚本（Transaction Script）**。还有其他的设计模式，将用户界面、业务逻辑、数据存储作为一个“模块”，可以实现用户拖拽就可以实现简单的编程，.net、VF曾经提供过这种设计模式，这种设计模式叫做 SMART UI。

这种模式有一些好处。

- 非常直观，开发人员学习完编程基础知识和数据库 CRUD 操作之后就可以开发
- 效率高，能短时间完成应用开发
- 模块之间非常独立

麻烦在于，当业务复杂后，这种模式会带来一些问题。

虽然最终都是对数据库的修改，但是中间存在大量的业务逻辑，并没有得到良好的封装。客人退菜，并不是将订单中的菜品移除这么简单。需要将订单的总额重新计算，以及需要通知后厨尝试撤回在坐的菜。

不长眼的新手程序员擅自修改数据片段，整体业务逻辑被破坏。这是因为并没有真正的一个 “订单” 的对象负责执行相关的业务逻辑，`Sevice` 上的一个方法直接就对数据库修改了，保持业务逻辑的完整，完全凭程序员对系统的了解。

![业务逻辑一致性难以维护](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/system-error-when-data-unconsistant.png)

我们在各个餐厅交流的时候，发现这并不是一个 IT 系统的问题。某些餐厅，所有的服务员都可以收银，即使用纸和笔，收营员划掉菜品没有更新小计，另外的服务员结账时会发生错误。于是餐厅，约定修改菜品必须更新订单总价。

我们吸收到这个业务逻辑到 IT 系统中来，并意识到系统中这里有一些隐藏的模型：

- 订单
- 菜品

我们决定，抽象出订单、菜品的对象，菜品不应该被直接修改，而是通过订单才能修改，无论任何情况，菜品的状态变化都通过订单来完成。

复杂系统的状态被清晰的定义出来了， `Service` 承担处理各个应用场景的差异，模型对象处理一致的业务逻辑。

在接触 Eric Evans 的 DDD 概念之前，我们没有找到这种开发模式的名字，暂时称作为 **朴素模型驱动开发**。

![朴素模型驱动开发](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/simple_ddd_patten.png)


## 模型和领域模型

从上面的例子中，模型是能够表达系统业务逻辑和状态的对象。

模型是一个非常宽泛的概念，任何东西都可以是模型，我们尝试给模型下一个定义，并随后继续将领域模型的概念外延缩小。

**模型，用来反映事物某部分特征的物件，无论是实物还是虚拟的** 古人用八个卦象作为世界运行规律的模型；地图用线条和颜色作为地理信息的模型；IT 系统用 E-R 作为对象或者数据库表关系的模型；

我们知道要想做好一个可持续维护的 IT 系统，实际上需要对业务进行充分的抽象，找出这些隐藏的模型，并搬到系统中来。如果发生在餐厅的所有事物，都要能在系统中找到对应的对象，那么这个系统的业务逻辑就非常完备。

现实世界中的业务逻辑，在 IT 系统业务分析时，适合某个行业和领域相关的，所以又叫做领域。

**领域，指的特定行业或者场景下的业务逻辑**。

**DDD 中的模型是指反应 IT 系统的业务逻辑和状态的对象，是从具体业务（领域）中提取出来的，因此又叫做领域模型**。

通过对实际业务出发，而非马上关注数据库、程序设计。通过识别出固定的模式，并将这些业务逻辑的承载者抽象到一个模型上。这个模型负责处理业务逻辑，并表达当前的系统状态。**这个过程就是领域驱动设计。**

我从这里面学到了什么呢？

我们做的计算机系统实际上，是替代了现实世界中的一些操作。按照面向对象设计的话，我们的系统是一个电子餐厅。现实餐厅中的实体，应该对应到我们的系统中去，用于承载业务，例如收银员、顾客、厨师、餐桌、菜品，这些虚拟的实体表达了系统的状态，在某种程度上就能指代系统，这就是模型，如果找到了这些元素，就很容易设计出软件。

后来，如果我什么业务逻辑想不清楚，我就会把电断掉，假装自己是服务员，用纸和笔走一边业务流程。

分析业务，设计领域模型，编写代码。这就是领域驱动设计的基本过程。随后会介绍，如何设计领域模型，当我们建立了领域模型后，会讲解如何使用领域模型指导开发工作。

- 指导数据库设计
- 指导模块分包和代码设计
- 指导 RESTful API 设计
- 指导事务策略
- 指导权限
- 指导微服务划分（有必要的情况）

![领域驱动设计过程](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-process.png)

在我们之前的例子中，收银员需要负责处理收银的操作，同时表达这个餐厅有收营员这样的一个状态。收营员收到钱并记录到账本中，账本负责处理记录钱的业务逻辑，同时表达系统中有多少钱的状态。

## 分析领域模型时，请把”电“断掉

我们进行业务系统开发时，大多数人都会认同一个观点：将业务和模型设计清楚之后，开发起来会容易很多。

但是实际开发过程中，我们既要分析业务，也要处理一些技术细节，例如：如何响应表单提交、如何存储到数据库、事务该怎么处理等。

使用领域驱动设计还有一个好处，我们可以通过隔离这些技术细节，先讲业务逻辑建模，然后再完成技术实现，因为业务模型已经建立，技术细节无非就是响应用户操作和持久化模型。

我们可以吧系统复杂的问题分为两类：

- 业务复杂度
- 技术复杂度

![技术复杂度和业务复杂度](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/isolation.png)

**技术复杂度，软件设计中和技术实现相关的问题，例如处理用户输入，持久化模型，处理网络通信等。**

**业务复杂度，软件设计中和业务逻辑相关的问题，例如为订单添加商品，需要计算订单总价，应用折扣规则等。**

当我们分析业务并建模时，不要关注技术实现，会带来极大的干扰。我学到最实用的思维方法，就是在这个过程把”电“断掉，技术复杂度中的用户交互想象成人工交谈，持久化想象成用纸和笔记录。

DDD 还强调，业务建模应该充分的和业务专家在一起，不应该只是实现软件的工程师自嗨。业务专家是一个虚拟的角色，有可能是一线业务人员、项目经理、或者软件工程师。

由于和业务专家一起完成建模，因此尽量不要选用非常专业的绘图的工具和使用技术语言。可以看出 DDD 只是一种建模思想，并没有规定使用的具体工具。我这里使用 PPT 的线条和形状，用 E-R 的方式表达领域模型，如果大家都很熟悉 UML 也是可以的。甚至实际工作中，我们大量使用便利贴和白板完成建模工作。

这个建模过程可以是技术人员和业务专家一起讨论出来，也可以是使用 ”事件风暴“ 这类工作坊的方式完成。

这个过程非常重要，DDD 把这个过程称作 **协作设计**。

通过这个过程，我们得到了领域模型。

![领域模型v1](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-v1.png)

上图使我们通过业务分析得到的一个非常基本的领域模型，我们的点餐系统中，会有座位、订单、菜品、评价几个模型。一个座位可以由多个订单，每个订单可以有多个菜品和评价。

同时，菜品也会被不同的订单使用。

## 上下文、二义性、统一语言

我们用这个模型开发系统，使用领域模型驱动的方式开发，相对于事务脚本的方式，已经容易和清晰很多了，但还是有一些问题。

有一天，市场告诉我们，这个系统会有一个逻辑问题。就是系统中菜品被删除，订单也不能查看。在我们之前的认知里面，订单和菜品是一个多对多的关系，菜品都不存在了，这个订单还有什么用。

菜品，在这里存在了致命的二义性！！！这里的菜品实际上有两个含义：

- 在订单中，表达这个消费项的记录，也就是订单项。例如，5号桌消费的鱼香肉丝一份。
- 在菜品管理中，价格为30元的鱼香肉丝，包含菜单图片、文字描述，以及折扣信息。

菜品管理中的菜品下架后，不应该产生新的订单，同时也不应该对订单中的菜品造成任何影响。

这些问题是因为，技术专家和业务专家的语言没有统一， DDD 认识到了这个问题，统一语言是实现良好的领域模型的前提，因此应该 ”大声的建模“。我在参与这个过程目睹过大量有意义的争吵，正是这些争吵让领域模型变得原来越清晰。

这个过程叫做 **统一语言**。

![领域模型v2](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-v2.png)

和现实生活中一样，产生二义性的原因是因为我们的对话发生在不同的上下文中，我们在谈一个概念必须在确定的上下文中才有意义。在不同的场景下，即使使用的词汇相同，但是业务逻辑本质都是不同的。想象一下，发生在《武林外传》中同福客栈的几段对话。

![对话](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/conversation.png)

这段对话中实际上有三个上下文，这里的 ”菜“ 这个词出现了三次，但是实际上业务含义完全不同。

- 大嘴说去买菜，这里的菜被抽象出来应该是食材采购品，如果掌柜对这个菜进行管理，应该具有采购者、名称、采购商家、采购价等。
- 秀才说实习生把账单中的菜算错了价格，秀才需要对账单进行管理，这里的菜应该指的账单科目，现实中一般是会计科目。
- 老白说的客人点了一个酱鸭，这里老白关注的是订单下面的订单项，订单项包含的属性有价格、数量、小计、折扣等信息。

实际上，还有一个隐藏的模型——上架中商品。掌柜需要添加菜品到菜单中，客人才能点，这个商品就是我们平时一般概念上的商品。

我们把语言再次统一，得到新的模型。

![DDD v3](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-v3.png)

4个被红色虚线框起来的区域中，我们都可以使用 ”菜品“ 这个词汇（尽量不要这么做），但大家都明确 ”菜品“ 具有不同的含义。这个区域被叫做 **上下文**。当然上下文不只是由二义性决定的，还有可能是完全不相干的概念产生，例如订单和座位实际概念上并没有强烈的关联关系，我们在谈座位的时候完全在谈别的东西，所以座位也应该是单独的上下文。

识别上下文的边界是 DDD 中最难得一部分，同时上下文边界是由业务变化动态变化的，我们把识别出边界的上下文叫做**限界上下文（Bounded Context）**。限界上下文是一个非常有用的工具，限界上下文可以帮助我们识别出业务的边界，并做适当的拆分。

限界上下文的识别难以有一个明确的准则，上下文的边界非常模糊，需要有经验的工程师并充分讨论才能得到一个好的设计。同时需要注意，限界上下文的划分没有对错，只有是否合适。跨限界上下文之间模型的关联有本质的不同，我们用虚线标出，后面会聊到这种区别。

![DDD-v4](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-v4.png)

使用上下文之后，带来另外一个收获。模型之间本质上没有多对多关系，如果有，说明存在一个隐含的成员关系，这个关系没有被充分的分析出来，对后期的开发会造成非常大的困扰。

## 聚合根、实体、值对象

上面的模型，尤其是解决二义性这个问题之后，已经能在实际开发中很好地使用了。不过还是会有一些问题没有解决，实际开发中，每种模型的身份可能不太一样，订单项必须依赖订单的存在而存在，如果能在领域模型图中体现出来就更好了。

举个例子来说，当我们删除订单时候，订单项应该一起删除，订单项的存在必须依赖于订单的存在。这样业务逻辑是一致的和完整的，游离的订单项对我们来说没有意义，除非有特殊的业务需求存在。

为了解决这个问题，对待模型就不再是一视同仁了。我们将那相关性极强的领域模型放到一起考虑，数据的一致性必须解决，同时生命周期也需要保持同步，我们把这个集合叫做**聚合**。

聚合中需要选择一个代表负责和全局通信，类似于一个部门的接口人，这样就能确保数据保持一致。我们把这个模型叫做**聚合根**。当一个聚合业务足够简单时，聚合有可能只有一个模型组成，这个模型就是聚合根，常见的就是配置、日志相关的。

相对于非聚合根的模型，我们叫做**实体**。

![DDD-v5](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-v5.png)

我们把这个图完善一下，聚合之间也是用虚线链接，为聚合根标上橙色。识别聚合根需要一些技巧。

- 聚合根本质上也是实体，同属于领域模型，用于承载业务逻辑和系统状态。
- 实体的生命周期依附于聚合根，聚合根删除实体应该也需要被删除，保持系统一致性，避免游离的脏数据。
- 聚合根负责和其他聚合通信，因此聚合根往往具有一个全局唯一标识。例如，订单有订单 ID 和订单号，订单号为全局业务标识，订单 ID 为聚合内关联使用。聚合外使用订单号进行关联应用。

还有一类特殊的模型，这类模型只负责承载多个值的用处。在我们饭店的例子中，如果需要对账单支持多国货币，我们将纯数字的 `price` 字段修为 `Price` 类型。

```java

public clsss Price(){
    private String unit;
    private BigDecimal value;    
    
    public Price(String unit,BigDecimal value){
        this.unit = unit;
        this.value = value;
    }
}

```

价格这个模型，没有自己的生命周期，一旦被创建出来就无须修改，因为修改就改变了这个值本身。所以我们会给这类的对象一个构造方法，然后去除掉所有的 `setter` 方法。

我们把没有自己生命周期的模型，仅用来呈现多个字段的值的模型和对象，称作为**值对象**。

值对象一开始不是特备好理解，但是理解之后会让系统设计非常清晰。”地址“是一个显著的值对象。当订单发货后，地址中的某一个属性不应该被单独修改，因为被修改之后这个”地址“就不再是刚刚那个”地址“，判断地址是否相同我们会使用它的具体值：省、市、地、街道等。

值对象是相对于实体而言的，对比如下。

| 实体                   | 值对象                 |
| ---------------------- | ---------------------- |
| 有 ID 标识             | 无 ID 标识             |
| 有自己的生命周期       | 一经创建就不要修改     |
| 可以对实体进行管理     | 使用新的值对象替换     |
| 使用 ID 进行相等性比较 | 使用属性进行相等性比较 |

另外值得一提的是，一个模型被作为值对象还是实体看待不是一成不变的，某些情况下需要作为实体设计，但是在另外的条件下却最好作为值对象设计。

地址，在一个大型系统充满了二义性。

- 作为订单中的收货地址时，无需进行管理，只需要表达街道、门牌号等信息，应该作为值对象设计。为了避免歧义，可以重新命名为收货地址。
- 作为系统地理位置信息管理的情况中具有自己的生命周期，应该作为实体设计，并重命名为系统地址。
- 作为用户添加的自定义地址，用户可以根据 ID 进行管理，应该作为实体，并重命名为用户地址。

我们使用蓝色区别实体和聚合根，更新后的模型图如下：

![ddd-v6](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-v6.png)

虽然我们使用 E-R 的方式描述模型和模型之间的关系，但是这个 E-R 图使用了颜色、虚线，已经和传统的 E-R 图大不相同，把这种图暂时叫做 **CE-R** 图（Classified Entity Relationship）。DDD 没有规定如何画图，你可以使用其他任何画图的方法表达领域模型。

## 使用领域模型指导程序设计

在了解到 DDD 之前，到底该用一对多和多对多关系？RESTful API 设计时到底应该选哪一个对象作为资源地址，评价应该放到订单路径下还是单独出来？订单删除相关有多少对象应该纳入事务管理？

在没有领域模型之前，这些大概率凭借经验决定，当我们把领域模型设计出来之后，领域模型可以帮助我们做出这些指导。领域模型不只是为编写业务逻辑代码使用，这样对领域模型来说就太可惜了。

下面是领域模型指导软件开发的一些方面，具体细节后面会再逐个讨论。

### 指导数据库设计

通过 CE-R 图，我们明显可以设计出数据库了。不过还有一些细节需要注意。

首先，在之前的认知里面，多对多关系是非常正常的。但是通过对领域模型的分析后发现，传统处理多对多关系时，需要额外增加一张关联表，这张关联表本质上是一个”关系“的实体没有被发掘出来。否则，在实际开发中会造成系统耦合，以及使用 ORM 的时候产生困惑。

菜品和订单之间是多对多关系吗？

如果是，菜品和订单之间耦合了。实际上，菜品的管理处于系统操作的上游，菜品不依赖订单的任何操作，也就是说订单的任何变化菜品无需关心。

订单拥有多个订单项，每个订单项从菜品读入数据并拷贝，或者引用一个菜品的全局 ID （菜品在另外一个聚合）。这样在设计表结构时订单和订单项关联，订单项不关联菜品。订单项应该从程序读取菜品信息。看起来多对多的关系，被细致分析后，变成了一个一对多关系。

![数据库设计](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/database-design.png)

在使用 ORM 时，良好的领域模型尤其有用。不合适的关联关系不仅让 ORM 关联变得混乱，还会让 ORM 的性能变差。

使用领域模型建立数据库的要点：

- 留意多对多关系，并拆解成一对多关系
- 将外键放到实体上
- 值对象和实体为一对一关系，值对象中保存实体的主键即可，无需为值对象分配 id
- 使用 ORM 时，聚合根和实体可以配置为级联删除和更新

### 指导 API 设计

RESTful API 已经变成了主流 API 设计方式，当设计好领域对象后，设计 API 的难度大大降低。

使用聚合根作为 URI 的根路径，使用实体作为子路径。通过 ID 作为 Path 参数。

![API 设计](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/api-design.png)

值对象没有 ID，应该只能依附于某个实体的路径下做更新操作。

![API 设计 v2](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/api-design-v2.png)

另外根据这个关系，处理批量操作的时候应该在实体的上一级完成，例如批量添加订单的订单项，可以设计为：

```
POST /orders/{orderId}/items-batch
```

不要设计为:

```
POST /orders/{orderId}/items/batch
```

### 指导对象设计

在实践中过程中，像 Java、Typescript具有类型系统的语言，对象很容易被误用。如果 `User` 对象既被拿来当做数据库操作使用，又被拿来当做接口呈现使用，这个类最终变成了上帝类，存在大量可有可无的属性。

例如用户注册时候需要输入重复密码，如果在 `User` 对象中添加 `confirmPassword` 属性，存储时候确并不需要。

因此 DDD 中，数据库各种对象的使用应该针对不同的场景设计。回到我们上面说的技术复杂度和业务复杂度中来。领域模型解决业务复杂度的问题，领域模型只应该被用作处理业务逻辑，存储、业务表现都应该和领域模型无关。

![对象设计](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/objects-design.png)

简单来说，可以把这些 `Plain Object` 分为三类:

- DTO，和交互相关或者和后端、第三方服务对接
- Entity，数据库表映射
- Model，领域模型
  
另外，在使用领域模型使用上也需要额外注意

- 领域对象尽量使用组合的方式，而不是继承，现实业务逻辑中继承这种概念实际上很少。例如菜品的设计，有热菜、汤菜、凉菜，实际上这里面并不是菜的继承，而应该抽象出分类这个模型。
- 不要滥用领域模型，有些业务逻辑，实在找不出一个领域模型很正常，所以 DDD 中存在一个领域服务。例如，生成一个 UUID。有些业务逻辑不持有系统业务状态，Eric 的书中比喻为像加油站一样的业务逻辑。

### 指导代码组织

代码组织，通俗来说就是如何分包。一种狭义的对 DDD 的理解就是指按照 DDD 风格进行代码组织，虽然 DDD 的内容远不止于此。

在很长一段时间，我对 DDD 分包策略陷入困惑，后来我明白到，讨论 DDD 风格的分包，必须将单体引用和微服务应用分开考虑。

> 微服务应用在逻辑上和解耦良好的单体应用是一致的。

但是微服务是一种分布式架构，映射到单体应用中，各个包分布到不同的服务器中了。我们先以单体应用入手，最后再讨论如何将单体应用架构映射到到微服务中。

在事务脚本的模式中，我们一般将代码分为三层架构。DDD 特别的抽离出一层叫做 `application`。这一层是 DDD 的精华，领域模型关心业务逻辑，但是不关心业务场景。

`application` 用来隔离业务场景，显得非常重要。举个例子，用户被添加到系统中，领域模型处理的是：

1. 用户被添加
2. 授予基本权限
3. 积分规则创建
4. 账户创建（三户模型，客户、用户、账户往往分开）
5. 客户资料录入

但是，用户被添加到系统中由多个应用场景触发。

- 用户被邀请注册
- 用户自己注册
- 管理员添加用户

`application` 需要隔离应用场景，并组织调配领域服务，才能使得领域服务真正被复用。因此 `application` 需要承担事务管理、权限控制、数据校验和转换等操作。当领域服务被调用时，应该是纯粹业务逻辑，并与场景无关。

如果我们将三层架构和 DDD 架构对比，DDD 架构如右图所示。

![三层架构对比](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-vs.png)

我们将 DDD 的代码架构展开，可以看到更为细节的内容。 DDD 代码实现上需要 `Repository`、`Factory` 等概念，但这些是可选的，我们在后面具体讲代码结构的部分再阐述。

![单体DDD架构](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/ddd-mono.png)

我们再来看，DDD 的单体应用架构映射到微服务架构下会是怎么样的。

![单体到微服务](https://github.com/linksgo2011/dda-book/raw/master/images/02-DDD-foundamental/mono-micro-service.png)

微服务必须考虑到不再是一种服务器，`Domain` 层被抽离出来作为 `Domain Server` 存在，`Domain Server` 不关心业务场景，因此不需要 `application` 层。`Application Server` 需要 `Application` 层，`Domain` 层由后端的 `Domain Server` 提供。

另外补充，一些 DDD 代码组织的基本逻辑：

- 隔离业务复杂度和技术复杂度
- 使用接口隔离有必要的耦合和依赖倒置

领域建模工作坊
===


## 事件风暴工作坊简介

EventStorming 工作坊是 Alberto Brandolini 发明，一种用于领域驱动建模的工作坊形式，中文环境下又被称作为事件风暴工作坊。

工作坊的英文名称是 Workshop 通俗来说就是找一波人在一个大屋子里开会，进行研讨活动。和培训不同的是，工作坊一般没有讲师，也没有固定的内容。工作坊一般会有一个主持人，有些环境下又被称为催化师，负责工作坊的流程和推动，但内容是所有参与者共同完成的。

通过事件这个视角探索一个软件系统中的关键数据流动，然后提取和抽象出共同的对象或实体，从而为软件设计带来极大的便利。这是事件风暴工作坊的基本逻辑。例如通过和业务人员探索“订单已提交”、“支付已完成”、“用户已通知”，从而抽象出 “订单”、“支付”、“通知”等相关模型。

事件风暴工作坊的另外一个特点是，让业务专家和开发人聚到一起研讨软件的设计，它聚焦于业务而非具体的技术。如果亲临事件风暴工作坊的现场，你一定会被业务专家和开发人员的协作感到震惊。开发者往往选用 UML 作为建模的工具，然而哪怕是最轻量级 UML 工具也很难让多个人协同操作。另外，很少有业务人员能了解或者知道怎么使用 UML，因此 Alberto 在遇到这个问题时，采用了最简单直接的方法，用会议室的整面墙壁和便利贴代替屏幕和鼠标。

本篇描述的事件风暴工作坊是根据具体情况不断调整和改进过后的形式，并增加了为微服务设计相关的内容，已经和 Alberto 先生有较大出入。对 Alberto 先生的工作坊形式有兴趣可以访问  https://www.eventstorming.com/ 了解。


为微服务划分设计的事件工作坊基本分为三个阶段：准备工作、战略设计、战术设计

**准备工作**

1. 环境准备
2. 人员调度
3. 物料准备
4. 日程安排
5. DDD 导入培训
6. 人员 DDD 知识摸底
7. 检查准入条件

**战略设计**

1. 识别问题域
2. 业务场景分析
3. 事件风暴
4. 命令风暴
5. 识别领域模型
6. 识别限界上下文
7. 识别上下文关系

**战术设计**

1. 聚焦单个上下文，识别聚合
2. 识别聚合根、实体、值对象

特别的，为了承接事件风暴工作坊的产出，下面是一些微服务设计的后续工作，可以不在工作坊内完成：

1. 通过上下文关系进行微服务设计
2. 通过微服务设计API
3. 通过聚合设计数据库
4. 通过聚合设计代码包

## 工作坊准备

开始事件风暴工作坊之前，有一些准入条件，当这些条件不满足时，不应启动工作坊，强行启动可能不会带来有效的成果和收益。

下面是一份检查清单，稍后会详细解释：

- 工作坊的目标已经明确，确认当前工作坊是为了做重构还是设计目标方案
- 工作坊的范围已经明确
- 业务场景选定
    - 选择最长的业务场景
    - 可以由多个业务场景凭借
    - 选择业务重要性高的场景
    - 选择业务相关性强的场景
- 业务专家已经指定明确
- 技术专家已经指定明确
- 有初步的用户故事地图分析结果（可选）
- 澄清风险


## 战略设计

### 事件风暴

为什么要识别事件？程序=数据结构+算法，软件本身就是一台能通过特定算法并通过修改数据结构的虚拟机器。

因此软件系统可以理解为一种能通过特定响应外部操作进而修改自身状态的虚拟机器，这台机器的生命力来自于响应外部的操作。我们可以通过数据的流动来观察这台机器，那么“事件”就是系统状态改变的关键信息点。

下面是操作步骤：

1. 选定场景范围
2. 在墙上预留空间，根据场景在墙面顶部标出时间线
3. 识别事件，事件以完成时描述，例如 “订单已支付” 
4. 如果多个事件的触发存在条件，可以使用规则，规则代表系统中的一些条件。使用 “xxx规则” 描述
5. 按照时间线发散，补充所有事件
6. 完成一次事件梳理后，进行逆向检查，反方向验证事件是否合理。  
    1. “是否遗漏了事件？”
    2. “事件是否被默认合并了？例如订单已支付，应该还有支付已完成”
    3. “该事件是否一定需要存在？”
7. 重复以上步骤。

[TODO: 操作流程图]

事件风暴是整个工作坊中最重要的一部分。是否能合理的识别出事件，是工作坊成功地重要输入。其中，有一些概念难以理解，下面是我在经历大量实践的过程中总结的一些经验和要点：

- 事件可以理解为对系统状态的修改。
- 事件应该是独立和原子的。
- 事件使用完成时作为约定，标志着系统状态变化结束。例如：用户已注册。
- 避免使用无用的缀词，例如 XXX 已完成。
- 事件一定要前后匹配，这个环节挑战的是业务分析时逻辑是否严丝合缝。例如订单已完成，应该是由上一个事件支付已经完成触发的，否则开发时会缺少逻辑。
- 规则和事件的区别表现在系统中某个算法、逻辑判断还是数据状态的修改。例如验证通过注册成功，这其中包含了一个验证的规则，和用户已注册的事件。
- 这个阶段的事件和业务相关，暂时不要把技术带入
- 如果某些信息在场人员知识不足，或无法做出决定可以先跳过
- 完成一遍事件梳理后进行“逆向检查”
- 忽略现有实现，分析的是现有业务
- 任何对系统状态没有修改的的行为都不算事件。例如显示列表、鼠标滚动。除非系统需要关注用户行为，那么鼠标滚动就成为了事件，需要被考虑和建模。
- 注意不要遗忘和外部系统交互的事件，例如商品是从外部系统同步过来的，应该有“商品已获取”
- 如果是调用外部系统数据，应该使用 “xxx 数据已获取”。表明系统的状态被改变。


这个环节往往占据了这个事件风暴大部分时间，如何确认这个环节的产出是有效的呢？

在后面的环节中可以总结一些经验：

- 一个场景的事件应该是完整自洽的，订单已支付必然存在未支付的情况，通过正反面分析问题。
- 抓住规则，穷尽这个规则后面的所有的事件，例如订单支付成功、不成功、取消、超时等各种情况
- 抓住事件的前后关联，留意伴生事件，例如用户支付成功会不会有消息发送的情况。

最后，我们的工作坊应该是迭代式的，这个环节尽可能挖掘所有的事件，但是现实中即使富有经验的领域专家和业务专家，也无法因此穷尽所有的事件。因此我们必须承认我们会在后面的环节对事件进行调整，因此做好一定的准备。

### 命令风暴

## 战术设计





微服务系统设计
===

微服务划分的参考原则

- 可拆可不拆的，先不拆
- 参考限界上下文作为业务边界
- 参考弹性边界作为云环境下的伸缩边界
- 参考子域划分
- 参考领域模型的设计结果，以及依赖情况，和事务边界对齐
- 结合未来团队、架构组织方式
- 结合 devops 能力
- 和变化对其

## API 设计

API 本身的含义指应用程序接口，包括所依赖的库、平台、操作系统提供的能力都可以叫做 API。我们在讨论微服务场景下的 API 设计都是指 WEB API，一般的实现有 RESTful、RPC等。API 代表了一个微服务实例对外提供的能力，因此 API 的传输格式（XML、JSON）对我们在设计 API 时的影响并不大。

API 设计时微服务设计中非常重要的环节，代表服务之间交互的方式，会影响服务之间的集成。
通常来说，一个好的 API 设计需要满足两个主要的目的：

- **平台独立性。** 任何客户端都能消费 API，而不需要关注系统内部实现。API 应该使用标准的协议和消息格式对外部提供服务。传输协议和传输格式不应该侵入到业务逻辑中，也就是系统应该具备随时支持不同传输协议和消息格式的能力。

- **系统可靠性。** 在 API 已经被发布和非 API 版本改变的情况下，API 应该对契约负责，不应该导致数据格式发生破坏性的修改。在 API 需要重大更新时，使用版本升级的方式修改，并对旧版本预留下线时间窗口。

### API 设计的原则

实践中发现，API 设计是一件很难的事情，同时也很难衡量设计是否优秀。根据系统设计和消费者的角度，给出了一些简单的设计原则。

### 使用成熟度合适的 RESTful API

RESTful 风格的 API 具有一些天然的优势，例如通过 HTTP 协议降低了客户端的耦合，具有极好的开放性。因此越来越多的开发者使用 RESTful 这种风格设计 API，但是 RESTful 只能算是一个设计思想或理念，不是一个 API 规范，没有一些具体的约束条件。

因此在设计 RESTful 风格的 API 时候，需要参考 RESTful 成熟度模型。


| 成熟度等级             | 解释           | 示例           |
| -------------- | ------------------ | ------------------ |
| Level 0           | 定义一个根 URI，所有的操作通过 POST 请求完成   | `POST /?action=changeUserPassword` |
| Level 1 | 创建独立的资源地址，隔离 API 范围 | `POST /user?action=update` |
| Level 2     |   使用 HTTP 动词定义对资源的操作       | `GET /users/001`        |
| Level 3     | 使用 API 超媒体（HATEOAS，返回的 body 中索引相关的资源地址 ）        |    `{"links":["/users/","/products/"]}`     |

根据自己的应用场景选择对应的成熟度模型，一般来说系统成熟度模型在 Level 2 左右。

### 避免简单封装

API 应该服务业务能力的封装，避免简单封装让 API 彻底变成了数据库操作接口。例如标记订单状态为已支付，应该提供形如 `POST /orders/1/pay` 这样的API。而非 `PATCH /orders/1`，然后通过具体的字段更新订单。

因为订单支付是有具体的业务逻辑，可能涉及到大量复杂的操作，使用简单的更新操作将业务逻辑泄漏到系统之外。同时系统外也需要知道 `订单状态` 这个内部使用的字段。

更重要的是，破坏了业务逻辑的封装，同时也会影响其他非功能需求。例如，权限控制、日志记录、通知等。

### 关注点分离 

**好的接口应该做到不多东西，不少东西。** 怎么理解呢？在用户修改密码和修改个人资料的场景中，这两个操作看起来很类似，然后设计 API 的时候使用了一个通用的 `/users/1/udpate` URI。

然后定义了一个对象，这个对象可能直接使用了 `User` 这个类：

```json

{
  "username": "用户名",
  "password": "密码"
}

```
这个对象在修改用户名的时候， `password` 是不必要的，但是在修改密码的操作中，一个 `password` 字段却不够用了，可能还需要 `confirmPassword`。

于是这个接口变成：

```json

{
  "username": "用户名",
  "password":"密码",
  "confirmPassword":"重复密码"
}

```
这种类的复用会给后续维护的开发者带来困惑，同时对消费者也非常不友好。合理的设计应该是两个分离的 API：

```json

// POST /users/{userId}/password

{
  "password":"密码",
  "confirmPassword":"重复密码"
}

```

```json

// PATCH /users/{userId}

{
  "username":"用户名",
  "xxxx":"其他可更新的字段"
}

```
对应的实现，在 Java 中需要定义两个 DTO，分别处理不同的接口。这也体现了面向对象思想中的关注点分离。

### 完全穷尽,彼此独立

API 之间尽量遵守完全穷尽，彼此独立 (MECE) 原则，不应该提供相互叠加的 API。例如订单和订单项这两个资源，如果提供了形如 `PUT /orders/1/order-items/1` 这样的接口去修改订单项，接口 `PUT /orders/1` 就不应该具备处理某一个 `order-item` 的能力。

这样的好处是不会存在重复的 API，造成维护和理解上的复杂性。如何做到完全穷尽和彼此独立呢？

简单的方法是使用一个表格设计 API，标出每个 URI 具备的能力。

| URI             | 业务能力           | 
| -------------- | ------------------ |
|     /orders/1  | 对订单本身操作   | 
|     /orders/1/order-items  | 对订单项操作   | 

资源 URL 设计来源于 DDD 领域建模就非常简单了，聚合根作为根 URL，实体作为二级 URI 设计。聚合根之间应该彻底没有任何联系，实体和聚合根之间的责任应该明确。

产生这类问题的根源还是缺乏合理的抽象。如果存在 API 中可以通过用户组操作用户，通过用户的 URI 操作用户属于的用户组，这其中的问题是缺少了成员这一概念。用户组下面的本质上并不是用户，而是用户和用户组的关系，即成员。

### 版本化

一个对外开放的服务，极大的概率会发生变化。业务变化可能修改 API 参数或响应数据结构，以及资源之间的关系。一般来说，字段的增加不会影响旧的客户端运行。但是当存在一些破坏性修改时，就需要使用新的版本将数据导向到新的资源地址。

版本信息的传输，可以通过下面几种方式

- URI 前缀
- Header
- Query 

比较推荐的做法是使用 URI 前缀，例如 `/v1/users/` 表达获取 `v1` 版本下的用户列表。

常见的反模式是通过增加 URI 后缀来实现的，例如 `/users/1/updateV2`。这样做的缺陷是版本信息侵入到业务逻辑中，对路由的统一管理带来不便。

使用 Header 和 Query 发送版本信息则较为相似，不同之处在于，使用 URI 前缀在 MVC 框架中实现相对简单，只需要定义好路由即可。使用 Header 和 Query 还需要编写额外的拦截器。

### 合理命名

设计 API 时候的命名涉及多个地方：URI、请求参数、响应数据等。通常来说最主要，也是最难的一个是全局命名统一。

其次，命名需要注意这些：

- 尽可能和领域名词保持一致，例如聚合根、实体、事件等
- RESTful 设计的 URI 中使用名词复数
- 尽可能不要过度简写，例如将 `user` 简写成 `usr`
- 尽可能使用不需要编码的字符

用领域名词来对 API 设计命名不是一件特别难的事情。识别出的领域名词可以直接作为 URI 来使用。如果存在多个单词的连接可以使用中横线，例如 `/orders/1/order-items`

### 安全

安全是任何一项软件设计都必须要考虑的事情，对于 API 设计来说，暴露给内部系统的 API 和开放给外部系统的 API 略有不同。

内部系统，更多的是考虑是否足够健壮。对接收的数据有足够的验证，并给出错误信息，而不是什么信息都接收，然后内部业务逻辑应该边界值的影响变得莫名其妙。

而对于外部系统的 API 则有更多的挑战。

- 错误的调用方式
- 接口滥用
- 浏览器消费 API 时因安全漏洞导致的非法访问

所以设计 API 时应该考虑响应的应对措施。针对错误的调用方式，API 不应该进入业务处理流程，及时给出错误信息；对于接口滥用的情况，需要做一些限速的方案；对于一些浏览器消费者的问题，可以在让 API 返回一些安全增强头部，例如：X-XSS-Protection、Content-Security-Policy 等。

## API 设计评审清单

- URI 命名是否通过聚合根和实体统一
- URI 命名是否采用名词复数和连接线
- URI 命名是否都是单词小写
- URI 是否暴露了不必要的信息，例如 `/cgi-bin`
- URI 规则是否统一
- 资源提供的能力是否彼此独立
- URI 是否存在需要编码的字符
- 请求和返回的参数是否不多不少
- 资源的 ID 参数是否通过 PATH 参数传递
- 认证和授权信息是否暴露到 query 参数中
- 参数是否使用奇怪的缩写
- 参数和响应数据中的字段命名统一
- 是否存在无意义的对象包装 例如 `{"data":{}'}`
- 出错时是否破坏约定的数据结构
- 是否使用合适的状态码
- 是否使用合适的媒体类型
- 响应数据的单复是否和数据内容一致
- 响应头中是否有缓存信息
- 是否进行了版本管理
- 版本信息是否作为 URI 的前缀存在
- 是否提供 API 服务期限
- 是否提供了 API 返回所有 API 的索引
- 是否进行了认证和授权
- 是否采用 HTTPS
- 是否检查了非法参数
- 是否增加安全性的头部
- 是否有限流策略
- 是否支持 CORS
- 响应中的时间格式是否采用 `ISO 8601` 标准
- 是否存在越权访问






遗留系统的服务化改造
===




架构实践
===

## 某地区电信运营商

## 某船运公司

## 某物联网平台提供商


# 简介

本教程致力于介绍微服务架构的设计逻辑，力求清晰易懂，前后连贯。重点围绕解决微服务如何划分这一问题，并介绍如何应对微服务化后带来的其他问题。其中涉及到的技术、方法都是围绕这一思想，技术选型中具体的技术细节请参考相关文档。

教程适合对企业架构有一定了解的开发者、架构师，对架构感兴趣的读者也可参考。

致谢

- 文档构建基于 Pandoc
- 文档样式来源 [https://phodal.github.io/mifa](https://phodal.github.io/mifa)

参考资料
=== 
- https://www.ibm.com/developerworks/cn/java/j-cn-java-and-microservice-1st/index.html 微服务
- https://www.infoq.cn/article/microservices-intro
- https://microservices.io/
https://www.nginx.com/blog/introduction-to-microservices/
- https://xebia.com/blog/eventstorming-continuous-discovery-beyond-software-modelling/ EventStorming; Continuous discovery beyond software modelling
- https://www.jdon.com/50847 EventStorming 超越软件建模的持续发现
- https://herbertograca.com/2017/09/28/clean-architecture-standing-on-the-shoulders-of-giants/ Clean Architecture: Standing on the shoulders of giants
- https://www.jianshu.com/p/b565f0c00c0c 整洁架构(译)
- https://www.jianshu.com/p/eadbec49fbbc?from=timeline&isappinstalled=0 插图版：领域驱动的微服务架构设计工作坊实施步骤
- https://developer.github.com/v3/#current-version 
- https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design

