---
title: 听说你刚开始写 Python
date: 2016-08-24T10:17:04.322Z
---

听说你准备写 Python, 在你开始真正的 Python 生涯之前，你可以先问问你下面三个问题:

* 你确定你要写 Python 吗
* 你确定你要写 Python 吗
* 你确定你要写 Python 吗

如果你答案中有一点迟疑，那么请你出门左拐到隔壁的 Ruby，或者右拐到隔壁的 Node，或者笔直的往前走, 看看 Go, 也不满意的话，绕过旁边的 PHP, 走回老家，开始玩玩 Perl, 也是极好的。

当然如果你的答案是三个大大的 YES, ( 因为我靠写 Python 为生, 笔者就是这样的 ), 那么和我一起准备好成吨的痛苦的眼泪开始我们的第一个 Python 项目，通过这个简单的项目，也许我们能够开心的码 Python 代码了.

### 基础环境

* Python 版本管理

和其他很多动态语言一样，多版本管理是天然需求了，Ruby 有 rvm, Node 有 nvm, 用脚趾头可能会猜 Python 的版本可能就是 pvm, 还真是不是，它叫 pyenv, 基本上和 rvm 和 nvm 差不多，你掌握下面这些知识，就可以安全的让你度过你 Python 生涯的第一阶段了

```
brew install pyenv      # 安装
pyenv install --list    # 看看可以安装那些包
pyenv install 2.7.12    # 安装 python 2.7.12
pyenv install 3.5.2     # 安装 python 3.5.2
pyenv local 2.7.12      # 设置当前目录使用版本 2.7.12 的 Python
pyenv global 3.5.2      # 设置当前目录使用版本 3.5.2 的 Python
pyenv version           # 显示当前所使用的 Python
```

* Python 包管理

如果你之前是使用 Homebrew 安装了 Python，那么包管理工具 pip 应该已经安装好了，也可以这样安装 pip.

```
curl https://bootstrap.pypa.io/ez_setup.py -o - | sudo python
easy_install pip
```

* 包版本管理

不像 Node 的 npm，npm 可以安装依赖到当前项目本地，不会产生和系统全局依赖版本冲突的问题, 而 Python 的 pip 是会检查系统是否已经存在了相关包，然后才会决定是否安装，而且是安装到全局目录上。为了解决这个问题, Python 社区出现了 virtualenv 这个工具，你可以这么使用它..

```
pip install virtualenv    # 安装virtualenv
virtualenv ENV            # 在当前项目中使用 virtualenv
source ./ENV/bin/active   # 让它生效
```

这样你就可以在当前目录中安装任何合适版本的依赖包了。

### 开始写项目

* package 和 module

在开始写代码之前，Python 有两个概念你需要去了解一下, package 和 module. 在 Python 中，任何以 .py 结尾的文件都是一个 module, 而 package 则是那些则是 modules 的集合, 那些包含 <code> __init__.py </code> 的目录都是一个 package，package 下面可以有 subpackage.

```
a
├── __init__.py
├── a.py
└── b
    ├── __init__.py
    ├── b.py
    └── c
        ├── __init__.py
        └── c.py

```

上面这样的目录结构可以知道 package a 下面有 subpackge b, 而 package b 下面又有 subpackage c，我们可以这样去使用这些 packages

```
import a
from a import a
from a.b import b
from a.b.c import c
```
当然更好的做法是在 __init__.py 中去定义我们需要 export 出来的 module, 比如我们可以:

```
from a import a_say_hello
from b import b_say_hello
__all__ = ['a_say_hello', 'b_say_hello']
```

然后你就可以这样去 import 需要的东西了

```
  from a import a_say_hello, b_say_hello
```

* 项目目录结构

无论你的项目是一个 Application 还是一个 Library, 都可以遵循下面的项目结构

```
.
├── README.md
├── bin
│   └── <app>
├── <package_name> / <app_name>
│   ├── __init__.py
├── setup.py
└── tests
    ├──  test_a.py
    ├──  test_b.py
```

当然，一个简单的 README.md 是必不可少了，如果你需要发布的一个 Application, bin 目录下就是你的 Application 的可执行文件，也就是用户 pip install <your package> 之后可以直接运行的。而 <package_name> 目录下则是你的源代码目录，这下面你可以安装你的逻辑需求分拆成你需要的各种子目录，形成子模块。正如前面所说，你可以通过 __init__.py 来灵活的控制你的代码间依赖可见性。 tests 目录就是你的测试目录，测试是一种信仰，如果一个项目连测试都没有，怎么看出它是一个靠谱的项目呢。setup.py 则是项目的核心管理入口，相当 Node 项目的 package.json. 正确写好 setup.py 才能让我们的 Python 项目顺利的事半功倍完美开始。

* setup.py 入门

其实不是 setup.py 入门，而是 setuptools 入门，通过 setuptools，我们才能愉快的做下面的事情:
  1. 定义要发布的包和模块
  2. 运行测试
  3. 项目安装
  4. 平台相关的信息定义
  5. 支持 Python 3

首先当然需要安装一下 setuptools 这个库，如果你之前已经使用 ez_setu.py 了这个工具，那么 setuptools 这个库就已经安装好了。当然你可以像安装其他库一样安装 setuptools 就好了, 那么让我们来看看一个基础使用的 setup.py 大致是什么样的吧。

```
from setuptools import setup, find_packages, Command
import unittest, os

class CleanCommand(Command):
    """Custom clean command to tidy up the project root."""
    user_options = []
    def initialize_options(self):
        pass
    def finalize_options(self):
        pass
    def run(self):
        os.system('rm -vrf ./build ./dist ./*.pyc ./*.tgz ./*.egg-info')

def test_suite():
    test_loader = unittest.TestLoader()
    test_suite = test_loader.discover('tests', pattern='test_*.py')
    return test_suite

setup(
    name = 'package-cleaner',
    version = '0.0.5',
    description = 'A artifactory packages cleaner',
    packages = find_packages(),
    test_suite = 'setup.test_suite',
    scripts = ['bin/clean_package'],

    author = 'Minghe Huang',
    author_email = 'h.minghe@gmail.com',
    install_requires = ['requests'],
    url = 'http://<project home page>',

    cmdclass = {
        'clean': CleanCommand,
    },
)
```

这个简单的 setup.py 不仅设置我们项目的基本信息，名字，版本，描述等，而且还定义了我们的项目依赖, 当然，你还可以看到我们拓展了一个命令: clean, 这样我们就可以很容易的清理掉 build 出来的中间文件了。有了这些定义之后，我们就可以轻易的做下面的事情了.

```
python setup.py install       # 安装项目
python setup.py build         # build项目，为发布做准备
python setup.py test          # 跑测试
python setup.py sdist upload  # 发布你的项目
```

* 如何写测试

你可以使用 Pytest 来写测试，当然使用 Python 自带的 unittest 也基本上满足要求，看一个简单的例子，你就基本上知道怎么写测试了。

```
import unittest

from package_cleaner import PackageCleaner

class TestPackageCleaner(unittest.TestCase):
    def setUp(self):
        self.server = 'https://repo.xxxx.com/artifactory'
        self.cleaner = PackageCleaner(self.server)

    def test_clean(self):
        path = 'https://repo.xxxx.com/artifactory/api/storage/Solutions'
        cleanable, reason = self.cleaner.is_cleanable(path)
        self.assertEquals(cleanable, False)
        self.assertEquals(reason, 'not consider develop branch')

    @unittest.skip('not safe yet')
    def test_romove(self):
      """
      blahbal
      """

if __name__ == '__main__':
    unittest.main()
```

unittest 提供一些基本的断言函数，通过这些基本的断言函数，基本上可以构造任意的断言了，可以通过 .skip 来略过一些 testcase, 当然还有其他的进阶功能，比如 testcase group 啊就不再多说了，文档很是齐全。

### 最后

至此，你基本可以稍微轻松的写 Python 代码了，赶紧好好写一个牛逼的项目，让大家玩玩吧.
