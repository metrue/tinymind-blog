---
title: 善用Git Hook 安全使用 GitHub
date: 2016-02-17T09:20:02.322Z
---

作为程序员，我们都应该对仔细的对待敏感信息，无论是自己的私人信息，还是公司的商业信息，轻微的泄露，有时候后果也许也会十分严重。那么如何最大限度的避免敏感信息的泄漏呢，当然首先我们让自己成为一个细心谨慎的工程师，当然我们也可以用一些简单的方法帮助我们防止敏感信息被泄露。

我是一个重度的 GitHub 用户，之前在 [Intel](www.intel.com) 和 [Synopsys](www.synopsys.com) 时候，虽然所在的组都没有使用 Git 作为版本管理工具，我自己的私有项目也会使用 Git 来管理，而同时有一些项目需要开源，我就将它们放到了 GitHub 上，我们都知道 GitHub 是程序员们最重要的开源协作社区了，当你的代码开放出去了之后，所有人都可以随时随地的 clone, force, 然后如果有同学热心的和你一起改进你的项目的时候，还会给你发 Pull Request, 这样你就可以 merge 他的代码了，这就是程序员们在 GitHub 上的写作方式。

开放，就意味着所有人可以浏览和获取，所有人当然就包括一些怪蜀黍啦，如果你有敏感信息嵌入在代码中，然后被这些怪蜀黍哪去做坏事，那后果可以很严重，特别像我这样长时间游走在很多项目中间，很多时候难免会不小心把一些不应该放的信息 hardcode 到代码中，当然当时也许只是为了调试方便或者其他原因，但是当开发结束，兴高采烈的提交代码的时候却很容易忘记了删掉这些敏感信息。git push 一敲，怪蜀黍们就在 GitHub 拿到这些信息，然后开始作恶了。

### 怎么办呢

在吃过这种亏了之后，我就想，我怎么才能避免自己再出这样的事情呢，除了让自己更仔细，遵循一定的代码管理原则，比如特定配置文件不放到版本管理系统，我还能做什么呢。我就想，我把 git 封装一下好了，把 git commit wrap 成自己的脚本，在脚本中做安全检查，比如关键字检测。这不是 hook 吗，我直接做一个 global 的 git hook 不就可以了吗。

我的目标是避免敏感信息被 commit， 那么应该做的 pre-commit 的 hook, 当然为了方便，我们应该建立一个全局的 pre-commit hook，也就是设置一个全局的 hook, 这样我们就可以避免在每一个 repo 下面都重复的建立相应的 hook, 所有的 repo 公用一些通用的 hook. 具体的做法是这样子。

1. 配置全局模版目录

```ruby
git config --global init.templatedir '~/.git-templates'
{%  endhighlight %}

这样之后，只用你运行了 <code> git init </code>, git 就会把这个 <code> ~/.git-templates </code> 的文件都复制到你的 repo 的 .git 目录下。

2. 创建全局的 hooks 目录

```ruby
mkdir -p ~/.git-templates/hooks
{%  endhighlight %}

3. 开始编写你的 hook

```ruby
cd ~/.git-templates/hooks
vim pre-commit
{%  endhighlight %}

由于我们想要的防止敏感信息被commit，所以我们需要建立的commit之前的checker，也就是 pre-commit hook

下面就是一个简单的 pre-commit 的原型.

```ruby
#!/bin/bash
#
# pre-commit hook
# to check if sensitive information added to source code
#

DANGER_WORDS=('aws' 'key' 'password' 'username' 'phone' 'email' 'ssh')

for i in ${DANGER_WORDS[@]}; do
  git diff | grep -i ${i}
  if [ $? = 0 ];then
    echo "there is DANGER information being commited, are you sure?"
    echo "ENTER yes/no"
    read YES_OR_NO
    if [ ${YES_OR_NO} = 'yes' ];then
      # do nothing
      echo 'commit is going on'
    else
      echo 'giving up commit'
      exit 0
    fi
  fi
done
{%  endhighlight %}

这样没建立一个新的 repo, 也就是 git init 的时候，这个 pre-commit 就会生效，如果你希望你的之前的 repo 也拥有这样的 hook, 你可以到所有的 repo 下面 运行

```ruby
git init
{%  endhighlight %}

这样一个简单的 pre-commit 就大功告成了，在你 commit 的时候帮你检查你是否添加了敏感信息。
