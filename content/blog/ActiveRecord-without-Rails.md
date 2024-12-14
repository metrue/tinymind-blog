---
title: 在 Rails 之外使用 ActiveRecord
date: 2014-07-28T09:20:02.322Z
---

ActiveRecord做为Rails的核心组件，也是最优秀的ORM框架之一，Ruby社区的好东西真是多啊，而每一样都
够我研究好一阵子。这就是Ruby最好玩的地方。当第一次使用Rails的时候，我就一直想着如何构建一个自己
类Rails框架呢，就当着一个练习也是好，只有这样才能真正的理解Rails的美。

Rails作为漂亮的MVC框架，它也是由很多组件来构建出来的。而ActiveRecord则是MVC中M（Model）的支撑组件。
它屏蔽了繁复的数据库操作，将它们都映射成类的方法，每一个类对应着数据库中的表，而类的每一个实例这是
对应着表的一行，而且实例的属性则是表的每一列。

为了在Rails之外使用ActiverRecord，我们首先要

```ruby
$ gem install activerecord
```
我想用我最近写的一个小工具来简单说说ActiveRecord的用法,简单介绍一下我的小工具：由于我们部门是软件工程部，负责
着公司大多数产品的开发，测试，发布等等，没有运行的任务成百上千，所使用的工具也很多，定制的脚本更是不计其数，但是
我发现这些程序的运行没有一个很好的反馈机制，结果的分布是零散的，这样所导致的结果是我们每天下班之前对于这些job的运行
状况不好收集，不能够很好的向美国的同事进行交接。

所以我要实现的是一个集中的任务状态管理系统，我首先把这个需求简化成:我只需要一个任务的回报中心，每一个运行的程序
隔一段时间可以向这个任务管理服务器进行回报，服务事实将这些程序的运行状态进行更新，现实在web页面上，我们可以良好的进行
监控。


那么我们就可以把模型定义为：每一个账户有很多的任务。

所以我们需要两张表：user 和 task, 对应到Ruby，也就是我们需要两个类: User 和 Task, 它们的关系是

```ruby
class User < ActiveRecord
    has_many :tasks
end

class Task < ActiveRecord
    belongs_to :user
end
```
也许你已经注意到了，我们继承了ActivRecord这个类，而其中的 has_many 和 belongs_to 都是类方法，和attr_reader, 
attr_writer 类似。这样ActiveRecord就一定自动帮你帮User和Task数据库中的表，而属性则和数据库中的列进行了映射.
当然我们要建立一个数据库。

```ruby
  def connect_db
    activerecord::base.establish_connection(:adapter => "sqlite3", :database => "test.db")
  end

  # ++
  # creat table in database
  # ++
  def setup_user_table
    activerecord::schema.define do
      drop_table :users if table_exists? :users
      create_table :users do |table|
        table.column :name, :string
        table.column :email, :string
      end
    end
  end

  def setup_task_table
    activerecord::schema.define do
      drop_table :tasks if table_exists? :tasks
      create_table :tasks do |table|
        table.column :user_id, :integer
        table.column :name, :string
        table.column :status, :string
      end
    end
  end
```

这里我们采用的sqlite3作为数据库，建立两张表，User拥有名字和email两个属性，而Task则拥有所属的user，名字，状态三个属性。
通过这三个方法，我们就快速的建立两个表，而之前的两个类则自动已经和这两个表映射。以后对于数据库的任何操作都可以
通过类的来进行。详尽的源代码可以查看的我的项目github。

```ruby
https://github.com/metrue/mee
```


