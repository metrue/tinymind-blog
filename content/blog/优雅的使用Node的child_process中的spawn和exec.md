---
title: 优雅的使用 Node 的 child_process
date: 2016-06-09T09:20:02.322Z
---


我们都知道在 Node 中我们可以通过 <code> child_process </code> 中的 exec 或者 spawn 来运行外部命令，而外部命令将成为我们 Node 进程的子进程，然后我们可以在我们的 Node 进程中对这些子进程进行管理。这篇文章，我们来聊聊如何优雅的使用 exec, spawn 以及管理我们 Node 进程的子进程。

### 开始使用

如果用 spawn, 我们一般可以这样写我们的程序

```
  const script = `/path/to/your/script`; // or just some command
  const arguments = [];
  const p = spawn(script, arguments);

  p.stdout.on('data', (data) => {
    // do something
  });

  p.stderr.on('data', (data) => {
    // do something
  });

  p.on('exit', (err) => {
    // do something
  });
```

而如果使用 exec 的话，我们可以这样写.

```
  const script = `/path/to/your/script`; // or just some command
  const arguments = [];
  const p = exec(script, (err, stdout, stderr) => {
    // do something
  });
```

这样，我就可以愉快的在我们的 Node 程序中运行外部命令了。直到我们要对我们的子进程做更多的管理的时候。

### 管理子进程

如果我们要运行的外部命令或者脚本是一个很健康的脚本，我们就可以省心了，然后很多时候，我们要运行的脚本通常豆不是我们自己写，而且很多时候不由我们控制，甚至我们都不知道运行的是什么任务，比如我们要运行的是一个恶意脚本，比如:

```
  !#/bin/bash

  while [ 1 ]; do
    // calculate huge stuff
  done
```

这样永恒运行的脚本，会一直暂用我们的 cpu。所以我们会想到，我们应该给我们要运行的任务设计一个 Timeout 机制呀。所以我们的程序会变成这样。

```
  const script = `/path/to/your/script`; // or just some command
  const arguments = [];
  const p = exec(script, (err, stdout, stderr) => {
    // do something
    if (err) console.log(err);
    if (stderr) console.log(stderr);
    if (stdout) console.log(stdout);
  });

  const TIMEOUT = 3600;
  setTimeout(() => {
    process.kill(p.pid, 'SIGKILL');
  }, TIMEOUT);
```

这样我们有可以愉快的去玩耍了。直到有一天，我们的客户把脚本变成了下面的这样子。

```
  !#/bin/bash

  sleep 10000; // 或者 cat /dev/zero > /dev/null
  while [ 1 ]; do
    // calculate huge stuff
  done
```

我们发现我们机器怎么越来越慢呀，用 ps 一看，原来我们的坏任务一直都没有被杀掉，这是为什么呢，这个版本的脚本和上一个脚本不就多一个命令，为什么就杀不掉呢，难道我们设定的定时杀没有被调用吗，我们可以看看。将定时程序变成下面的样子。

```
  setTimeout(() => {
    const killed = process.kill(p.pid, 'SIGKILL');
    if (killed) {
      console.log('killed');
    }
  });
```

咦，我们看到了 killed 呀，明明表明我们脚本已经被杀掉了。但是我们脚本的子进程 sleep/cat 却没有退出，原因是当我们的脚本在运行到 sleep/cat 的时候，这些作为操作系统的原生命令，他们属于我们脚本的子进程，他们的父进程就是我们的正在运行的脚本，父进程此时会等待 ( waitpid ) 子进程的运行, 就在此时，收到了 SIGKILL, 父进程立即被杀掉，没有进行对于它的子进程的清理工作，所以它的子进程变成了孤儿进程 ( organ child ) , 托管到了上级的父进程了。

也就是当我们的脚本里运行着子进程，我们杀掉父进程，也就是我们的脚本进程，子进程并不会因此而死掉，而是变成了孤儿进程，进行运行着。

但是，为什么我们的 exec 的回调也没有被调用呢，让我们看看文档吧。原来只有进程终止了之后才会走进回调中。exec 所运行的脚本被 (SIGKILL 或者 SIGTERM ) 所 KILL 之后，子进程依然还在运行，所以 exec 的回调不会被调用，这个时候，如果手动去杀掉这个剩下的孤儿进程 ( 比如我们可以用 kill <pid> 的方式 ) ，那么我们就可以到 exec 的回调函数得到了运行，而且给出正确了的 error 信息。


那么我们如何可以知道，我们的脚本进程是否真的退出了，是否会有一个异步事件发生呢，有的，不过 exec 不支持，但是 spawn 可以做到这一点。

```
const spawn = require('child_process').spawn;
const script = `${__dirname}/timeout_task.sh`;
const p = spawn(script);

p.stdout.on('data', (data) => {
  // do something
});

p.stderr.on('data', (data) => {
  // do something
});

p.on('exit', (err) => {
  // do something
});

setTimeout(() => {
  console.log(process.kill(p.pid, 'SIGKILL'));
}, 1000);

```

这样，我们可以看到，当我们用 ( SIGKILL 或者 SIGTERM ) 去 kill 运行的子进程时，'exit' 事件就会被触发，我们可以看到预期的 error 信息。这样我们就在可以在这个回调里来做一些事情了。但是我们还是没有办法在 KILL 这个进程的同时能有完全的清理其正在运行的子进程。那么如何才能优雅的解决这个问题呢？

如果我们能够获得我们正在运行的进程的 group id，然后试图去杀掉这个 group 下的所有进程，这样是否可以解决我们的问题呢？这样显然不行，因为我们并不能确定这个 group id 下面的子进程都是我们的目标进程，除非我们自己去生成这个 group id，然后将我们运行的进程的 group 设为这个 id . 显然这种方式也并不优雅，如果能够让要运行进程不依附于我们的 Node 程序，而是一个独立的进程，那么杀掉这个独立进程的 group 下面的进程，显然是合理而且正确的。而 spawn 真的可以帮我们做到这一点。

```
const spawn = require('child_process').spawn;
const script = `${__dirname}/timeout_task.sh`;
const p = spawn(script, { detached: false, shell: true });

p.stdout.on('data', (data) => {
  // do something
});

p.stderr.on('data', (data) => {
  // do something
});

p.on('exit', (err) => {
  // do something
});

setTimeout(() => {
  console.log(process.kill(-p.pid, 'SIGKILL'));
}, 1000);
```

你可以看到我们给 spawn 提供了这样的参数

```
{
  detached: true,
  shell: true
}
```

第一参数是说我们希望我们要运行的进程成为一个独立的进程，而不是作为我们 Node 进程的子进程。然后在调用 process.kill() 的时候，我们提供的参数是 (-pid, signal)，其实也就是相当于

```
  kill -- -<pid>
```

命令，也就是向同一个group下的所有进程发这个 signal。这样就可以确保我们运行的脚本及其子进程都能够接收到相应的signal。如果我们发的 signal 是 SIGKILL, 那么所有的子进程都将被杀掉。
