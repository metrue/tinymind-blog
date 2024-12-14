---
title: 征服 Git Submodule 的正确姿势
date: 2015-04-26T10:17:04.322Z
---

大多数时候我们的工程中都会使用到其他的项目，也许是自己写的专用模块，也许是第三方的库。如果在自己的工程中对这些项目进行良好的管理呢，在使用 Git 进行版本控制的项目中，我们可以使用 Git Submodule 来进行优雅的管理。

### 使用方法

* 添加 submodule

假如我们想在已有工程中添加第三方项目，我们可以

```ruby
  git submodule add repo_url
```

初次使用 git submodule 的话，我们可以看到我们的工作目录下会有 .gitmodules, 这里记录的是我们工程用到的项目的 remote url， 以及他们的路径信息。为了统一管理我们的submodules，我们一般会建立一个统一的目录来存放这些submodules。所以一般可以这样做。

```ruby
  mkdir submodules
  cd submodules
  git submodule add repo_url
```

这样你可以看到你的 submodules 目录中已经自动 pull 出来了哪些 submodules 的代码，但是值得注意的，submodule 项目和我们的工程目录本质上是独立的 git repo。我们的工程只是记录他所以来的项目的版本信息而已, 你可以通过如下命令获取相关信息

```ruby
  git submodule
```

* 更新 submodule

为了和 submodule 项目保持同步更新，我们需要在主项目进行

```ruby
  git pull
```

之后进行一次

```ruby
  git submodule update
```

* 开发 submodule

有时候我们是在工程中直接对 submodule 进行开发的。但是也许你会发现当你 cd 到你的 submodule 的时候，你并不是在 master 分支上，而是处于一种游离的状态 'detached HEAD ...', 所以正确的做法是

```ruby
  git checkout master # 或者其他开发分支
  git commit -m "message" # 开发了之后可以提交
  git push
```

那么如果忘记了切换到你想要的分支而且你已经做了更新呢， 你可以这样

```ruby
  git checkout master
  git cherry-pick commit_id # 可能会有conflict，如果有，就先解决
  git push
```

* 删除 submodule

很遗憾的是，Git 并没有 git submodule rm 这样的操作，而且你单独的删掉 submodule 的子目录也是错误，正确的做法是

```ruby
  git rm --cached pato_to_submodule
  rm -rf path_to_submodule

  然后再 .gitmodules 中删掉相应信息.
  最后还得在 .git/config 中删掉其相关内容.
```

### 最后

当然很多时候，我们clone别人的项目的时候，如果别人也是使用了 git submodule 的话，我们需要通过下面的方法来 pull 所有的 submodules

```ruby
  git submodule init
  git submodule update --recursive
```
当然也可以直接合并成
```ruby
  git submodule update --init --recursive
```
