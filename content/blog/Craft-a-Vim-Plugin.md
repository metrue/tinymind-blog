---
title: 自己动手写一个vim插件吧
date: 2016-07-17T09:17:47.322Z
---

每一个程序员都应该有洁癖, 这个洁癖应该包括两个方面:
一是代码书写的整洁，二是代码逻辑的清晰简洁。第二点需要我们不断的学习，大量的阅读思考，日积月累才能做到，因为逻辑的清晰，需要我们的良好的算法设计，结构优化，性能调优等等，只有长期的知识积累才能做到。然后代码书写的整洁却是我们可以随时做到，而且各种各样的
IDE
也都有着良好的美化功能。所以整洁的代码书写如果我们不想自己动手去整理，我们大可以让我们的工具帮我们来完成。

vim一直是我的主打编辑器，尝试了其他很多编辑器之后，我还是回到了vim。有了
[vundle](https://github.com/VundleVim/Vundle.vim),
vim实在是太方便了，无数优良的插件，让我们的vim几乎可以完成我们日常中所有需求。来到了 [secondspectrum](secondspectrum.com)
之后，我的每一段代码至少都会被一位同事 review,
所以良好的代码风格，整洁的代码书写真的关乎颜面啊，所有前天我同事和我说，我的代码里面有很很多句尾空格，能不能去掉。我大惊.
 :set list
一看，果然好多句尾空格，想了一下，总不能不同敲击 :%s/\s+$//g 来去掉这烦人的东西吧，还是google看看有没有可以用的插件吧，搜了一圈，没有看到合适的(莫非我姿势不对?)。

好吧，那就自己来做一个吧。

### 开始动手吧
我们知道写vim的插件使用
[vimL](https://en.wikipedia.org/wiki/Vim_(text_editor)#Vim_script), 当然现在我们也可以使用 Ruby, Python 等来编写 vim 插件，不过 vimL 其实也不难，简单看看就可以使用了。

* 构建Pathogen/Vundle/NeoBundle/Plug/VAM-compatible 的插件项目结构

```
初始化你的插件目录如下:
```

```

~/.vim/autoload/...
      /doc/...
      /autoload/...
      /ftplugin/...
      /indent/...
      /plugin/...
      /syntax/...
      /...

```

让我们来简单了解一下各个目录和文件都有什么用吧。

1. autoload
vim 插件和常规使用的一些 functions
2. doc
文档
3. ftplugin
使用于指定文件类型的vim插件脚本, 这里面所有以 .vim  结尾的文件在检测到文件类型之后，如果文件名匹配都会被  source .
4. indent
针对指定文件类型的缩紧设置
5. plugin
标准的vim插件, 这里所有以  .vim  结尾的文件在 vim 启动的时候都会被  source
6. syntax
这里放置的是语法高亮设置。

有了这些基本了解之后，我们就可以开始动手了

* 编写插件代码

给我的插件命名为 ’trims' 吧，简单但是不明了。

```
  $ mkdir trims
  $ cd trims
  $ mkdir doc plugin
  $ echo "a simple vim plugin to remove space in the end of line" > README
```

我们的插件只要这么简单的结构就可以了。

```
  $ vim plugin/trims.vim
```

内容添加如下:

```
fun! TrimWhitespace()
  let l:save_cursor = getpos('.')
  %s/\s\+$//e
  call setpos('.', l:save_cursor)
endfun

" trim end of line space hook
autocmd BufWritePre <buffer> call TrimWhitespace()
```

这样我们就完成了我们的插件书写，内容很简单，对吧。一个函数  TrimWhitespace() , 使用正则替换掉句尾的空格，然后在我们保持我们的文件 :w, 就调用我们的函数。

* 测试我们的插件

我们可以很简单的测试我们插件，我们可以在我们的  ~/.vimrc  添加  source /path/to/trims.vim, 或者我们把我们整个 trims 目录copy到我们的插件目录下。然后随意编辑一个文件，然后保存即可看到效果。

* 发布我们的插件

写好插件，我们当然希望可以帮助到其他人喽，如何让我们以及其他人可以简单的使用我们的插件呢，vundle 当然是最方便的啦。

1. push 插件到 [GitHub](https://github.com)

先到  [GitHub](https://github.com) 上开一个 repo，比如我们就叫做 "trims", 然后push我们的插件到repo上。

```
  $ cd trims
  $ git init

  $ git remote add origin https://github.com/metrue/trims.git
  $ git push -u origin master
```

2. 配置 .vimrc 按照并且使用我们的插件

首先，当然要先确保你已经安装并且配置好了
[vundle](https://github.com/VundleVim/Vundle.vim), 然后在你的  ~/.vimrc  中添加

```
  Plugin 'metrue/trims'
```

然后运行 vim, 运行  :BundleInstall . 安装完毕之后，你的vim就可以使用我们自己写的去掉句尾空格的插件了.

### 结尾
本文demo的插件地址:
[https://github.com/metrue/trims](https://github.com/metrue/trims)
