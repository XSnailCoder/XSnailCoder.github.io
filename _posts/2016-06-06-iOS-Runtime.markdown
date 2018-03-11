---
layout:     post
title:      "Runtime"
subtitle:   "不适合人类阅读，非常水的自我笔记"
date:       2016-04-14
author:     "xsnailcoder"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - iOS
---


### Runtime 简介
* Objective-C 是一门动态语言，它将很多静态语言在编译和链接时候做的事情方到了运行时去处理，对于 OC 函数，属于动态调用过程，在编译的时候并不能决定真正要调用的哪个函数，只有在真正运行的时候才会根据函数的名称找到对应的函数来调用。

* runtime (简称运行时），是一套 **C**  和 **汇编** 写的API, Objective-C 就是 **运行时机制**，也就是在运行的时候的一些机制，其中最主要的就是 **消息机制**

### Runtime 消息机制

* 方法调用的本质就是让对象发送消息。

示例代码


    //创建dog对象
    Dog *dog = [[Dog alloc]init];
    
    //创建dog对象底层实现
    Dog *dog = objc_msgSend(objc_getClass("Dog"), sel_registerName("alloc"));
    dog = objc_msgSend(dog, sel_registerName("init"));
    
    //调用对象方法
    [dog run];
    //调用对象方法本质就是让实例对象发送消息
    objc_msgSend(dog, @selector(run));
    
    //调用类方法
    [Dog eat];
    //用类名调用类方法 ，底层会自动把类名转化成类对象调用。本质就是让类对象发送消息
    objc_msgSend([Dog class], @selector(eat));

### Runtime 方法交换
* 使用场景： 系统的方法不能满足开发需求，给系统自带的方法扩展一些功能，并保持原有功能
* 方法一： 集成系统的类，重写系统的方法。
* 方法二： 使用runtime 交换方法。

**Method Swizzling 原理**

在Objective-C中调用一个方法，其实是向一个对象发送消息，而查找消息的唯一依据是selector的名字。所以，我们可以利用Objective-C的runtime机制，实现在运行时交换selector对应的方法实现以达到我们的目的。

每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的Method实现

我们先看看SEL与IMP之间的关系图：
![ios](/img/ios/runtime/selandimp.png)
从上图可以看出来，每一个SEL与一个IMP一一对应，正常情况下通过SEL可以查找到对应消息的IMP实现。

但是，现在我们要做的就是把链接线解开，然后连到我们自定义的函数的IMP上。当然，交换了两个SEL的IMP，还是可以再次交换回来了。交换后变成这样的，如下图
![ios](/img/ios/runtime/selandimpexchange.png)

从图中可以看出，我们通过swizzling特性，将selectorC的方法实现IMPc与selectorN的方法实现IMPn交换了，当我们调用selectorC，也就是给对象发送selectorC消息时，所查找到的对应的方法实现就是IMPn而不是IMPc了



**方法交换代码示例**


    @implementation ViewController

	- (void)viewDidLoad {
	
	    [super viewDidLoad];
	    // Do any additional setup after loading the view, typically from a nib.
	    // 需求：给imageNamed方法提供功能，每次加载图片就判断下图片是否加载成功。
	    // 步骤一：先搞个分类，定义一个能加载图片并且能打印的方法+ (instancetype)imageWithName:(NSString *)name;
	    // 步骤二：交换imageNamed和imageWithName的实现，就能调用imageWithName，间接调用imageWithName的实现。
	    UIImage *image = [UIImage imageNamed:@"123"];
	
	}

    @end


**image 分类**


    @implementation UIImage (Image)

    // 加载分类到内存的时候调用

	+ (void)load
	{
	    // 交换方法
	
	    // 获取imageWithName方法地址
	    Method imageWithName = class_getClassMethod(self, @selector(imageWithName:));
	
	    // 获取imageWithName方法地址
	    Method imageName = class_getClassMethod(self, @selector(imageNamed:));
	
	    // 交换方法地址，相当于交换实现方式
	    method_exchangeImplementations(imageWithName, imageName);
	
	
	}

    // 不能在分类中重写系统方法imageNamed，因为会把系统的功能给覆盖掉，而且分类中不能调用super.

    // 既能加载图片又能打印

	+ (instancetype)imageWithName:(NSString *)name
	{
	    // 这里调用imageWithName，相当于调用imageName
	    UIImage *image = [self imageWithName:name];
		
	    if (image == nil) {
	        NSLog(@"加载空的图片");
	    }
		
	    return image;
	}

    @end






### Runtime 动态添加方法
* 使用场景 ：如果一个类方法非常多，加载类到内存的时候也比较耗费资源，需要给每个方法生成映射表，可以使用动态给某个类，添加方法解决。

**代码示例**

	@implementation ViewController
	
	- (void)viewDidLoad {
	    [super viewDidLoad];
	   
	    Person *p = [[Person alloc] init];
	
	    // 默认person，没有实现eat方法，可以通过performSelector调用，但是会报错。
	    // 动态添加方法就不会报错
	    [p performSelector:@selector(eat)];
	
	}
	
	@end

**Peron 类**

	@implementation Person
	// void(*)()
	// 默认方法都有两个隐式参数，
	void eat(id self,SEL sel)
	{
	    NSLog(@"%@ %@",self,NSStringFromSelector(sel));
	}
	
	// 当一个对象调用未实现的方法，会调用这个方法处理,并且会把对应的方法列表传过来.
	// 刚好可以用来判断，未实现的方法是不是我们想要动态添加的方法
	+ (BOOL)resolveInstanceMethod:(SEL)sel
	{
	
	    if (sel == @selector(eat)) {
	        // 动态添加eat方法
	
	        // 第一个参数：给哪个类添加方法
	        // 第二个参数：添加方法的方法编号
	        // 第三个参数：添加方法的函数实现（函数地址）
	        // 第四个参数：函数的类型，(返回值+参数类型) v:void @:对象->self :表示SEL->_cmd
	        class_addMethod(self, @selector(eat), eat, "v@:");
	
	    }
	
	    return [super resolveInstanceMethod:sel];
	}
	@end

### Runtime 动态添加属性
* 原理：给一个类声明属性，其实本质就是给这个类添加关联，并不是直接把这个值的内存空间添加到类存空间。

**代码示例**

	@implementation ViewController
	
	- (void)viewDidLoad {
	    [super viewDidLoad];
	    // Do any additional setup after loading the view, typically from a nib.
	
	    // 给系统NSObject类动态添加属性name
	
	    NSObject *objc = [[NSObject alloc] init];
	    objc.name = @"小明";
	    NSLog(@"%@",objc.name);
	
	}
	
	@end

NSObject 分类

	// 定义关联的key
	static const char *key = "name";
	
	@implementation NSObject (Property)
	
	- (NSString *)name
	{
	    // 根据关联的key，获取关联的值。
	    return objc_getAssociatedObject(self, key);
	}
	
	- (void)setName:(NSString *)name
	{
	    // 第一个参数：给哪个对象添加关联
	    // 第二个参数：关联的key，通过这个key获取
	    // 第三个参数：关联的value
	    // 第四个参数:关联的策略
	    objc_setAssociatedObject(self, key, name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
	}
	
	@end










