# 2.0版本新特性

## 群组架构
2.0版本中新增了群组架构，用于克服系统吞吐能力的瓶颈。

传统的区块链平台，整个网络维护一个账本，所有节点参与到这个账本的共识和存储，系统的吞吐能力无法横向扩展。

群组架构允许网络中存在多个不同的账本，每个账本是一个独立的小组，节点可以选择加入某些小组，参与到该组账本的共识和存储，随着小组数量的增加，系统的吞吐能力能够横向扩展。

群组架构中，各群组独立执行共识流程，这样的设计设计考虑了性能、隐私性和扩展性。如所有群组采用同一个共识模块，一方面共识模块会成为性能的瓶颈，在参与节点多，系统吞吐量大的情况下，共识计算无法扩展。 另一方面，如果所有参与机构都加入共识，相当于在共识环节打破了群组边界，群组内的交易被其他机构知晓，如果只由部分“权威”机构参与共识，则存在中心化的风险。所以，按群组的粒度划分，由群组内参与者决定如何进行共识，一个群组内的共识不受其他群组影响，群组内维护自己的交易事务和数据，使得各群组之间解除耦合，独立运作，也便于进行横向扩展。

群组架构也有别与传统的多链架构，传统多链架构中，每条链在物理上和逻辑上都相互独立，一个节点只能参与其中一条链，在运维和管理上都导致成本成倍增加。
群组架构的设计，一个组相当于一条链，实现了传统多链的扩展目的，同时一个节点可以参与到多条链，能够极大地简化运维复杂度，降低管理成本。

另外，FISCO BCOS还将持续基于群组架构，实现动态管理和跨链服务，实现企业间建立联盟和组建链像建“聊天群”一样便利。

更多的群组介绍，请参考[群组架构设计文档](./design/architecture/group.md)和[群组使用教程](./tutorial/group_use_cases.md)

## 分布式存储
2.0版本中新增了对分布式数据存储的支持，克服了本地化数据存储的诸多限制。

1.0版本中，节点采用LevelDB将数据存储于本地，这种模式受限于本地磁盘大小，当业务量增大时数据膨胀，数据迁移也是一个非常复杂的问题，业界大部分的区块链平台也面临类似问题。

通过支持节点将数据存储在远端分布式系统中，分布式存储方案有以下优点：
- 支持多种存储引擎，选用高可用的分布式存储系统，可以支持数据简便快速地扩容；
- 将计算和数据隔离，节点故障不会导致数据异常；
- 数据在远端存储，数据可以在更安全的隔离区存储，这在很多场景中非常有意义；
- 分布式存储不仅支持Key-Value形式，还支持SQL方式，使得业务开发更为简便；
- 世界状态的存储也从原来的MPT存储结构转为分布式存储，避免了世界状态急剧膨胀导致性能下降的问题。
- 优化了数据存储的结构，更节约存储空间，存取效率更高。

同时，2.0版本仍然兼容1.0版本的本地存储模式。更多关于存储介绍，请参考[分布式存储设计文档](./design/storage/index.html)

## 预编译合约
2.0版本中新增了对预编译合约的支持，大幅提升了智能合约的执行效率。

FISCO BCOS 1.0从以太坊技术体系继承而来，合约采用Solidity编写。
Solidity合约需要通过特殊的编译器（Solc）编译成二进制字节码以及ABI（接口描述文件），二进制字节码部署在链上，由EVM（以太坊虚拟机）解释执行。
Solidity具有很多优良的特性，比如图灵完备、可扩展性强等等。但是Solidity合约也有一些缺陷，包括执行效率较低，部署流程复杂等。

2.0实现了一套在底层通过C++实现的预编译合约框架，可以方便扩展用户所需的合约。预编译合约执行效率远远高于Solidity合约，突破了EVM的执行性能限制。
同时，预编译合约兼容Solidity的调用方式，使用方式保持一致。

FISCO BCOS 2.0的所有系统合约已经采用预编译合约方式实现，天然集成在底层平台，无需用户手动部署。
另外，还有类似CRUD操作等也由预编译合约实现，更多预编译合约的介绍，请参考[预编译设计文档](./design/virtual_machine/precompiled.md)和[预编译合约开发文档](./manual/smart_contract.html#id2)

## 合约CRUD接口
2.0版本中新增了符合CRUD接口的合约接口规范，简化了将主流的面向SQL设计的商业应用迁移到区块链上的成本。

1.0版本的合约，数据存储在合约的成员变量，例如采用Map/List等形式，合约的逻辑直接面向这些存储变量。
2.0版本基于预编译合约，实现了一套CRUD基本数据访问接口规范合约，基于CRUD接口编写业务合约，实现传统面向SQL方式的业务开发流程，这种方式有下面几个好处：
- 与传统业务开发模式类似，简化了合约开发学习成本；
- 合约只需关系核心逻辑，存储与计算分离，方便合约升级；
- CRUD底层逻辑基于预编译合约实现，数据存储采用[分布式存储](./design/storage/storage.md)，效率更高；

同时，2.0版本仍然兼容1.0版本的合约，更多关于CRUD接口的介绍，请参考[使用CRUD接口](./manual/smart_contract.html#crud)

## 并行交易处理
2.0版本中新增了合约交易的并行处理机制，进一步提升了合约的并发吞吐量。

1.0版本以及大部分业界传统区块链平台，交易是被打包成一个区块，在一个区块中交易顺序串行执行的。
2.0版本基于预编译合约，实现一套并行交易处理模型，基于这个模型可以自定义交易互斥变量。
在区块执行过程中，系统将会根据交易互斥变量自动构建交易依赖关系图——DAG，基于DAG并行执行交易，最好情况下性能可提升数倍（取决于CPU核数）。

正在紧张开发测试中，将在2.0后续版本中加入。

## 虚拟机
2.0版本引入了最新的以太坊虚拟机版本，支持Solidity 0.5版本。同时，引入了EVMC扩展框架，支持扩展不同虚拟机引擎。
底层内部集成支持interpreter虚拟机，未来可扩展支持WASM/JIT等虚拟机。

更多关于虚拟机的介绍，请参考[虚拟机设计文档](./design/virtual_machine/index.html)

## 控制台
2.0版本新增控制台，便于用户快速接入使用区块链功能。

控制台是2.0重要的交互式客户端工具，它拥有丰富的命令，可以查询区块链状态，管理区块链节点，部署并调用合约等功能。
相比于传统的nodejs等脚本工具，控制台安装简单、使用体验更好。详细请查看[控制台使用手册](./manual/console.md)

## 密钥管理服务
2.0版本对落盘加密进行了重塑升级，开启落盘加密功能时，依赖KeyManager服务进行密钥管理，安全性更强。

KeyManager在Github开源发布，节点与KeyManager的交互协议是开放的，支持机构设计实现符合自身密钥管理规范的KeyManager服务，比如采用硬件加密机技术。
该部分更详细的文档请参考[使用文档](./manual/storage_security.md)和[设计文档](./design/features/storage_security.md)

## 准入控制
2.0版本对准入机制进行了重塑升级，包括网络准入机制和群组准入机制，在不同维度对链和数据访问进行安全控制。

采用新的权限控制体系，基于表进行访问权限的设计，另外还支持CA黑名单机制，可以实现对作恶/故障节点的屏蔽。
详情请查看[准入机制设计文档](./design/security_control/index.html)

## 异步事件
2.0版本同时支持交易上链异步通知、区块上链异步通知以及自定义的AMOP消息通知等机制。

## 模块重塑
2.0版本对核心模块进行升级重塑，进行模块化的单元测试和端对端集成测试，支持自动化持续集成和持续部署。

## Release note
请查看[v2.0.0](https://github.com/FISCO-BCOS/FISCO-BCOS/releases/tag/v2.0.0-rc1)
