---
title: Vim Profiling
date: 2023-02-06T05:35:07.322Z
---

Vim 突然时常就卡住这件事情特别的让人心烦，而且发现好难排查. 

### 无 .vimrc 启动

```
$ vim -u NONE a.txt
```

这样尝试去编辑任意文件，看是依然有问题，如果有问题，那说明不是 .vimrc 的原因，也就是和你的 plugins 或者配置没有关系. 

### 启动时间分析

```
$ vim --startuptime start_time.txt
```
通过上面的 `--startuptime` 参数可以把分析 Vim 的启动时间.

### Vim 编辑过程的性能分析

如果在编辑文件的过程中有时候很慢，卡顿，那么可以在使用 Vim 的过程中进行 profiling. 依次执行如下的命令即可.

```
:profile start profile.log        # 开始 Profiling 并且被日志写到 profile.log 文件
:profile func *                   # 针对所有的函数
:profile file *                   # 针对所有文件
```

继续编辑文件，正常使用是用 Vim, 如果这个时候发生卡顿，那么在等待其恢复正常状态之后.

```

:profile pause                    # 停止 Profiling
:noautocmd qall!                  # 退出 Vim

```

### 查看日志排查问题

可以通过打开 `profile.log` 文件, 去分析里面没有过程的耗时情况了.
