---
title: iOS on Rails: Block basic in Ruby and Objective C
date: 2015-01-05T09:20:02.322Z
---

在 Ruby 和 Objective C 中都存在 block 这个很好玩的东西，
在 Javascript 和其他的语言中，一般称为闭包 ( closure ).
而其本质基本上是一样: bock/closure 是一个函数或者指向函
数的指针，以及其执行的上下文变量。但是在不同的语言中，
block 的具体体现略为不同，Ruby 的 block 非常的简洁易懂，
而 Objective C 的 block 基本上保持 C 语言中函数指针的形式。

无论在 Objective C 还是 Ruby， 或者其他语言中，block/closure
的作用都是: 穿越作用域 和 完成特定的运算功能。本文我们就来
对比一下 block 在 Ruby 和 Objective C 中是如何实现这两个功能的。

### Block 的基本形式

Ruby 中的 Block 只有两种形式:

* 大括号包围的形式
```ruby
{ block body }
```
* do end 包围的形式
```ruby
do
    block body
end
```

Ruby 中几乎所有的东西都是对象，但是上面的 Block 却不是，那么 Ruby 是否
可以让 Block 也成为一种对象呢，这样就可以将 Block 保存起来以后使用了。
当然是有的，Proc 和 Lambda
```ruby
dec = proc { |x| x -= 1 }
dec.call(1) ==> 0
```

```ruby
inc = lambda { |x| x += 1 }
inc.call(1) ==> 2
```
Proc 和 Lambda 的区别是: Lambda 由对参数格式要求严格，也就是调用 Lambda 的使用参数
必须和创建 Lambda 的时候一致，其次，return 在 Lambda 中表示从 Lambda 中返回，而在
Proc 则表示从定义 Proc 的作用域中返回。所以如果你在 <main> 中定义 Proc，在 Proc 中
使用 return 的时候会遇到这样的错误:
```ruby
in `block in <main>’: unexpected return (LocalJumpError)</main>`
```

而在 Objective C 中 Block 看起来很纠结，手写起来太容易出错了， 我一般
是这样来记忆 Objective C 中的 Block:
当 Block 作为左值时:
```ruby
returnType (^blockName)(parameterTypes)
```
当 Block 作为右值时:
```ruby
^returnType(parameters) { blockBody }
```

基于上面的原则我们可以很快写出 Block 的几种基本使用场景:

* 作为局部变量
```ruby
returnType (^blockName)(parametertypes) = ^returnType(parameterts) { blockBody}
```

* 作为属性
```ruby
@property (nonatomic, copy) returnType (^blockName)(parameterTypes)
```
* 作为定义方法时候的参数(parameter)
```ruby
- (void)someMethodThatTakesABlock:(returnType (^)(parametertTypes))blockName;
```
* 作为调用方法时候的传递参数(argument)
```ruby
[someObject someMethodThatTakesABlock:^returnType(parameters) { blockBody }];
```
* 使用 typedef 定义一个 Block 类型
```ruby
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^returnType(parameterts) { blockBody };
```
* inline Block
这中 Block 正如 Block 的字面意思一样，就是一个代码段，也分为有参和无参两种
有参：
```ruby
^returnType (parameterTypes) {
    blockBody;
}(parameterts);
```
无参:
```ruby
{
    blockBody;
}
```

### Block 的威力
正如前面说的，Block 的作用可以总结为: 穿越作用域，以及完成特定运算。

* 穿越作用域

对于 Ruby 来说，分隔作用域的三道门为: class, module, def。 为了是得
变量可以穿越这三道门，我们可以使用 Class.new 代替 class 去定义一个类，
使用 Module.new 去代替 module 去定义一个 module，以及使用 define_method
去代替 def 去定义个方法，这样上下文的变量就可以在定义中得到使用了。
```ruby
var = “foo”
MyClass = Class.new do 
  puts “#{var} is in MyClass now”
end
```
他们的区别是，class 和 module 定义中的代码会马上执行，而方法定义中的
代码之用在调用的时候才会执行。

对于 Objective C 来说，作用域的概念基本上都是继承于 C 的，和本文内容
相关时 blockBody 如何访问上下文变量的问题, 基本上和 Ruby 一样。
```ruby
NSString x = @”hello world”;

^{
  NSLog(@“%@”, x);
}
```

* 完成特定的预算

由于可以在 Block 的 blockBody 完成任意的操作，所以 Block 无论是在 Ruby 还是
Objective C 中都有这广泛的运用，也是 Ruby 和 Objective C 中的一等公民。比如
Ruby 中内置的数据类型的绝大多数操作都支持 Block，如 Array 中的 each, Hash中
的each_pair 等，而在 Rails, Jekyll, Grape 等优秀的基于 Ruby 的框架中，Block
的使用就更加频繁和优雅了。 同样在 Objective C 中，可以说没有 Block 就没有Objective C,
iOS 中 Cocoa 里面大量的设计模式都必须有 Block 作为支撑。而大量的优秀的开源
框架更是将 Block 用的微妙微翘, 比如上篇文章 [iOS on Rails: up and running](http://blog.minghe.me/ios/rails/2014/12/30/iOS%20on%20Rails:%20up%20and%20running.html)
中使用到的 AFNetworking 框架，在 HTTP GET 的 succecss 和 failure 的处理函数
都是使用的是 Block. 所以深入理解 Block 是学习 Ruby 和 iOS 开发的基础，甚至
对于理解其他语言如 Javascript 都是非常有意义的。

### 最后
你依然可以在 GitHub 上看我们的完整代码示例.
[https://github.com/metrue/iOSonRails](https://github.com/metrue/iOSonRails)



