---
title: 前端配置化项目思考
date: 2020-03-26 00:15:03
tags:
---
进入项目已经两个多月了，慢慢对项目有了一点感觉。结合同事的讲解看，总体来说方案还是很快捷的。只是感觉似乎还有些点不太完善，同时觉得这个方案做的早了一点，并且留的时间似乎不多，如果继续在上面加逻辑后面应该会比较臃肿。目前的一点想法，是早上洗澡的闲暇者无事想的，或许不是太完善先记录一点可能后面再深入熟悉会有一定完善。还有就是这个方案玩久了人是没有编程能力的，对于开发它的危害性是大于对业务的好处，况且它并不是对业务完全友好的。

## 目前的方案
### 总述
前端在平台上分为安卓、ios、web,主要功能还是以web的方式呈现，ios和安卓使用web容器展示。属于全球招聘计划，表单流程为主。目前的方案是通过配置实现前端页面渲染同时控制前后交互的逻辑，在前端这边定制好视图后扩展基本靠配置。这个部分是针对业务实现和扩展来做的，还有一部分是代码设计部分。从大的结构看，分为流程控制部分、异常部分、表单渲染及主题配置。

![概述](/imgs/flow-one.png?center)

前端使用的<code>react</code>所以整套方案最主要的是利用大量的<code>hook</code>去管理状态，并且通过<code>useContext</code>实现分层。来说说为什么选这样的方案，出发点是一套web代码多处使用，同时能快速通过配置扩展出一个市场。然而这种说法不知道多少人喊，好与不好再说吧。
### 从服务到配置到前端
#### 核心url
![基本关系](/imgs/cr_one.png?center)

在这个部分核心的依赖其实<code>makeBaseApi</code>这个块，它依赖访问的url中的参数构建一个基本的访问api。页面的最终渲染依赖于配置接口，流程查询接口，和数据支持接口三者缺一不可。首先进入页面会访问配置接口取出配置，配置里面包含<code>1.流程的控制配置以及对应页面的配置</code>,<code>2.页面渲染配置</code>,<code>3.接口调用配置</code>。从配置入手来看流程的处理方式，再说这样的方式约束性在哪里。图可能比较多，个人觉得这样比较直观一点吧。

![流程控制](/imgs/cr_two.png)

#### 状态的管理与使用
这里的状态其实分为两部分，依赖外部的（config接口）、内部控制的（当前提交数据和所处流程）。使用上游的数据来源都在一处-<code>context</code>，更新分两处，本地更新通过dispatch、外部更新通过next。由配置返回数据后，会把一些流程数据初始化到flowcontext本地，内部逻辑流转通过dispatch更改context再流回组件控制渲染，所以控制上其实条件更多的是依赖网络的另一方告诉，并且结构一定是完整的。说点题外话，大部分项目因为代码和业务的冲突没有处理好其实注定最后落不了地或者草草收尾的，这却又给了一些从业者学习的机会，它是好的！所以总需要存在这种情况。
1.第一个约束就是代码要适应配置的调整，配置变了代码其实就废了。这其实不太像配置化。
2.第二个矛盾在于这未必让业务扩展更快，当你的代码要适应配置改变而改变的时候，那就说明分支是没有用的，真正的分支控制由git转移到配置上了。在这上面因为对于配置依赖的方式决定了需要扩展出对配置的版本管理来避免多人开发的逻辑交叉。
3.交互方式与模块设计存在冲突，把流程控制抽出来放到npm，是要保持通用性实现多方依赖。将edit和model这类具象的状态引入其中，必然存在改页面逻辑需要去改npm包。污染公用控制层逻辑的同时更加大了冲突和异常的风险。




