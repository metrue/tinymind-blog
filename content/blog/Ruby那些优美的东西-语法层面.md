---
title: Ruby那些优美的地方-语法层面
date: 2015-02-16T09:20:02.322Z
---

我对优美的东西真实一点抵抗力都没有啊，其实学习 Ruby 看的第一本书是 "Ruby 元编程", 一下子就陷入了 Ruby 的坑，
"Ruby 元编程" 是一本把 Ruby 的与众不同的特性展示的最好的一本书，整本书一句废话都没有，无论是 Ruby 刚入门还
是 Ruby 老手，我都特别推荐这本书，在看这本书之后，你才发现你开始深入体会了 Ruby 让程序员变得快乐的真正意义。

我不敢说也不是 Ruby 老手，我只是单纯喜欢这门优美的语言，本文我想从语法的层面来展示 Ruby 那些优美的细节。当
然我更希望自己后续可以写一篇 "Ruby: 那些优美的的地方-设计层面"

### 循环

* for

  几乎对于所有的编程语言而言，for 语句的存在都是为了遍历数组，而在 Ruby 中遍历数组的方式可以有

```
  for i in array
    code block
  end
或者
  array.each do |element|
    code block
  end
或者
  array.each_with_index do |element, index|
    code block
  end
或者
```

其实对于循环，Ruby 有非常漂亮的其他语法.

 - upto

```
  1.upto(10) do |i|
    code block
  end
```

  - downto
```
  10.downup(1) do |i|
    code block
  end
```

  - .. 运算符: 创建从开始到结束的范围

```
  (1..10).each do |i|
    code block
  end
```

  - ... 运算符: 创建从开始到结束的范围, 但是不包含结束

```
  (1...10).each do |i|
    code block
  end
```


### 分支控制

* if

  Ruby 的 if 语句和其他语言基本上没有区别，比较有意思的可以使用 short if，在很多场景下很方便.

```
  if expression
    code block
  end

或者
  if expression
    code block
  elsif expression
    code block
  end
```

  Short if 如

```
  variable = (expression) ? ( expression-if-true ) : ( expression-if-false)
```

* case

  case 语句非常有用，当对表达的结果有可预知的范围时，用 case 语句非常的方便.

```
  case expression
  when expression1
    code bloc
  when expression2
    code bloc
  else
    code bloc
  end
```

* unless

  还有比较好玩的 unless 语句, 它和 if !expression 实际上是等价.

```
  unless expression
    code bloc
  end
```

* while, until
  也和其他语言如 Shell 是一样的.

```
  while expression
    code bloc
  end

  until expressino
    code bloc
  end
```
