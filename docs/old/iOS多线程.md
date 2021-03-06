# iOS 多线程

零、线程的注意点（掌握）

1.不要同时开太多的线程（1~3 条线程即可，不要超过 5 条）

2.线程概念

> 主线程 ： UI 线程，显示、刷新 UI 界面，处理 UI 控件的事件
> 子线程 ： 后台线程，异步线程

3.不要把耗时的操作放在主线程，要放在子线程中执行

一、NSThread（掌握）

1.创建和启动线程的 3 种方式

> 先创建，后启动

```
// 创建
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(download:) object:nil];
// 启动
[thread start];
```

> 创建完自动启动

```
[NSThread detachNewThreadSelector:@selector(download:) toTarget:self withObject:nil];
```

> 隐式创建（自动启动）

```
[self performSelectorInBackground:@selector(download:) withObject:nil];
```

2.常见方法

1> 获得当前线程

```
(NSThread *)currentThread;
```

2> 获得主线程

```
(NSThread *)mainThread;
```

3> 睡眠（暂停）线程

```
(void)sleepUntilDate:(NSDate *)date;
(void)sleepForTimeInterval:(NSTimeInterval)ti;
```

4> 设置线程的名字

```
(void)setName:(NSString *)n;
(NSString *)name;
```

二、线程同步（掌握）

1.实质：为了防止多个线程抢夺同一个资源造成的数据安全问题

2.实现：给代码加一个互斥锁（同步锁）
@synchronized(self) {
// 被锁住的代码
}

三、GCD

1.队列和任务
1> 任务 ：需要执行什么操作

- 用 block 来封装任务

2> 队列 ：存放任务

- 全局的并发队列 ： 可以让任务并发执行

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

- 自己创建的串行队列 ： 让任务一个接着一个执行

```
  dispatch_queue_t queue = dispatch_queue_create("cn.heima.queue", NULL);
```

- 主队列 ： 让任务在主线程执行

```
  dispatch_queue_t queue = dispatch_get_main_queue();
```

2.执行任务的函数

1> 同步执行 : 不具备开启新线程的能力
dispatch_sync...

2> 异步执行 : 具备开启新线程的能力
dispatch_async...

3.常见的组合（掌握）

1> dispatch_async + 全局并发队列
2> dispatch_async + 自己创建的串行队列

4.线程间的通信（掌握）

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
// 执行耗时的异步操作...

dispatch_async(dispatch_get_main_queue(), ^{
// 回到主线程，执行 UI 刷新操作
});
});
```

5.GCD 的所有 API 都在 libdispatch.dylib，Xcode 会自动导入这个库

- 主头文件 ：

```
#import <dispatch/dispatch.h> （自动导入的）
```

6.延迟执行（掌握）

1> perform....

```
// 3 秒后自动回到当前线程调用 self 的 download:方法，并且传递参数：@"http://555.jpg"
[self performSelector:@selector(download:) withObject:@"http://555.jpg" afterDelay:3];
```

2> dispatch_after...

```
// 任务放到哪个队列中执行
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
double delay = 3; // 延迟多少秒
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delay \* NSEC_PER_SEC)), queue, ^{
// 3 秒后需要执行的任务
});
```

7.一次性代码（掌握）

```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
// 这里面的代码，在程序运行过程中，永远只会执行 1 次
});
```

四、单例模式(懒汉式)
1.ARC

```
@interface HMDataTool : NSObject

- (instancetype)sharedDataTool;
  @end

@implementation HMDataTool
// 用来保存唯一的单例对象
static id \_instace;

- (id)allocWithZone:(struct \_NSZone \*)zone
  {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
  \_instace = [super allocWithZone:zone];
  });
  return \_instace;
  }

- (instancetype)sharedDataTool
  {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
  \_instace = [[self alloc] init];
  });
  return \_instace;
  }

* (id)copyWithZone:(NSZone \*)zone
  {
  return \_instace;
  }
  @end
```

2.非 ARC

```
  @interface HMDataTool : NSObject

- (instancetype)sharedDataTool;
  @end

@implementation HMDataTool
// 用来保存唯一的单例对象
static id \_instace;

- (id)allocWithZone:(struct \_NSZone \*)zone
  {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
  \_instace = [super allocWithZone:zone];
  });
  return \_instace;
  }

- (instancetype)sharedDataTool
  {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
  \_instace = [[self alloc] init];
  });
  return \_instace;
  }

* (id)copyWithZone:(NSZone \*)zone
  {
  return \_instace;
  }

* (oneway void)release {

}

- (id)retain {
  return self;
  }

- (NSUInteger)retainCount {
  return 1;
  }

- (id)autorelease {
  return self;
  }
  @end
```

五、NSOperation 和 NSOperationQueue

1.队列的类型
1> 主队列

- [NSOperationQueue mainQueue]
- 添加到"主队列"中的操作，都会放到主线程中执行

2> 非主队列

- [[NSOperationQueue alloc] init]
- 添加到"非主队列"中的操作，都会放到子线程中执行

  2.队列添加任务

- - (void)addOperation:(NSOperation \*)op;
- - (void)addOperationWithBlock:(void (^)(void))block;

    3.常见用法
    1> 设置最大并发数

* (NSInteger)maxConcurrentOperationCount;
* (void)setMaxConcurrentOperationCount:(NSInteger)cnt;

2> 队列的其他操作

- 取消所有的操作

* (void)cancelAllOperations;

- 暂停所有的操作
  [queue setSuspended:YES];

- 恢复所有的操作
  [queue setSuspended:NO];

  4.操作之间的依赖（面试题）

- NSOperation 之间可以设置依赖来保证执行顺序
- [operationB addDependency:operationA];
  // 操作 B 依赖于操作 A，等操作 A 执行完毕后，才会执行操作 B
- 注意：不能相互依赖，比如 A 依赖 B，B 依赖 A
- 可以在不同 queue 的 NSOperation 之间创建依赖关系

  5.线程之间的通信
  NSOperationQueue \*queue = [[NSOperationQueue alloc] init];
  [queue addOperationWithBlock:^{
  // 1.执行一些比较耗时的操作

  // 2.回到主线程
  [[NSOperationQueue mainQueue] addOperationWithBlock:^{

  }];
  }];

六、从其他线程回到主线程的方式
1.perform...
[self performSelectorOnMainThread:<#(SEL)#> withObject:<#(id)#> waitUntilDone:<#(BOOL)#>];

2.GCD
dispatch_async(dispatch_get_main_queue(), ^{

});

3.NSOperationQueue
[[NSOperationQueue mainQueue] addOperationWithBlock:^{

}];

七、判断编译器的环境：ARC 还是 MRC？
#if \_\_has_feature(objc_arc)
// 当前的编译器环境是 ARC

#else
// 当前的编译器环境是 MRC

#endif

八、类的初始化方法
1.+(void)load

- 当某个类第一次装载到 OC 运行时系统（内存）时，就会调用
- 程序一启动就会调用
- 程序运行过程中，只会调用 1 次

  2.+(void)initialize

- 当某个类第一次被使用时（比如调用了类的某个方法），就会调用
- 并非程序一启动就会调用

  3.在程序运行过程中：1 个类中的某个操作，只想执行 1 次，那么这个操作放到+(void)load 方法中最合适

九、第三方框架的使用建议 1.用第三方框架的目的
1> 开发效率：快速开发，人家封装好的一行代码顶自己写的 N 行
2> 为了使用这个功能最牛逼的实现

2.第三方框架过多，很多坏处(忽略不计)
1> 管理、升级、更新
2> 第三方框架有 BUG，等待作者解决
3> 第三方框架的作者不幸去世、停止更新（潜在的 BUG 无人解决）
4> 感觉：自己好水

3.比如
流媒体：播放在线视频、音频（边下载边播放）
非常了解音频、视频文件的格式
每一种视频都有自己的解码方式（C\C++）

4.总结
1> 站在巨人的肩膀上编程
2> 没有关系，使劲用那么比较稳定的第三方框架

十、cell 的图片下载 1.面试题
1> 如何防止一个 url 对应的图片重复下载

- “cell 下载图片思路 – 有沙盒缓存”

2> SDWebImage 的默认缓存时长是多少？

- 1 个星期

3> SDWebImage 底层是怎么实现的？

- 上课 PPT 的“cell 下载图片思路 – 有沙盒缓存”

  2.SDWebImage
  1> 常用方法

* (void)sd*setImageWithURL:(NSURL *)url placeholderImage:(UIImage \_)placeholder;
* (void)sd*setImageWithURL:(NSURL *)url placeholderImage:(UIImage \_)placeholder options:(SDWebImageOptions)options;
* (void)sd*setImageWithURL:(NSURL *)url placeholderImage:(UIImage \_)placeholder completed:(SDWebImageCompletionBlock)completedBlock;
* (void)sd*setImageWithURL:(NSURL *)url placeholderImage:(UIImage \_)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock;

2> 内存处理：当 app 接收到内存警告时
/\*\*

- 当 app 接收到内存警告
  \*/

* (void)applicationDidReceiveMemoryWarning:(UIApplication *)application
  {
  SDWebImageManager *mgr = [SDWebImageManager sharedManager];
  // 1.取消正在下载的操作
  [mgr cancelAll];
  // 2.清除内存缓存
  [mgr.imageCache clearMemory];
  }

3> SDWebImageOptions

- SDWebImageRetryFailed : 下载失败后，会自动重新下载
- SDWebImageLowPriority : 当正在进行 UI 交互时，自动暂停内部的一些下载操作
- SDWebImageRetryFailed | SDWebImageLowPriority : 拥有上面 2 个功能
