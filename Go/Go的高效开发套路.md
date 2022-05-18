# [Go的高效开发套路](https://segmentfault.com/a/1190000041709994)

## A. 背景

当前在公司进行Go服务端研发工作时，发现缺少Go开发的最佳实践，而导致以下现象

1. 用Go开发时会比较迷茫，不知如何下手，怎么开展工作比较高效。
2. 重复造轮子比较严重。
3. 项目的代码质量参差不齐，导致交付的产品质量参差不齐。
4. 产品运行黑盒，可观测性差，能跑就行。
5. 代码实现考验研发人员水平，但顶尖的毕竟是少数，往往比较差，而且顶尖也说不准会犯错。
6. 一个人负责整个功能开发，一旦人员离职，代码维护就会难上艰难。

而我在进行产品研发工作时，意识到后续产品的功能会愈加复杂，不注重这些问题的话，后续恐怕出现代码愈加难以维护、产品功能难以改动、用户需求无法快速满足的多米诺效应。
为了尽早制止该现象的发生，探索一套可以在Go领域进行最佳实践的范式，从而达到以下效果：

1. 代码易维护，起码有两个人熟知代码实现目标，也容易进行代码迭代更新。
2. 代码设计合理，避免过于奇葩、过度以及不全面的实现。
3. 产品质量保证，交付产品可观测性强、Bug率低。
4. 降低重复造轮子现象，专注于业务逻辑开发，提升研发效率。

故此从研发效能方面研究解法，探索从个人开发行为到团队协作上的提升，最终在研发代码质控以及研发框架两个方面去实践各种方法，最终总结出当前的高效开发的模式，并持续优化沉淀中，希望通过实践验证该模式的效果，推广至所有使用Go进行服务端开发的场景实现，提高整个团队研发效能。

## B. 模式简介

本模式有三个部分组成，分别为标准化研发规范、契约协作以及统一的开发框架。

- 标准化研发规范旨在研发人员在进行代码开发时能够从代码编程风格、Git提交规范、Review机制能达成共识契约并严格遵守，为产品的质量负责。
- 契约协作旨在研发人员与外部需要调用其服务的合作伙伴，形成以契约为准，双方工作开展不互相强依赖具体实现的协作模式，降低双方沟通成本。
- 统一的开发框架旨在约束研发人员使用统一的框架开发，沉淀各种场景实现用于优化并演进框架，达到开发时可快速复用场景、持续迭代优化各场景实现以及为未来更多场景做坚实的积累。

## C. 模式实践-Go服务端开发

> 当前在实际开发业务中都有相应的实践

### 标准化研发规范

#### 代码编程规范

在接手一些组件的开发时，看到代码的实现上存在风格迥异，阅读理解上比较困难，所以定下约束风格的编程规范，可以提升代码的可阅读性，代码的易维护性。
在探索Go代码编程规范中，业界有EffectiveGo和UberGoGuide等比较知名的编程规范。

- EffectiveGo作为Go官方出品的编程规范，几乎所有使用Go语言开发的程序员都有看过并从中获益，普适性高。
- UberGoGuide是业界中使用Go语言开发的佼佼者，其开源了很多高质量的开源软件，比如Zap；它所开源的编程规范也是在EffectiveGo以及其他编程规范作为基础上去扩展。

从综合考虑上，Uber有中文翻译版上手速度快，且其公司按照规范开源的软件质量实现也侧面证明了该规范的优越性，所以选择UberGoGuide作为指导编程规范，但并不作为唯一参考标准，也考虑可以根据实际运行的最佳实践作为相应变动。

#### Git提交规范

在团队协作开发组件时，查看到Git的提交信息很随意，仅仅简单描述几句，但完全不知道实际做了什么。在回溯代码问题时，不知道这个提交做了什么，对代码维护上造成一定阻碍。估计参考业界优秀统一规范Conventional Commit 1.0进行约束。

Conventional Commit 1.0，是一种规范提交信息的轻量级规约，它提供了一些易于理解的规则用于指导我们如何编写commit信息，因此这也很容易被机器进行解读，结合SemVer版本规范，可以很容易通过工具既能自动化管理版本号，也进一步约束开发人员对提交信息的严谨性，毕竟如果乱打commit信息会直接影响到版本的制定。

因此在实践中上，严格遵守Conventional Commit 1.0来进行提交信息的规定，对于提交信息不明确者也可以拒绝其代码提交。

#### Review机制

在原先的团队合作模式中，大部分是各自开发相应的模块并直接提交到Master分支，然后就发布了，这样的做法虽然速度很快但交付质量却很堪忧，毕竟它很考验研发人员的综合水平，但毕竟不是人人都是神，所以它无法避免到缺陷的数量增长，不仅对整体代码质量堪忧，甚至对交付产品的质量也无法保证，更甚会影响到整个团队的口碑。为了解决这些问题，引入CodeReview机制来管制代码合并到代码交付的过程，从而管控整个代码质量。

CodeView机制参与的人员角色有代码提交者（Commiter）以及代码审核者（Reviewer），代码提交者每次提交代码后，均需要由代码审核者进行代码，审核通过后才能合并代码进主分支，从而达到可发布的可能性。

这样的机制实行下，需要可以做到以下方面效果：

- 使研发人员在每次提交中，有其他视角去验证代码实现方案、解决思路，保证一定的客观性。
- 担任Reviewr角色的人员，需要在过程做到识别bugs、是否有逻辑问题还有是否有覆盖全边缘场景，这样可以保证实现的思路足够全面。

其好处列举如下：

- 知识共享：团队内部在进行Review就能知道代码的实现、使用的设计等进行共享，有利于团队代码水平的提高。
- 可以更早的发现bug：避免在功能推出后才发现Bug，做到及时止损。
- 确保合规性：程序员的背景不同导致各自的代码风格不同，但通过规范标准约束项目代码的合规性。
- 增强安全性：当有安全技术背景的专业人员参加到review中时，安全性的等级是非常高的。
- 增强团队合作意识：当团队成员一起去解决一个问题，这样有利于提升他们各自的OwnerShip思维以及团队的归属感。

当前也需要做到以下约定：

1. Reviewr和Commiter在一次提交上，严格意义上不能为同一个人。
2. Comment时需要是善意的就事论事，双方均需要以案例和严谨的逻辑推敲去进行讨论，而且只有Reviewr同意后才能终止Comment。
3. 提交代码前需要自身已经经过编译以及测试。
4. 一次提交尽量做到核心代码不超过400行的。
5. 一次Review尽量低于1小时。
6. 需要知会到Reviewr清晰知道这次提交的实现目的，最终期望是什么。
7. 如果害怕提交的代码写的不好，那就将代码实现达到自身认可的程度先。
8. 单次提交如果代码量过大或逻辑复杂可以设置两个以上的Reviewer进行Review。
9. Reviewer和Commiter存在连带责任，Commiter提交的代码在生产出现问题并出现价值损失时，Reviewer也需要承担次要责任。
10. Review机制很容易会被忽视并绕过，但其后果有时候是很惨痛的，所以需要团队人员一致认可并遵守，才能做得好。

### 契约协作

#### PB文件既契约模式

在开发人员进行API开发到完成后，往往都需要提供文档给别人去调用，这样的做法有以下缺点：

1. 接口文档交付缓慢，接口需求方需要等待接口文档给出才能进行开发，这样对于需求方来说会更加开发风险会增加。
2. 接口更新不及时，当开发人员在更改API时，90%的概率会忘记更新文档，导致第三方在调用时出现很多状况需要沟通，这样增加双方的沟通成本，降低了双方的开发效率。

为了解决以上问题，推行Protobuf文件（以下简称PB文件）既契约的模式。
该模式遵循Google API 指南，实现了对应通信协议支持，并且遵守了 gRPC API 使用 HTTP 映射功能进行 JSON/HTTP 的支持。因此可以做到通过定义PB文件即可定义出REST和RPC API，通过类似GoogleAPI的仓库方式进行API Schema的管理。

再者结合丰富的Protoc插件可以自动化Swagger文档或生成不同语言类型的代码，如下：

- Go HTTP API：通过protoc-gen-go-http插件进行生成Go版的Http相关代码
- Go gRPC API：通过protoc-gen-go-grpc插件生成Go版的gPRC相关代码
- Swagger文档：通过protoc-gen-openapiv2插件生成

因此，研发人员通过PB文件即可清晰知道API的定义，参数、返回内容等bin并能依据PB文件生成相应的代码（Server和Client）就可进行各自的开发，也可以直接生成Swagger文档来查看使用，极大的降低了沟通的成本。

该模式也需要遵守以下规则：

1. 新契约变更需要通知到对方
2. 已运行一段时间的旧契约遇到BreakChange需求，应考虑新起契约，不应在原有基础改动。
3. 新契约定制时需先通过需求方Review后，再继续开发，避免返工，也保证需求实现准确性。
4. 契约尽量以清晰简洁明了的结构体定义为优先，对于特殊需求再使用Map、PBValue等。

### 统一的开发框架

> 对于框架的入门使用有另外一篇文章进行指导

#### 开发框架选型

在过往的编程经验中，对一个框架的积累沉淀有助于提升整体的开发效能，毕竟前人栽树后人乘凉，前人已经摸清楚框架的使用，就有足够的信心指导后人以此进行开发、优化框架代码以及建立起整套的周边生态，可以有余力应付未来的复杂业务场景。

因当前团队人力有限仅4人，即使有心也无力去做到自研框架这种需要大量脑力心力且需要积累沉淀的工作，于是放弃自研框架，选择借力开源产品。

在从框架选型上，从当前团队需求场景考量，团队需要的是一套可以约束实现规范、生态完善、用户自主性强、后台性能要求不高以及支持微服务化的框架。

在Go这边选型中预选了GVA、Gin和Kratos作为比对并进行实践，总结GVA、Gin以及Kratos三者比对如下

1. Gin, 为一个基础轻量级高性能的Web框架，实现场景案例较多，但代码实现无约束规范需要靠开发者自己摸索，普适性较强。
2. GVA，是一个前后端配套的框架，主要解决是快速开发web应用，但框架代码实现上并没有很好的理论指导以及规范约束，整体使用后的代码质量较差难维护，应对复杂业务需求能力差，最让人诟病的是滥用的公共变量。
3. Kratos，bilibli业务验证出品，配套健全的微服务框架，设计上遵循整洁架构以及DDD思想，实现的模块耦合度低，用户自主性强，应对复杂的业务需求能力强，但总体设计理念上倾向于标准规范定制，整体性能方面中上，需自己扩展模块实现更高性能。

|  框架  |     类型      | 协议支持  | 扩展性 | 指导力 | 易维护 |  生态  | 性能 | 框架代码质量 | GitHubStar数 |
| :----: | :-----------: | :-------: | :----: | :----: | :----: | :----: | :--: | :----------: | :----------: |
|  GVA   | Web前后端框架 |   HTTP    |   差   |   弱   |   弱   | 不完善 |  低  |      低      |    11.8k     |
|  Gin   | 后端HTTP框架  |   HTTP    |   强   |   弱   |   中   |  完善  |  高  |      高      |    56.1k     |
| Kratos |  微服务框架   | HTTP/gRPC |   强   |   强   |   强   |  完善  |  中  |      高      |    16.9k     |

最终采用Kratos作为统一的开发框架，具体决策依据如下

1. 多协议：默认支持HTTP/gRPC，可自主按照规范定制扩展，支持采用gin来实现transport核心。
2. API契约理念：支持PB文件定义，可生成服务端、客户端代码以及Swagger文档，并提供在线实时文档功能。
3. DDD思想分层开发：指导开发人员进行开发时，能按照领域驱动的方式进行开发，让功能更加聚焦，实现更加精炼，且耦合度低，比如将ClickHouse替换为SLS时只需要改动基础设施层实现即可。
4. 框架结构模块化清晰：几乎涵盖微服务框架开发中所涉及的模块，比如认证、注册、自监控等，并这些模块均可自定义实现，适配性强。
5. 高扩展性的中间件设计：极强扩展性的中间件设计，灵活扩展各种业务所需中间件模块。
6. 框架代码质量：总体代码质量上层，模块依赖解耦分明，用户阅读上手容易，自主定制优化可行。
7. 性能方面：现在业务暂时或将来1~2年均以Web后端应用以及微服务为主，所以性能要求方面不苛刻，且如需高性能时也可自主实现扩展模块实现。

#### 开发框架沉淀

当前组内已在Kratos框架有一定的积累，为后续提高后端开发提高很好的基础，具体如下：

1. Layout快速开发：依靠Kratos的Layout能力，定制符合自身的Layout模板，在后续开发中可以快速使用，当前Layout实现上已包含各层的写法模板、Prom指标集成、Opentelemetry全链路集成、本地缓存实现等。
2. 自定义API文档信息的写法指导：已充分踩坑如何丰富API文档，对于任何自定义信息需求均能满足。
3. 快速使用Kratos开发的指南：配合Layout，助力快速上手框架，快速进行业务开发。
4. 通过在线API文档生成前端SDK：通过Swagger-gen快速生成前端SDK，前端无需手写SDK，可维护强。
   之后也会沉淀更多的场景到框架和文档中，丰富框架的使用场景，以助力开发提效。

#### 实践效果

当前将该模式在内部项目中实践，有以下效果提升

1. Bug情况：在经过次实践后代码级别的bug数量从项目开始到现在低于10个，需求级别Bug数量低于15个。
2. 前后端协作效率：前后端当前沟通协商基本只在需求协商、协议定义和实际联调阶段进行协商，整体交付功能速度均低于2个星期。
3. 特性完成速度：后端服务交付实现特性，均低于5天一周期，项目进展卡点不在后端实现上。
4. 团队合作氛围：review机制的加入，让团队成员更加富有凝聚力以及更愿意做知识分享，提升了团队人员整体能力。
5. 交付服务自监控完善：落地全链路监控，提升前后端全链路可观测性，定位问题低于5分钟内。

总体的效果是符合预期设计的，但还有许多优化的空间比如API管理、Error规范、CICD标准化，将继续沉淀优化该模式，提升后续特性以及未来新项目的交付速率，提升开发人员幸福感，降低开发成本。

## Reference

- [https://github.com/uber-go/gu...](https://link.segmentfault.com/?enc=Hj4uLAm4t4z1pzCebiSvjg%3D%3D.yj3cTDlZ7V1t%2B3NaXX8XqRgH%2Fzv6wlohY4xLr%2BzQ81zuz4DREZaZqQxBYoUTuc2kTfro8oH7FjGixU26IuHmZg%3D%3D)
- [https://go.dev/doc/effective_go](https://link.segmentfault.com/?enc=tsycZ3U%2BVpsml3ig2c26FA%3D%3D.ySwhSxTPcz1UsMNnc7GSschr9bdh8u%2FWJilTD6d1KQU%3D)
- [https://www.conventionalcommi...](https://link.segmentfault.com/?enc=0P%2B5qiT26%2B7sz%2FBDvtQbcQ%3D%3D.LXqm4fNUtItl4wDfGbX6pe87zNLR0jLFhFYeDs6yItPMT264L%2FvN4di3elp2IWD7)
- [https://semver.org/](https://link.segmentfault.com/?enc=musSKhZSWLMCS7gXCFYxvw%3D%3D.NmtZNpvXal%2BH%2BKqrd5kgG8jMLbSP5QkV9fa8qc00lOM%3D)
- [https://www.bookstack.cn/read...](https://link.segmentfault.com/?enc=z%2BItacpaOGuNTOonDkk0pw%3D%3D.bzaPAKCb%2Fenw0oQULAEPMiPuDTY7ZsObod%2FieORS%2Bjbc0TZtbiCetrFfOYCGvGc0mdCT%2BYM4yiI0y7CWrmi4xJLyTAQF%2FXG8A64iD2W2Mt8%3D)
- [https://google.aip.dev/](https://link.segmentfault.com/?enc=9Iwdx%2FfqepBLIqAv2yrM5Q%3D%3D.qrAxE%2BSbL72MsYuvPpR9kHWUS%2F%2FndH6%2Bh1ELltJ3e5k%3D)
- [https://developers.google.com...](https://link.segmentfault.com/?enc=KrYAbxM7%2F%2BAb54rGYPgBIw%3D%3D.D6MFaGW1dCJFjuHPuRJjXoOoihTKkhLlcxjNziD%2FxLQnxzHkhqWL5eTDBFUqhScE%2BT80eTyYLjtj%2FIGuKKbOeA%3D%3D)
- [https://developers.google.com...](https://link.segmentfault.com/?enc=YAvfJGqncKhqPzgQmZONfA%3D%3D.HoeJ0us7IOTvCffL8L%2BNFdRxF8MOFwrqSAstyB%2BeFTxA4U414ym3UInJx3rmBBcHD1kzpHoMzBGnNb9i2ekrfg%3D%3D)
- [https://colobu.com/2017/03/16...](https://link.segmentfault.com/?enc=hk%2BlbfY%2BKYrW1crLbtRWDw%3D%3D.hF0el17IWfNISkD6AfoTGXUPoPgvQVVGZenKJJnJpO9pFjfuwrvkggbMx6qtrkH3pM2poAl3c67GAafu%2BPaj5g%3D%3D)
- [https://go-kratos.dev/docs/](https://link.segmentfault.com/?enc=y8VaB1fE3OJ6qhMIXFzOiQ%3D%3D.DZl9zWeiGUV0jxzeSFCoaD9PbkKl2PrShmefJI%2F3%2BAk%3D)
- [https://about.gitlab.com/topi...](https://link.segmentfault.com/?enc=KTwc3DKchNWkn4OcpFyMIg%3D%3D.nIHsY5QO2oPBz4HdW5ibjeKrXmaKkaYJosBKUoVS88wFB%2F4Idtc9aLOtJsQtUy5EMwQpwlPBas7uuCErsqYd7bBJ0X%2Fqq5xpsacHALAWLac%3D)
- [https://www.perforce.com/blog...](https://link.segmentfault.com/?enc=2s6U9eHzeyMCmed6RFpmlA%3D%3D.bQ3hAbGMBDOTxtRXZSN2WWr5Yl188X1hJyKd8tT9Ap9Gw6NtKLXPw1jHtdGkc3g%2F2T3ARgeiz70m3PhochokX2XCo5mMMaUlnY4iq7QTGmo%3D)