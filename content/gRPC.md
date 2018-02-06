# gRPC
###1、概念

***

gRPC 一开始由 google 开发，是一款语言中立、平台中立、开源的远程过程调用(RPC)系统。

在 gRPC 里*客户端*应用可以像调用本地对象一样直接调用另一台不同的机器上*服务端*应用的方法，使得您能够更容易地创建分布式应用和服务。与许多 RPC 系统类似，gRPC 也是基于以下理念：定义一个*服务*，指定其能够被远程调用的方法（包含参数和返回类型）。在服务端实现这个接口，并运行一个 gRPC 服务器来处理客户端调用。在客户端拥有一个*存根*能够像服务端一样的方法。

### 2、使用技术 protocol buffers

***

gRPC默认使用protocol buffers，这是Google开源的一套成熟的结构数据序列化机制。用proto files 创建gRPC服务，用protocol buffers消息类型来定义方法参数和返回类型。

通常建议在gRPC里使用proto3。因为这样你可以使用gRPC支持全部范围的语言，并且能避免proto2客户端与proto3服务端交互时出现的兼容性问题。

### 3、定义服务

***

使用protocol buffers接口定义语言来定义服务方法，用protocol buffer 来定义参数和返回类型。客户端和服务端均使用服务定义生成的接口代码（以官网例子为demo）。

在官网例子里使用protocol buffers IDL定义。





规格：
iPhone X的屏幕宽度同iPhone6、6s、7、8的4.7英寸屏幕宽度相同（375pt），屏幕垂直高度增加了145pt，一位置增加了20%的可视空间。
竖屏规格：1125px x 2346px（375pt x 812pt @3x）
横屏规格：2436px x 1125px （812px x 375pt @3x）

一、启动页适配
填充iPhonex启动页2x、3x

二、状态栏、搜索框
高度增加了24像素（来电或者热点不会导致状态栏高度变化）；
搜索框宽度在iphonex展示位置

三、底部栏
Tabbar 高度增加了34像素
Toolbar高度不变，只是向上偏移了34像素

四、SafeArea安全区域
安全区域指在屏幕顶部和底部区域之间能正常显示内容的区域。顶部区域包括导航栏、状态栏或者传感器区域。底部区域包含Tabbar、工具栏或者home键指示器区域。

五、屏幕适配
如果你的项目存在导航栏界面push到全屏界面，或者手势滑动做很炫的过场动画，那么你可能会用到自定义导航栏NavigationBar，每个ViewController维护自身的Navigationbar实例。

UINavigationBar *navigationBar = [[UINavigationBar alloc] initWithFrame:CGRectZero];
navigationBar.backgroundColor = [UIColor blueColor];
navigationBar.barTintColor = [UIColor blueColor];
navigationBar.titleTextAttributes = @{NSForegroundColorAttributeName:[UIColor whiteColor]};
navigationBar.items = @[[UINavigationItem new]];
navigationBar.topItem.title = self.title;
[self.view addSubview:navigationBar];

navigationBar.translatesAutoresizingMaskIntoConstraints = NO;

if (@available(iOS 11.0, *)) {
    NSLayoutConstraint *left = [navigationBar.leftAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.leftAnchor];
    NSLayoutConstraint *right = [navigationBar.rightAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.rightAnchor];
    NSLayoutConstraint *top = [navigationBar.topAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor];
    NSLayoutConstraint *height = [navigationBar.heightAnchor constraintEqualToConstant:44];
    [NSLayoutConstraint activateConstraints:@[left, right, top, height]];
}else{
    NSLayoutConstraint *left = [navigationBar.leftAnchor constraintEqualToAnchor:self.view.leftAnchor];
    NSLayoutConstraint *right = [navigationBar.rightAnchor constraintEqualToAnchor:self.view.rightAnchor];
    NSLayoutConstraint *top = [navigationBar.topAnchor constraintEqualToAnchor:self.topLayoutGuide.bottomAnchor];
    NSLayoutConstraint *height = [navigationBar.heightAnchor constraintEqualToConstant:44];
    [NSLayoutConstraint activateConstraints:@[left, right, top, height]];
}


导航栏背景未扩展到状态栏，正常应该显示蓝色。

解决方案:

设置Navigationbar的UIBarPositioningDelegate返回UIBarPositionTopAttached即可。

typedef NS_ENUM(NSInteger, UIBarPosition) {
    UIBarPositionAny = 0,
    UIBarPositionBottom = 1, // The bar is at the bottom of its local context, and directional decoration draws accordingly (e.g., shadow above the bar).
    UIBarPositionTop = 2, // The bar is at the top of its local context, and directional decoration draws accordingly (e.g., shadow below the bar)
    UIBarPositionTopAttached = 3, // The bar is at the top of the screen (as well as its local context), and its background extends upward—currently only enough for the status bar.
} NS_ENUM_AVAILABLE_IOS(7_0);
navigationBar.delegate = self;

- (UIBarPosition)positionForBar:(id <UIBarPositioning>)bar
  {
    return UIBarPositionTopAttached;
  }

  备注：navigationbar扩展到statusbar的颜色为barTintColor的值。如果失效，检查下是否将translucent设置为NO，并且Navigationbar必须为添加到ViewController的一级subView。
  自定义导航栏后发现SafeArea没有变化,这样设置contentview的时候会将navigationbar遮挡。

safeAreaInsets:{44, 0, 34, 0}）
解决方案：设置additionalSafeAreaInsets

/* Custom container UIViewController subclasses can use this property to add to the overlay
 that UIViewController calculates for the safeAreaInsets for contained view controllers.
 */
@property(nonatomic) UIEdgeInsets additionalSafeAreaInsets API_AVAILABLE(ios(11.0), tvos(11.0));
设置该值后也要相应调整下导航栏的布局，之前是在SafeArea之内，现在要改为之外。

self.additionalSafeAreaInsets = UIEdgeInsetsMake(44, 0, 0, 0);
NSLayoutConstraint *left = [navigationBar.leftAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.leftAnchor];
NSLayoutConstraint *right = [navigationBar.rightAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.rightAnchor];
NSLayoutConstraint *bottom = [navigationBar.bottomAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor];
NSLayoutConstraint *height = [navigationBar.heightAnchor constraintEqualToConstant:44];
[NSLayoutConstraint activateConstraints:@[left, right, bottom, height]];
可以看到安全区域也更新了:

safeAreaInsets:{88, 0, 34, 0}



***

## iPhone X 适配

***

###1、搜索框 ：UISearchController

系统ios11  高：56

系统 ios11 高：44

###2、头部适配

iPhoneX     状态栏： 30

​                    导航栏：58   

iPhoneX       总体：88

非iPhone X  状态栏：20

​                     导航栏：44

​                      总体：64

###3、UItableview 系统11之后：

​      XXX.estimatedRowHeight = 0

​      XXX.estimatedSectionHeaderHeight = 0

​      XXX.estimatedSectionFooterHeight = 0



###4、启动页适配

增加iPhone X 启动页增加

image size ： 1125 x 2436 pixels

###5、屏幕底部圆角距离

​     iphone X 因屏幕位圆角：底部增加 34









