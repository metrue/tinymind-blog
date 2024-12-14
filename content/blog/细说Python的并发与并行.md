---
title: 细说 Python 的并发和并行
date: '2016-08-30T10:17:04.322Z
---

让我们来完成一个简单的 [Artifactory]() 客户端来深入的熟悉一下 Python 的并行和并发吧。所以我们的需求是这样的: 我们的客户端可以遍历任何的路径，列举处这个目录下的所有文件, 而且尽量快的展示结果，好比如所说我们点击一个文件，要尽量的给我们展示其下面的文件一样，显然这里的性能瓶颈是获取子目录的时间消耗。对于 Artifactory 更是如此，因为每一次获取子目录都是一个 HTTP 请求，那么我们如何可以以最快的速度获取某个目录下面的所有文件呢?

### 并行和并发

这个概念很多人都分不清，显然我自己也没法给处一个权威的解释，不过下面的两个 talk 可以听听，也许你可以了解到到底什么是并发 ( concurrency)，什么是并行(parallelism).

* [concurrency is not parallelism](https://blog.golang.org/concurrency-is-not-parallelism)
* [七周七并发模型](https://www.amazon.cn/%E4%B8%83%E5%91%A8%E4%B8%83%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B-%E5%B8%83%E5%BD%BB/dp/B00V4B2KEI/ref=sr_1_1?ie=UTF8&qid=1472529795&sr=8-1&keywords=%E4%B8%83%E5%91%A8%E4%B8%83%E5%B9%B6%E5%8F%91)

[effective Python](https://www.amazon.cn/Effective-Python-%E7%BC%96%E5%86%99%E9%AB%98%E8%B4%A8%E9%87%8FPython%E4%BB%A3%E7%A0%81%E7%9A%8459%E4%B8%AA%E6%9C%89%E6%95%88%E6%96%B9%E6%B3%95-%E5%B8%83%E9%9B%B7%E7%89%B9%C2%B7%E6%96%AF%E6%8B%89%E7%89%B9%E9%87%91/dp/B01ASI36QS/ref=sr_1_1?ie=UTF8&qid=1472529988&sr=8-1&keywords=effective+python) 中是这么说的，并行和并发的关键区别，在于能不能提速(speedup), 某程序若是并行程序，其中有多条不同的执行路径都在平行的向前推进，则总任务的执行时间会减半，执行速度会变成普通程序的两倍，反之，如果该程序是并发程序，那么它即使可以使用看似平行的方式分别执行多条路径，也依然不会是总任务的执行速度得到提升。我对这种解释是不认可的。

我自己简单的理解是: 并发是设计，并行是程序的运行状态。所以才会有多种的并发模型，而这些模型的目的就是为了让程序过程达到并行的状态。

### 设计一个高效的 Artifactory 客户端

任何树状模型的结构的遍历无非就是两种: 深度优先 (Depth-First-Search) 和 广度优先 (Breadth-First-Search), 对于二叉树来说，深度优先遍历还可以分为: 前序，中序，和后序遍历三种，区别就是根节点访问的顺序。对于目录的访问来说，显然是广度优先的算法，每次都是获取都是目录树某一层的所有节点, 根据 Artifactory 的 [REST API](https://www.jfrog.com/confluence/display/RTF/Artifactory+REST+API), 我们的节点访问可以是这样的.

```
import requests

def visit(node):
  res = requests.get(node)
  dict = res.json()
  try:
    return dict['children']
  except KeyError:
    return None
```

* done comes first

本着'done comes first'的原则，我们第一可以工作的 Artifactory 客户端是这样的

```
class Client(object):
  def tree(self, path):
      def travel(path, items=[]):
          children = self.get_children(path)
          if stat.children:
              for p in [stat.uri + c['uri'] for c in stat.children]:
                  travel(p, items)
          else:
              items.append(path)

      items = []
      travel(path, items)
      return items
```

假设我们的目录是这样子:

```
  -a
    - a1
      - a11
      - a12
      - a13
    - a2
      - a21
      - a22
      - a23
```

当我们其实我们的 client.tree(a) 最后的得到的结果会是我们预想的:

```
a/a1/a11
a/a1/a12
a/a1/a13
a/a2/a21
a/a2/a22
a/a2/a23
```

然而我们需要的时间是: T(a) + T(a1) + T(a2) + T(a11) + T(a12) + T(a13) + T(a21) + T(a22) + T(a23), T(n) 表示去获取节点 n 的时间，获取节点 n 的目的是为了检查其是否有子节点.

* 优化一下

稍微分析一下，在广度优先中，在同一级的节点中，比如 a1, a2，获取他们的子节点的时候，因为没有任何依赖，也就是可以并行进行的。那么我们是否可以用多线程去完成呢，答案是否定，原因是因为 Python 的多线程并不会真的并不会提高我们的任务时间利用率，因为 Python 的 GIL (Global Interpreter Lock), 这是一个 mutex，由于 CPython 的内存管理不是线程安全，所以这个 mutex 起到防止多个线程同时执行。所有当你在  Python 中使用多线程的时候，真实情况是这样的:

```
thread-A - - - - - -
thread-B  - - - - - -
```

可以看出线程 A 和线程 B 在同一个时间单位里只有一个线程在运行，所以总的运行时间并不能减少。所以尝试使用多线程来做并行计算并不能达到我们的需求。那么拿到我们就没有办法加快我的程序了吗？

当然不是，多线程不行，那么为什么不用多进程呢? 所以我们的代码可以这样.

```
def walk_tree(root, walk_func):
    store = Store(root)

    while not store.is_end():
        pool = ProcessPoolExecutor(max_workers=8)
        all_nodes = store.pollall()
        for n in all_nodes:
            store.visiting(n)

        results = pool.map(walk_func, all_nodes)
        for node, children in results:
            store.visited(node)

            if children:
                for c in children:
                    store.to_visit(c)
            else:
                store.done(node)

    return store
```

可以看到我们使用 ProcessPoolExecutor 来实现我们的多进程，这个来自与 concurrency.futures 这个模块 (Python 2, Python 3 它变成了 futures). 这个模块是对 multiprocessing 这个模块的封装，让我们可以很方便的进行多线程的管理。

这是一个通用的树状结构遍历方法，给它一个根节点，以及一个节点信息获取的方法，它就可以多进程的获取一棵树的所有节点了。

### show me the codes

这篇文章的完整思路都这个 Artifactory 客户端, 开源在 GitHub 上.

[artifactor](https://github.com/metrue/artifactor)
