---
title: iOS on Rails: Up and Running
date: 2014-12-30T09:20:02.322Z
---

作为一个 iOS 开发，相信很多人和我一样想给自己的App快速的建立后端Server, 把各种
高负荷的计算工作都扔到后端，这样 App 的开发就可以重点放在提高用户体验上。 而作
为一个 Ruby 开发者，我们同样希望自己优雅的 Ruby 代码不仅仅作为 Web App 的后端，
更希望自己也能够给自己的后端写移动 App，在地铁上无聊的时候玩玩。 所以我对自己的
定义是“一个喜欢Ruby的iOS开发者”，我将会陆续的分享我使用 Rails 作为自己App后端的
经验，每一次的分享都将是一个完整的项目，你可以直接 clone 我放在 Github 上面的项
目，然后在自己的本地完美的跑起来，由浅入深。我想把这一系列称之为“iOS on Rails”,
后续可能包括: REST API, 授权认证，消息通知，统计等。

而今天这这篇文章是一个开始，一个很简单的开始: 是一个 Rails 和 iOS 相遇的故事。

## 需求

我们要做一个自己的新闻客户端。

## 分析

这就是一个很典型的需要后端支持的 App 的场景，App不可能去代替后端去完成新闻的获取，
信息的处理等工作，App的目的是为终端用户提供最优质的新闻阅读体验。

## 实现

### Rails

有很多种方式去为 iOS App 搭建一个后端，但是也许 Rails 是最快的方式，
Rails 让我们可以在几分钟之内就可以搭建起一个干净的API Server：backend-as-a-service.
显然我们App需要的resource是新闻（news），那么我们的REST API应该是这样子:

```obj
 GET /news
```

对于Rails来说，我们只要进行下面几步就可以达到目的了:

- 创建一个 Rails 项目

```obj
    rails new Server
```

- 创建相应的model和controller

```obj
    rails g model news
    rails g controller news
```

- 在config/routes.rb中设置路由

```obj
    Rails.Application.routes.draw do
      resources :news
    end
```

现在我们可以查看我们Server已经支持了哪些路由了

{% highlight sh %}
$ rake routes

     Prefix Verb   URI Pattern              Controller#Action
    news_index GET    /news(.:format)          news#index
               POST   /news(.:format)          news#create
      new_news GET    /news/new(.:format)      news#new
     edit_news GET    /news/:id/edit(.:format) news#edit
          news GET    /news/:id(.:format)      news#show
               PATCH  /news/:id(.:format)      news#update
               PUT    /news/:id(.:format)      news#update
               DELETE /news/:id(.:format)      news#destroy
```

当然我们需要去 controller 去实现相应的 Actioin，为了支持 'GET /news' 这个路由，我们需要去实现
index方法，所以我们的 App/controllers/news_controller.rb 如下:

```ruby
class NewsController < ApplicationController
  def index
    render :json => News.all
  end
end
```

- 添加测试数据

给我们的项目添加一点测试数据吧, db/seed.rb

```ruby
1000.times do
  News.create(title:Faker::Name.name,
              link:Faker::Internet.url,
              pub_date:Faker::Time.between(2.days.ago, Time.now),
              image:Faker::Internet.url,
              video:Faker::Internet.url,
              text:Faker::Lorem.paragraph,
              category: Faker::Lorem.word,
              from_site: Faker::Lorem.word)
end

```

{% highlight sh %}
    $ rake db:seed
```

简单的测试一下我们的API。

{% highlight sh %}
    $ rails s
    $ curl 127.0.0.1/news
```

是不是看到哗啦啦的一大波数据，这就说明基于Rails的一个简单的 API Server就完成了。

### iOS

对于很多的 Ruby 开发者来说，习惯了使用 Vim/Emacs 写代码之后，打开笨重的 Xcode，真是一件
不是那么爽的事情啊，不过对于 iOS 开发来说， Xcode 真的对我们的开发提供了很多方便的功能，
无论是 Storyboard 拖拖拽拽还是代码的自动补全和提示都是非常顺手，因为 Objective C 的代码
风格和 Ruby 的差异真是太大。当然如果你自认为没有 Vim/Emacs 就不能写代码的纯洁 geek，可以
给 Xcode 安装 Vim/Emacs 的插件。

开始编写我们的 iOS App 吧。打开 Xcode，创建一个新项目，然后选择 Sigle View Application。
删掉 main Storyboard 以及 LaunchScreen.xib 中的 View。删掉这些的目的让我们从一开始就用
纯代码的方式去完成我们的 iOS App，用 Storyboard 在开始的时候是可以让我们很快的进行布局等
工作，但是当我们的 App 开始变的复杂之后，纯代码的控制让我们可以绕过很多的坑，节省很多宝贵
时间。

在这片文章中，我们的目的很简单: 为我们的 App 创建一个简单的 View, 然后当我们的 App 
从我们的 Rails Server 拿到我们新闻之后， 将第一条新闻的标题显示在 View 的中间。所以
我们只要下面一段简单的代码就可以达到目的.

```obj
self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.window.backgroundColor = [UIColor whiteColor];

    UIViewController *viewController = [[UIViewController alloc] init];
    UIView *view = [[UIView alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    UILabel *label = [[UILabel alloc] initWithFrame:
                      CGRectMake(0,
                                 [[UIScreen mainScreen] bounds].size.height / 2 - 22,
                                 [[UIScreen mainScreen] bounds].size.width,
                                 44)
                      ];
    label.backgroundColor = [UIColor whiteColor];
    [label setTextAlignment:NSTextAlignmentCenter];
    [view addSubview:label];
    viewController.view = view;

    self.window.rootViewController = viewController;
    [self.window makeKeyAndVisible];

    [self get:@SERVER_URL parameters:@{}
      success:^(id JSON) {
          label.text = JSON[0][@"title"];
      } failure:^(NSError *error) {
          NSLog(@"%@", [error localizedDescription]);
      }];

    return YES;
}

- (void)get:(NSString *)url parameters:(NSDictionary *)params success:(void (^)(id))successHandler failure:(void (^)(NSError *))failureHandler {
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];                                                                                                            
    [manager GET:url parameters:params
         success:^(AFHTTPRequestOperation *operation, id responseObject) {
             successHandler(responseObject);
         }
         failure:^(AFHTTPRequestOperation *operation, NSError  *error) {
             failureHandler(error);
         }
     ];
}

```

作为一个骄傲的 Ruby 程序员，你应该已经猜到上面那段代码的大体意思：创建一个 View，在View的
中间放置一个 label，然后从我们的 Rails Server 上请求新闻，并且将第一条新闻标题放到 label
上面去显示。对，就是这样的。当然别忘了在 AppDelegate.m 中添加

```ruby
#import "AFNetworking.h"
#define SERVER_URL "http://127.0.0.1:3000/news"
```

然后你就点击 build and run 或者 command + R 让 Xcode 去编译并且运行我们的 App。如果没有什
么意外你应该可以看到我们的 App 已经完美的运行起来，虽然它非常的简单。但是我们的 Rails 在
此刻和我们的 iOS 终于合体了。(此处应该有掌声 ): )

完整的代码可以在 [https://github.com/metrue/iOSonRails](https://github.com/metrue/iOSonRails)
