---
title: 近几个月的工作内容
date: 2018-10-17
categories: Program
tags: [Program]
---

这几个月的时间主要围绕着游戏框架的外围功能，完善各个部分的工具链，不得不说Python确实是个好东西。
主要的内容如下：

#### 1、配置表导出（excel -> binaray/json/xml）
包含基础类型的序列化，还有复杂类型struct，array，dictionary，array&lt;struct&gt;，dictionary&lt;key, struct&gt;。
游戏中主要用到的是binaray，但这不方便策划动态的修改或者利用svn比对之前的修改，需要以字符串的格式来存储（这是一开始没有考虑到的）。

#### 2、AssetBundle资源打包
自己实现了一套资源打包的策略。不同于Unity 5.x及以上通过设置AssetBundle tag的方式（需要手工操作）来打包，而是设置一个Build目录，资源只要在Build目录就会打包，比较底层的依赖资源则放置在Build目录以外（以计算依赖资源的方式打包进去）。这样所有打包资源一目了然， 省去了手工操作的环节。不过同时带来的问题是依赖资源是以某种算法计算出来它属于哪个AssetBundle包，所以算法策略的选择很重要，这关系到后续热更新对AssetBundle包的校验，有可能在增加或者删除资源之后导致某个依赖资源从一个AssetBundle包移动到另外一个AssetBundle包。不过这样一来依赖需要自己管理AssetMainfest。

#### 3、网络层
其实网络层涉及到的内容不少，从协议文件（proto）转换为对应的类文件，每个协议类包含了自己的Encode和Decode实现。 同时还实现了Http和Socket两种通信机制。Socket通信，异步从buffer中获取数据时，要保证最小的内存分配（很多游戏框架就是随手new一个byte[], 然后调用Array.Copy(source-&gt;dest)）。Http通信，一般拿到的是JSON数据，需要反序列化为协议类的实例，而内存分配依然是很重要的考量依据。当然可以用第三方的JSON库，不过越是面向更普遍的需求，其性能就越低，越是面向特定的问题，就能把优化做到极致，所以选择一个合适的第三方库也不是那么随意。
网络层拿到数据需要派发到Model层，这里面隐含的一个问题是，在拿到巨量数据的时候，如何解决卡顿的问题。
socket协议解析过程大概有这么几步：
```buffer->bytes[]->deserialization->dispatch to model```
buffer到deserialization可以用缓存池和worker线程来缓解卡顿问题。dispatch to model可以用一个比较弹性的方式，每帧dispatch几条到model层。
大部分游戏框架不管拿到多少数据就是一通解，解完直接丢到逻辑层。单帧里面处理这么多内容必然造成卡顿。

#### 4、打包工具
利用Python（避免了学习不同平台的shell以及windows 批处理```难看的```语法）结合Jenkins做了两个打包的流程，一个是打包AssetBundle，一个是打包Apk，打包Apk的可操作性非常大，如果是对接巨量的渠道，那么节省的工作量是难以想象的，享受到一键出包的快感。Jenkins真是解放程序员双手的利器。

#### 5、想做但是没做成的
PSD to nGUI/ugui，市面上有很多插件和第三方工具，比如Fair GUI算是比较成熟的，不过它完全实现了一套自己的UI系统，现有的项目迁移过去比较麻烦。最大的难点在于让美术学习一套PS的图层命名规则，让别人学习并使用一套新的东西总是很麻烦。往往程序员自己写了一套工具给策划和美术使用，到最后都是自己在用。一部分原因在于工作流固定下来，要让大家跳出舒适区去用另外一套目前暂时看不到优势的东西比较困难，另一部分原因是工作内容的的安排使得持续开发并维护一个工具很难得到团队或者是老大的支持。

#### 6、未来想做的
**UI编辑器**，现在大部分国产游戏都可以算是UI游戏，抛开核心战斗，差不多只剩下UI了。UI基本上都是以事件进行驱动，定义好所有的事件（用户事件和逻辑事件），然后以拖拽的方式对Model层的数据进行增删改查，同时提供接口读取Config层的数据，基本数据结构以proto的方式定义，Editor的编写可以参考ShaderForge，写好了又是解放双手的一大利器。效果类似于Unreal的Blueprints（或者微软WPF），可以用拖拽的方式进行部分代码编辑，当然这其中涉及到的模块非常有限（UI、Model、Config），所以编码难度也会小很多。有几个难点，如何在已经载入的UI上通过鼠标点击来进行操作，然后需要对操作得到的数据进行序列化和反序列化。
**DSL的序列化与反序列化**（[Domain Specific Language](https://en.wikipedia.org/wiki/Domain-specific_language)）。任何特定的数据都可以用DSL来描述，JSON也算一种DSL，不过其数据表示仍然不够紧凑。可以根据特定的问题设计一种DSL，然后把这种DSL转换为运行时可识别的结构。考虑一种通用的方案，用JavaCC来扫描和解析这种DSL并转换为特定代码语言的解析器。
**思考各种能够流程化并且解放双手的环节**，积累各种工具，并且工具要保证通用性，同时尽量能够可视化，还要方便地嵌入到开发流程中。