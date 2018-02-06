# ios 知识点
1、适配（iPhone X）

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

1、搜索框 ：UISearchController

》ios11  高：56

《 ios11 高：44

2、头部适配

iPhoneX     状态栏： 30

​                    导航栏：58   

iPhoneX       总体：88

非iPhone X  状态栏：20

​                     导航栏：44

​                     总体：64

3、UItableview 系统11之后：

​      XXX.estimatedRowHeight = 0

​      XXX.estimatedSectionHeaderHeight = 0

​      XXX.estimatedSectionFooterHeight = 0



4、启动页适配



5、屏幕底部圆角距离

​     iphone X 因屏幕位圆角：底部增加 34









