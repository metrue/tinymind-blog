---
title: iOS on Rails: RESTful 服务的提供和消费
date: 2015-01-20T09:20:02.322Z
---

很多的朋友可能对 REST 和 RESTful 的概念比较模糊，其实很简单:

* REST 一种网络服务的架构风格或者设计模式.
* RESTful 是符合 REST 原则的网络服务.

可以简单的认为 REST 是名词，表示一种范式，而 RESTful 描述是否符合该范式。REST 主要有六点约束原则:

* Client-Server: 客户端发起通信
* Stateless: 无状态服务，每一次请求相互独立，不依赖于任何之前状态.
* Cacheable: 响应可以被缓存.
* Uniform Interface: 同意的接口.
* Layered System: 分层系统，仅相邻层互通。
* Code-On-Demand: 按需编码.

那么具体如何去实现 RESTful 的服务呢。只要我们记住如下几个要点就可以实现符合 REST 范式的 RESTful 服务。

* 资源和 URI 一一对应
* 客户端和服务器之间，传递这种资源的某种表现层；
* 客户端通过使用 HTTP 的标准方法对服务端的资源进行操作

### 具体实现

Rails 本身就是一个完美 REST 的架构，当我们创建一个资源时， Rails 已经自动的为我们生成相应的默认路由。

```ruby
$ rake routes
```

可以看到相应的路由，现在我们来对我们的 Rails 进行进一步的设计，让我们的 API Server 更加的 RESTful，而
RESTful 的 API 服务将大大降低了我们 iOS 客户端和 Rails 的交互成本. 更多优秀的 API 设计方案，我们可以参考
[Github](https://developer.github.com/v3/) 和 [Heroku](https://devcenter.heroku.com/articles/platform-api-reference) 的 API 方案。


#### Rails

* API 版本化

良好的版本控制可以很方便我们的前后向兼容支持。Rails 可以很容易就实现 API 的版本化.

```ruby
$ mkdir -p app/controllers/api/v1
$ mv app/controllers/* app/controllers/api/v1/
```

然后在对 controllers/api/v1/application_controller.rb 进行如下修改.

```ruby
namespace :api do
  namespace :v1 do
    resource :news, only: [:index, :show]
  end
end
```

因为我们的新闻资源只需要支持GET就可以，客户端不能去修改和删除服务器端的新闻资源.
我们可以立即查看到我们的路由情况.

```ruby
 $ rake routes
Prefix            Verb    URI Pattern                            Controller#Action
api_v1_news_index GET     /api/v1/news(.:format)                 api/v1/news#index
api_v1_news       GET     /api/v1/news/:id(.:format)             api/v1/news#show

```

* 通过 Etag 支持缓存

Etag 是个好东西，客户端在请求头添加 If-Not-Match, Rails 可以通过检查 Etag 确定资源
是否发生了变化，决定给客户端发送资源还是返回 304 Not Motified.

```ruby
def index
  all_news = News.all
  if stales(etag: all_news)
    render :json => all_news
  end
end

def show
  news = News.find_by(id: params[:id])
  if stale?(etag: news)
    render :json => news
  end
end
```

* 将授权码放到 Header

上篇文章我们的 Token 和 UUID 都放在 URL， 这样其实有安全隐患， WEB服务器对访问做记录已经成为了行业的一个标准，
访问记录不仅可以用来做访问量统计还能用来做访问特征分析。互联网广告平台就是利用访问记录来做精准营销的。如果 Token
包含在URL中就有很大的安全风险。 包含在 URL 中的 Token 串可能被进行重定向传递。通过这两种方式入侵者可以不通过授权
而使用泄漏的授权码访问那些受保护的数据，会造成数据泄漏的风险。

```ruby
def token_authentication
  device_uuid = request.headers[:UUID]
  token = request.headers[:Token]
  if User.find_by(device_uuid: device_uuid, token: token)
    return true
  end
  return false
end
```

* 返回正确的 HTTP 状态码

Rails 可以在 render 方法中很方便的设置 HTTP 状态码，

```ruby
  render :status => 304
```

#### iOS

* 设置 HTTP request 的请求头

我们使用了 AFNetworking 来协作我们进行方便的网络操作，我们可以很简单往 HTTP Headers 中加入 API Server 的授权信息.

```ruby
AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
[manager.requestSerializer setValue:[[UserManager sharedUserManager] getUUID] forHTTPHeaderField: @"UUID"];
[manager.requestSerializer setValue:[[UserManager sharedUserManager] getToken] forHTTPHeaderField: @"Token"];
```

* Etag 和 iOS 的缓存

这个主题我们将在之后详细的探讨。

* 创建和 Rails 的 Model 一致的 Model

本来我打算使用 [RestKit](https://github.com/RestKit/RestKit) 来负责 iOS 和 API Server 的交互, 不过后来我觉得用太多的库会
使得我们初学者缺少学习的机会，所以我们还是自己来写吧，当然善用开源的各种库可以大大减轻我们开发的难度。

```ruby
@interface News : NSObject

@property NSString *title;
@property NSString *link;
@property NSString *image;
@property NSString *video;
@property NSString *category;
@property NSString *pubDate;
@property NSString *site;

- (id)initWithDictionary:(NSDictionary *)aDict;

@end

```

其实 iOS 作为 RESTful 的消费者，不慎不需要做太多的牵连改动. 为了让我们的新闻良好的显示。
我们本期开始设计一下我们的 iOS 客户端吧。

* 设计

我们需要一个 UITableView 来列表式的显示我们的新闻，然后我们可以简单的设计每一列的单元格，
让以一种简洁而又有效的方式显示我们的新闻。所以我们需要添加两个类：NewsListViewController 和
NewsItemCell , 类的具体实现可以查看我们的完整代码示例。

与此同时，我们还需要让我们的新闻列表支持下拉刷新功能，我们借助 SVPullToRefresh 的威力.

```ruby
 __weak NewsListViewController *weakSelf = self;
[self.tableView addPullToRefreshWithActionHandler:^{
    [weakSelf requestNews];
}];

// setup infinite scrolling
[self.tableView addInfiniteScrollingWithActionHandler:^{
    [weakSelf requestNews];
}];

```
关于设计本文就只是一个开始，后续我们将详细的介绍 iOS 的设计范式.

### 最后

你依然可以在 GitHub 上看我们的完整代码示例.
[https://github.com/metrue/iOSonRails](https://github.com/metrue/iOSonRails)
