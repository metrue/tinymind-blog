---
title: 简聊 Udacity 前端开发
date: 2018-09-17T09:07:28.322Z
---

## 概述

在 Udacity，我们大部分的前端项目采用的 React 作为开发基础框架，也有少部分项目采用 Angular 来开发，比如我们的[主站 udacity.com](https://udacity.com). 不过新开的项目基本上都采用 React 了。几乎所有的项目都是 ES6+ 了，本分项目采用 TypeScript 来开发。

## 开发
> 工具: Github, CircleCI, AWS, Slack, Jira, Confluence, GSuite. VSCode, Vim

### 基础
为了让开发变的容易，跨项目的开发人员贡献代码变得简单，我们尽可以的采用同一个脚手架工具来初始化项目, 这中方式可以带来很多好处，最重要的几个方面，比如

* 相同的工具链
* 类似的项目结构
* 一致的构建过程

这样开发人员就可以很容易的融入到不同项目的开发当中去，由于使用相同的工具链，开发人员不需要花时间去重新熟悉各种新工具，类似项目结构可以让开发人员快速的熟悉代码，可以缩短贡献代码的时间。一致的构建过程让整个持续集成和持续部署变得一致和简单. 最大限度的去掉了重复的劳动力消耗。

### 流程
简单的流程图差不多是这样子:

```
production         -------------------deploy--------deploy-->
					/ 					/				/
初始化(checkout)/                  / merge		  /
				/                  /				/
master ---------------------------------------->
			\        \       /					/
			  \		   \  / squash merge	  / squash merge
fetures      f1 ------\					/
						   f2 ------------
```

从 master 分支开出 feature 分支进行开发, 开发结束之后，提交 Pull Request 到 master, 并且指定相应的工程师进行 Code Review, 如果修改的代码会影响到其他区域（美国，欧洲…)， 那么应该找相应的工程师进行 Review 确保改动正确。在你的 PR 得到 LGTM (Looks Good to Me) 之后，你可以直接发一个 Squash Merge 到 master 分支，master 分支会自动构建和测试，成功之后自动部署到 Staging 环境，我们是没有全职的 QA, 一般在 Staging 环境之后，我们自己和 PM以及其他的 Stateholders 会进行测试，重要的发布会进行 Bugbash，一般在 Staging 环境之后如果没有什么问题，就会尽快 Merge到 production 分支，同样，production 分支上的任何改动都会进入CI,然后自动部署到生产环境.

### Code Review
任何的代码都需要被至少一个工程师 Review 过，而且必须通过 GitHub 的 Matrix 设定。 Code Review 的地位应该比写代码本身更重要，因为交付一个损害用户体验的代码带来的副作用远远大于不交付任何功能。随意给一个 LGTM (Looks Good to Me) 是很不负责任。尽可能的在 Pull Request 中去讨论代码的问题，在 IM 中讨论问题有两个缺点:
* IM中讨论问题大多数时候没有经过仔细思考
* 让代码和讨论分离
这样会导致的后果是不利于就事论事，其他没有在 IM thread 里面的人不了解讨论的具体情况，所以我们尽量在 Pull Request 中去讨论代码问题。

PR 不仅仅是去实现一个功能或者修复一个 bug 的方式，同时也是相互学习的重要途径，我们在 PR 中给其他同事提出更优的解决问题方案，更良好的代码风格；也同时从同事的 PR 中获取到我们可能不了解的知识.

## 测试
> 工具: Jest, Enzyme, Coveralls

目前我们大部分的前端项目采用的是 [Jest ](https://jestjs.io/) 和  [Enzyme](https://airbnb.io/enzyme/) 来进行测试，使用 [coveralls](https://coveralls.io) 作为 coverage 监控工具，一般要求单元测试的覆盖率 >= 50% , 而对于一些骨干服务如登录，注册，服务通知等服务一般需要 >= 70%.

## 监控
> 工具: New Relic, Airbrake

在前端的监控方面，我们主要采用了 [New Relic](https://newrelic.com/) 和 [Airbrake](https://airbrake.io/) 两个工具，New Relic 在网站性能方面监控方面具备全方位的支持，从网络，DOM 处理时间，页面渲染时间，Ajax 响应时间等各个维度都有很好的监测。而 Airbrake 主要页面异常的实时报警，在支付或者登录注册等场景中，Airbrake 的异常报警功能非常的有用，可以帮我们在很早期就可以修复一些测试没有覆盖到，在用户场景中却时有发生的错误。分析这两个工具的报告，对于我们开发更棒的产品体验具有很好的指导作用。

## Growth 数据
> 工具: Segment, Google Analytics, Chartio

Udacity 在 Growth Hacking 方面是我经历的公司里面做的较为精细的一个，可能是因为我之前的雇主要么是传统软件公司，要么是 To B 的公司的原因，在 Growth Hacking 方面并没有做到完全的用户数据指导，在 Udacity, 无论是 Growth Team, Data Team 还是我们 Engineering Team 都遵循着数据为王的原则，所以数据分析师作为一个特别的存在，承接着来自各个团结的数据规划需求。

所以任何一个和用户有交互的项目的启动，数据需求也作为了 PRD 的一部分，需要数据分析师和工程师团队去一起审核一下可行性。在工具上，我们主要采用了 [Segment](https://segment.com/) , [Google Analytics](https://analytics.google.com/analytics/web/#/) 以及 [Chartio](https://chartio.com) 来进行数据收集和分析.
