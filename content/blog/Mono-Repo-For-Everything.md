---
title: Multirepos 到 Monorepo
pubDate: 2024-01-04T21:12:04.322Z
---

当项目变多了之后，如何管理代码就变成了问题。虽然我业余项目几乎都是一个人来完成，当我看我的
GitHub 上面的个人的 repo 数量我还是惊呆了，`public` 和 `private` 的所有 repo
的数目竟然超过了250个，不过也难怪，就一个 [ExpenSee](https://expensee.app)
项目我都分成了好几个:

* expensee: iOS/iPad app 的代码
* www.expensee.app: 官方网站的代码

而另外一个项目(原名: giki.app) [DailyMemo](https://dailymemo.app) 就更多了:

* giki.ios: iOS/iPad app 的代码
* giki.infra: 基础设施的代码
* giki.extensions: 各种插件(Chrome Extension, Firfox Addon) 的代码
* giki.sdk: sdk 的代码
* www: 官方网站的代码
* giki.api: 后端 API 的代码

就两个项目，一下子就有 8 个 repo, 这无疑带来了很多的负担，毕竟维护一个 repo 本身其实保护很多事情: repo 创建，配置, CI/CD 创建, 维护等等.

多个项目让这些事情都成倍的增长，而且给日常工作带了非常多的上下文切换负担。 
本来就是一个人的团队，没有理由采用这么复杂的管理策略，所以我决定回归到最简单的方案: Monorepo. 
将所有的个人项目都合并到同一个 repo,
而且要保证所有的提交历史都保留着，因为我曾经说过:

> 如果有一天我们老了，在我们的设备的某个目录不小心键入git log，也许我们会泪流满面，久久不止.

所以提交历史是我个人非常珍重的东西，另外，我还希望所有的 issues
也可以移动到了最终的 Monorepo 中。

简单研究了之后，其实整个过程还是蛮简单的，整体可以分为两个部分: Multirepo
合到 Monorepo 和 把各个 repo 的 issues 移动到同一个 repo 里.

## Multirepo to Monorepo

假设我们有 A 和 B 两个独立的 repo,
那么可以通过如下方式可以将他们合并到同一个repo
中，为了保持独立，我们把它们放到不懂的目录下面.

我们可以在 A repo 上进行如下操作.

```sh
cd A;                   # 比如 A repo 有 a1, a2 文件
mkdir a;                # 创建一个子目录
git mv a1 a/;
git mv a2 a/;
git commit -m "chore(mono): move current repo to sub dir"
git push;               # 我们这边直接 push 到master, 你可以开一个 Merge Request 来进行.
```

然后我们这时候可以到 B repo 上进行.

```sh
cd B;                           # 比如 B repo 有 b1, b2 文件
git checkout -b prepare_mono    # 创建一个 branch 来进行准备
mkdir b;                        # 创建一个子目录
git mv b1 b/;                   # 将文件移动到子目录中
git mv b2 b/;
git commit -m "chore(mono): move current repo to sub dir"
```

然后我们可以回到 A repo 上，然后将 B repo 集成进来.

```sh
cd A;
git remote add -f A <path to B repo on your local>;     # 把 B repo 增加成其中的一个 remote  
git checkout -b integate_B;                             # 让我们在一个单独的分之来进行把
git merge A/prepare_mono --allow-unrelated-histories;   # 然后把 B repo 的commits 集成进来
git push                                                # 然后 push 到remote branch 
```
然后我们就可以在 Github 的页面来开一个 Pull Request，然后看看是不是都把 B 的
commits 都囊括进来了，这里我们将才用 Merge Request, 而不是 Sqaush Merge.

## Transfer Issues

而 Github 的 issues 的移动，我借助的是 Github 的官方命令行工具[gh](https://cli.github.com/). 安装也很简单.

```sh
brew install gh     # 在 Mac 上直接用 brew 安装
gh auth login       # 然后登陆一下你的账号
```

比如我们要把 B repo 上的所有的 issues 移动到 A repo，那么可以这样.

```sh
cd B;        # 切换到 B 的本地repo目录
gh issue list -S "is:open is:issue archived:false" \
    --json number  | \
    jq -r '.[] | .number' | \
    xargs -I% gh issue transfer % https://github.com/<your gihub id>/<your target Repo>
```

自此，你基本上就完成了 Multirepo 到 Monorepo 的迁移，不过其实你过程应该会遇到一些问题需要去解决:

* 比如 Github Actions 的修复，因为我们很多时候 actions 的运行和目录结构有些关系
* 比如 Github Actions 需要用的一些 secrets，由于之前是多个repo分散设置，现在我们需要集中迁移

不过以后就开心了。
