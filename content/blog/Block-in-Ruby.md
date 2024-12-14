---
title: 深入浅出Ruby中的Block
date: 2014-07-02T09:20:02.322Z
---

深入的学习Ruby了之后，特别是看《Ruby元编程》之后，特别喜爱上Ruby。Perl的吉祥三宝《Learning Perl》, 《Intermediate Perl》以及 《Mastering Perl》要是像这本书一样写的深入浅出，也许我就没有机会去接触Ruby了。
好吧，闲话不叙了，昨天和[@Opera](http://weibo.com/p/1005051405474972/home?from=page_100505&mod=TAB#place)说Ruby中的块挺好的，本来想简单的叙述一下其基本功能，没想到自己却没有理解清楚，三言两语说的不清不楚，昨天晚上有认真的总结了一下。自己总算理清楚了Block的脉络。

### 从Block开始


#### Block是什么

Ruby几乎所有的东西都是对象，但是Block却不是。
Block 本质上就是方法的匿名参数, 所以Block只能被方法调用, 而且方法只能通过yield来调用Block, Block不能在Ruby中独立生存。Block的表现形式只用两种:

```ruby
{ Block content }
```

或者

```ruby
do
  Block content
end
```

所以{ }或者do end包围的整体就是Block。

```ruby
def test_block()
  yield
end

test_block { puts "is good" }

test_block do
  puts "is bad"
end
```

#### Block可以用来做什么

* <strong>让变量穿越作用域，使作用域扁平化。</strong>

在Ruby中，class，module以及def是作用域门的开关，也就是作用域的边界。各个作用域之间通过这三个门相互隔离，各司其职。但是有时候，我们却需要来回的穿梭在三个域里。Block可以很容易的帮我们做到。

1. 由于class实际上也只是Ruby中Class类的对象而已，所以可以用Class.new代替class来定义一个类，从而达到穿越class这个门的目的。具体的做法是用一个Bloca获取当前的绑定，然后把这个Block传给Class.new这个方法。

```ruby
  var = "foo"
  MyClass = Class.new do
	puts "#{var} is in MyClass now"
  end
```

2. 同理，也可以用Module.new代替module,Module#define_method代替def，就可以通通将module， def这两扇门也打开。

```ruby
  var = "foo"
  define_method :my_method do
	  puts "#{var} is in my method now"
  end

  MyModule = Module.new do
	  puts "#{var} is in MyModule now"
  end
```

但是值得注意的，类(class)和模块(module)的定义中的代码会马上执行，但方法定义中的代码只有在方法被跳用的时候才会被执行。

* <strong>共享作用域</strong>

如果在一个扁平作用域定义多个方法，可以通过用一个作用域门来进行保护，并共享绑定，这种技术称为共享作用域。

```ruby
  def define_methods
	shared_var = 0
	Kernel.send :define_method, :dec do |x|
	  shared_var -= x
	end

	Kernel.send :define_method, :inc do |x|
	  shared_var += x:
	end
  end
```

* <strong>充当上下文探针</strong>

Object#instance_eval()方法的作用是在对象的上下文中执行一个Block，所以这给我们很容易的在不碰其他绑定的情况可以很轻易的修改当前对象。这有一个好显然的好处是，我们在Block中进行运算，最后更新对象。

```ruby
  class MyClass
    def initialize
      @v = 1
    end
  end

  obj = MyClass.new
  v = 20
  obj.instance_eval do
	@v = v
  end
```

由于Block接受者obj是MyClass对象，本身就可以访问MyClass的实例变量，而obj和v又处于同一个作用域中，所以可以访问到v，所以这个Block像是深入对象中的探针一样，可以在通过外部的运算之后对对象内部进行操作。
域Object#instance_eval类似的是Object#instance_exec()， 它允许对块传入参数。

```ruby
  class MyClass
    def initialize
      @v = 1
    end
  end

  obj = MyClass.new
  v = 20
  obj.instance_eval(x) do |var|
	@v = var * v
  end
```

* <strong>实现一个洁净室</strong>

由于Block不能独自生存，必须要依赖与方法，那么有时候创建一个对象的目的就是运行一个Block，那么这个对象和块就形成了一个洁净室。洁净室的作用就是准备Block的运行环境，而且还暴露有用的方法可以用。

```ruby
  class MyClass
    def method1()
      ...
    end

    def method2()
      ...
    end
  end

  obj = MyClass.new
  obj.instance_eval do
	if method1
	  do_something
	end
  end
```

### 然后来到Proc

Proc 本质是转化成对象的Block。因为Block不是对象，它只用通过方法才能执行，那么如何把块保存起来供以后执行呢。这就是Proc类存在的原因。

```ruby
  dec = Proc.new {|x| x - 1}
  dec.call(2) # 1

  inc = proc {|x| x + 1}
  inc(2) # 3
```

上面的例子就是块的延迟执行(Deffered Evaluation), 因为Block本身只要出生就立即执行，当Block变成Proc对象之后，我们就可以在任意适合跳用call()方法去执行。那么这和普通的方法又什么区别呢。首先在Ruby中是可以将方法作为另一个方法的参数的，
但是作为参数的方法会立即执行，然后将其返回的参数作为参数传给另一个方法，也就是说本质上并没有达到传递方法的目的，因为作为参数的方法并没有在另个一个方法中实现调用，而仅仅是即时的运算而已。

Proc里有一个很重要的操作符“&”，绝大多数情况下，在方法中可以通过yield立即执行一个块，但是有两个情况yield却很难应付。

  * 把Block传给另一个方法
  * 把Block转换成Proc
  
所以&的作用是将一个Proc对象变成一个块。因为Proc对象可以在方法之间传递，去掉&之后又变成一个普通的块，可以立即执行。而所以Proc，包括后面会提到的Lamda使得Ruby很容易实现就可以实现高阶函数，而高阶函数则是Ruby函数式编程得基础

```ruby
  def method1
	yield(1)
  end

  def method2(a, &b)
	puts "#{a} is the first parameter"
	puts b.class
	puts b.call(2)
	puts method1(&b)
  end

  method2("a") {|x| x += 1}
```

上面得代码输出为，

```ruby
  a is the first parameter
  Proc
  3
  2
```

&暗示方法这里传递的将是一个Block，而且这个块在&的作用下称为了一个Proc对象，可以在方法之间传递。值得注意的是携带&的参数只能位于方法参数列表从左到右的最后一个。
与&对应的是proc方法和lamda方法，他们的作用是将块转化成一个Proc对象，按照Ruby惯例，proc生成的Proc是普通Proc，而lamda生成Proc称为lamda。

```ruby
  dec = proc {|x| x -= 1}
  dec.call(1)	 # 结果 0

  inc = lamda {|x| x += 1}
  inc.call(1) # 结果2
```

值得注意的是，proc在Ruby 1.9之后的版本中，proc是Proc.new 的别名，而在Ruby 1.8中，实际上Kernel.lambda的别名。有点乱。

lambda 和 proc （Proc.new) 有两个不同的地方，

  1. lambda 对参数个数要求严格，当调用lambda时，必须给对参数格式，而proc对于参数格式没有要求。
  
```ruby
   obj1 = lambda {|a, b| [a, b]}
   obj2 = proc {|a, b| [a, b]}

   obj1.call(1) # 参数未给对
   obj2.call(1, 2) # ［1，2］

   obj2.call() #[nil, nil]
   obj2.call(1, 2) #[1, 2]
```

  2. return 在lambda中表示从lamda中返回，而在proc中则表示从定义Proc的作用域中返回。

```ruby
  obj = lambda {return "abc"}
  obj.call # "abc"

  obj = proc {return "abc"}
  obj.call # 不能从顶级作用域中返回.
```

### 结束

全文结束。
