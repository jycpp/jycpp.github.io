---
title: 学习Objective-C的一些笔记
author: [Jet Yan]
date: 2023-12-26
comments: true
tags:
- Objective-C
- iOS开发
- 编程语言
- 面向对象编程
- 动态类型
- 消息传递
- 代码编译
- 执行效率
- Swift
categories:
- 软件开发
- MacOS
---

## 认识Objective - C

### 什么是Objective - C

Objective - C是一种面向对象的编程语言，它是C语言的超集，同时也兼容C语言。Objective - C的设计目标是提供一种简单、灵活、高效的编程语言，用于开发iOS和MacOS应用程序。Objective - C的语法类似于C语言，但是它引入了一些新的特性，如面向对象编程、动态类型、消息传递等。Objective - C的代码可以直接编译成机器码，因此它的执行效率非常高。

### 先给自己洗脑：OC在iOS开发中的现状与趋势

#### 逐步被Swift取代的趋势
Swift是苹果于2014年推出的全新编程语言，具有语法简洁、安全性能高、开发效率快等显著优势。它在设计上吸收了现代编程语言的众多优秀特性，如类型推断、闭包等，使得代码更加易读、易维护。随着时间的推移，苹果也在不断加大对Swift的支持力度，许多新的框架和API都优先以Swift进行开发和文档编写，同时一些新的编程特性和开发工具也更多地围绕Swift展开，这使得Swift在iOS开发社区中的影响力与日俱增，越来越多的开发者倾向于使用Swift进行项目开发，从长期来看，Objective - C被Swift取代是一个不可避免的趋势。

#### 仍将长期作为主流开发语言的原因
然而，目前大量的iOS项目依旧在使用Objective - C。这主要是因为Objective - C拥有悠久的历史，在iOS开发领域已经积累了庞大的代码库和丰富的开发资源。许多早期开发的APP以及一些大型的、成熟的商业项目都是基于Objective - C构建的，对这些项目进行重构和迁移到Swift需要耗费巨大的人力、物力和时间成本，所以短期内这些项目不会轻易放弃Objective - C。此外，Objective - C与底层系统的交互能力非常强，在一些对性能要求极高或者需要与旧有系统深度集成的场景下，Objective - C仍然具有不可替代的优势。因此，在未来相当长的一段时间内，Objective - C仍将是苹果手机版本APP开发的主流语言之一。

### 学习Objective - C编程的主要知识点

#### 基础语法
- **数据类型**：了解基本数据类型，如 `int`、`float`、`double`、`char` 等，以及对象类型，如 `NSString`、`NSArray`、`NSDictionary` 等。掌握这些数据类型的声明、初始化和使用方法。
- **变量与常量**：学会如何声明和使用变量和常量，理解它们的作用域和生命周期。
- **控制流语句**：掌握 `if - else`、`switch`、`for`、`while` 等控制流语句的使用，用于实现程序的逻辑判断和循环操作。
- **函数**：学习函数的定义、声明和调用，了解参数传递和返回值的处理方式。

#### 面向对象编程
- **类与对象**：理解类和对象的概念，学会定义类、创建对象以及访问对象的属性和方法。
- **继承**：掌握继承的机制，通过继承可以创建新的类，继承父类的属性和方法，并可以进行扩展和重写。
- **多态**：了解多态的概念和实现方式，多态可以提高代码的灵活性和可扩展性。
- **封装**：学会使用封装来隐藏对象的内部实现细节，提供公共的接口供外部访问。

#### 内存管理
- **引用计数**：理解Objective - C的引用计数机制，这是Objective - C内存管理的基础。了解对象的引用计数是如何增加和减少的，以及何时对象会被释放。
- **自动引用计数（ARC）**：掌握ARC的工作原理和使用方法，ARC是苹果推出的一种自动内存管理机制，它可以大大简化内存管理的工作。
- **弱引用和强引用**：了解弱引用和强引用的区别，以及在避免循环引用等场景下的使用方法。

#### 消息传递机制
- **消息发送**：Objective - C采用消息传递机制，学会如何使用 `[对象 方法名:参数]` 的语法来发送消息，理解消息的查找和调用过程。
- **选择器（Selector）**：掌握选择器的概念和使用方法，选择器可以用来表示一个方法的名称，在运行时可以通过选择器来动态调用方法。

#### 框架与API使用
- **Foundation框架**：这是Objective - C的基础框架，包含了许多常用的类和功能，如字符串处理、集合操作、日期和时间处理等。
- **UIKit框架**：用于开发iOS应用的用户界面，学习如何使用UIKit中的各种控件，如 `UIButton`、`UILabel`、`UITableView` 等，以及视图控制器的管理和导航。
- **网络编程**：了解如何使用 `NSURLSession` 等类进行网络请求，实现数据的下载和上传，处理网络错误等。

#### 调试与错误处理
- **调试工具**：学会使用Xcode的调试工具，如断点调试、日志输出等，来定位和解决代码中的问题。
- **异常处理**：了解Objective - C的异常处理机制，学会使用 `@try`、`@catch`、`@finally` 等关键字来捕获和处理异常。 


## 使用Objective - C进行iOS App开发

以下是使用Objective - C进行iOS App开发并将其上传到App Store的详细过程：

### 1. 申请苹果开发者账号
苹果开发者账号是发布iOS应用所必需的，分为个人账号、公司账号和企业账号，一般个人开发者选择个人账号即可。
- **访问苹果开发者官网**：打开 [苹果开发者网站](https://developer.apple.com/)，点击右上角“Account”登录或注册Apple ID。
- **注册开发者账号**：登录后，按照指引选择加入 Apple Developer Program。个人账号每年费用为99美元，需要提供个人信息、联系方式、付款信息等，完成付款后即可成为苹果开发者。

### 2. 安装Xcode
Xcode是苹果官方提供的集成开发环境（IDE），用于开发iOS、macOS等平台的应用程序。
- **打开Mac App Store**：在Mac电脑上找到并打开App Store应用程序。
- **搜索Xcode**：在App Store的搜索框中输入“Xcode”。
- **下载并安装**：找到Xcode应用后，点击“获取”或“下载”按钮，等待下载和安装完成，整个过程可能需要一些时间，具体取决于网络速度。

### 3. 建立第一个iOS App项目
完成Xcode安装后，就可以创建一个新的iOS项目。
- **启动Xcode**：在Launchpad或应用程序文件夹中找到Xcode并打开。
- **创建新项目**：选择“Create a new Xcode project”，在弹出的模板选择窗口中，选择“iOS” - “App”，点击“Next”。
- **配置项目信息**：
    - **Product Name**：输入应用的名称。
    - **Organization Identifier**：一般填写反向域名，如 `com.example`。
    - **Interface**：选择 “Storyboard”（可视化界面设计工具）。
    - **Language**：选择 “Objective - C”。
    - 其他选项根据需要进行设置，设置完成后点击“Next”。
- **选择项目保存位置**：选择一个合适的文件夹来保存项目，点击“Create”，Xcode会自动生成项目的基本结构。

### 4. 编写Objective - C代码
在项目中可以开始编写Objective - C代码实现应用的功能。
- **打开ViewController.m文件**：在项目导航器中找到 `ViewController.m` 文件，这是视图控制器的实现文件，主要用于处理视图的逻辑。
- **编写代码示例**：例如，在 `viewDidLoad` 方法中添加以下代码，在屏幕上显示一个标签：
```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 50)];
    label.text = @"Hello, Objective - C!";
    label.textAlignment = NSTextAlignmentCenter;
    [self.view addSubview:label];
}
```

### 5. 编译和调试
编写完代码后，需要对项目进行编译和调试，确保代码没有错误。
- **编译项目**：点击Xcode工具栏上的“Build”按钮（或者使用快捷键 `Command + B`），Xcode会对项目进行编译，检查代码中的语法错误和其他问题。如果编译成功，会在控制台显示“Build Succeeded”。
- **调试代码**：点击“Run”按钮（或者使用快捷键 `Command + R`），Xcode会尝试在模拟器或真机上运行应用。在运行过程中，可以设置断点（在代码行号旁边点击），当程序执行到断点处时会暂停，此时可以查看变量的值、执行代码步骤等，帮助定位和解决问题。

### 6. 模拟器预览
Xcode自带了iOS模拟器，可以在模拟器上预览应用的运行效果。
- **选择模拟器**：点击Xcode工具栏上的设备选择器，选择一个合适的模拟器型号，如 iPhone 14。
- **运行应用**：点击“Run”按钮，Xcode会将应用安装到模拟器上并启动，你可以在模拟器上像操作真实设备一样测试应用的功能和界面。

### 7. 真机体验
除了在模拟器上测试，还可以将应用安装到真实的iOS设备上进行测试。
- **连接设备**：使用Lightning数据线将iOS设备连接到Mac电脑。
- **信任设备**：在设备上点击“信任此电脑”。
- **配置签名**：在Xcode中选择项目，在“Signing & Capabilities”选项卡中，选择你的开发者账号，并确保“Automatically manage signing”选项被勾选。
- **运行应用**：在设备选择器中选择你的iOS设备，点击“Run”按钮，Xcode会将应用安装到设备上并启动。

### 8. 上传App Store
当应用开发和测试完成后，就可以将应用上传到App Store。
- **创建App记录**：登录 [App Store Connect](https://appstoreconnect.apple.com/)，点击“My Apps”，然后点击“+”按钮，选择“New App”，填写应用的基本信息，如名称、描述、图标等，点击“Create”。
- **准备应用二进制文件**：在Xcode中，选择“Product” - “Archive”，Xcode会对项目进行打包，生成一个归档文件。
- **上传归档文件**：归档完成后，会弹出“Organizer”窗口，选择刚刚生成的归档文件，点击“Distribute App”，选择“App Store Connect”，按照指引完成上传过程。

### 9. 提交审核
上传应用二进制文件后，需要提交应用进行审核。
- **填写审核信息**：在App Store Connect中，填写应用的审核信息，如隐私政策、使用说明等。
- **提交审核**：确认所有信息无误后，点击“Submit for Review”，苹果审核团队会对应用进行审核，审核时间一般为几个工作日到几周不等，审核通过后应用就会在App Store上架。 


## Objective-C学习笔记

### 基础知识

1. 对象方法的函数用“-”开头，类方法用“+”；
2. interface和implementation
3. @property属性会自动生成set和get方法；
4. 函数的多个参数和不带名参数传递
5. 常用对象（Java中叫容器）：NSString，NSNumber，NSArray，NSSet
6. 内存自动释放：NSAutoReleasePool
7. OC中的分类： class_name (category_class_name)
8. OC中的协议： class_name <protocol_class_name>
9. iOS的基础组件：NSBuddle和plist。
10. UIView控制器基础。


### 对于Foundation框架的学习

![20250211093831](https://s2.loli.net/2025/02/11/raBDhswSfJd3Ayq.png)


### 装箱和拆箱

![20250211093910](https://s2.loli.net/2025/02/11/75pfWHPlYdC1LZV.png)





## 对于UIViewController的学习理解和应用

`UIViewController` 是 iOS 开发中非常核心且基础的类，在构建 iOS 应用界面和管理界面逻辑方面发挥着关键作用。以下将从学习理解和实际应用两个维度进行详细阐述：

### 学习理解

#### 概念与作用
`UIViewController` 是视图控制器，它负责管理应用程序的一个或多个视图（`UIView`），起到了视图和数据之间的桥梁作用。一个视图控制器通常对应应用程序中的一个屏幕或一个特定的功能模块，负责处理用户与视图的交互、管理视图的生命周期、响应用户事件等。

#### 生命周期
理解 `UIViewController` 的生命周期是掌握其使用的关键，它的生命周期包含多个重要的方法，这些方法在不同阶段被系统自动调用：
1. **实例化**：
    - `initWithNibName:bundle:`：当使用 XIB 文件来设计视图时，会调用这个方法来初始化视图控制器。
    - `init`：默认的初始化方法，若不使用 XIB 文件，一般会使用这个方法进行初始化。
2. **视图加载**：
    - `loadView`：负责创建视图控制器的根视图，若不手动重写该方法，系统会根据情况从 XIB 文件或 Storyboard 加载视图。
    - `viewDidLoad`：视图加载完成后调用，通常在此方法中进行视图的初始化设置、数据加载等操作。
3. **视图即将显示与显示完成**：
    - `viewWillAppear:`：视图即将显示在屏幕上时调用，可以在此进行一些视图显示前的准备工作，如设置导航栏样式等。
    - `viewDidAppear:`：视图已经完全显示在屏幕上时调用，可用于启动一些需要在视图显示后执行的操作，如开始动画等。
4. **视图即将消失与消失完成**：
    - `viewWillDisappear:`：视图即将从屏幕上消失时调用，可用于保存数据、停止动画等操作。
    - `viewDidDisappear:`：视图已经完全从屏幕上消失时调用。
5. **内存警告处理**：
    - `didReceiveMemoryWarning`：当系统内存不足时调用，需要在此方法中释放一些不必要的资源。

#### 视图管理
`UIViewController` 可以管理一个或多个视图，通过 `view` 属性可以访问其根视图。可以在 `viewDidLoad` 方法中添加子视图到根视图上，例如：
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 50)];
    label.text = @"Hello, World!";
    [self.view addSubview:label];
}
```

#### 导航与转场
在 iOS 应用中，`UIViewController` 之间的导航和转场是常见的操作。可以使用 `UINavigationController` 进行导航管理，通过 `pushViewController:animated:` 方法将一个视图控制器压入导航栈，使用 `popViewControllerAnimated:` 方法将当前视图控制器从导航栈中弹出。

### 实际应用

#### 单视图应用
在简单的单视图应用中，一个 `UIViewController` 可以管理整个屏幕的视图和逻辑。例如，创建一个显示静态文本信息的应用，只需要在 `UIViewController` 的 `viewDidLoad` 方法中添加一个 `UILabel` 并设置文本内容即可。

#### 多视图应用
在复杂的多视图应用中，会使用多个 `UIViewController` 来管理不同的屏幕或功能模块。例如，一个电商应用可能有商品列表视图控制器、商品详情视图控制器、购物车视图控制器等。通过导航控制器或标签栏控制器来管理这些视图控制器之间的切换。
```objc
// 创建导航控制器
UINavigationController *navigationController = [[UINavigationController alloc] initWithRootViewController:rootViewController];

// 压入新的视图控制器
DetailViewController *detailViewController = [[DetailViewController alloc] init];
[navigationController pushViewController:detailViewController animated:YES];
```

#### 模态视图展示
除了使用导航控制器进行视图切换，还可以使用模态视图的方式展示新的视图控制器。模态视图会覆盖当前视图，通常用于展示临时的信息或进行一些独立的操作。
```objc
SecondViewController *secondViewController = [[SecondViewController alloc] init];
[self presentViewController:secondViewController animated:YES completion:nil];
```

#### 数据传递
在不同的 `UIViewController` 之间传递数据也是常见的需求。可以通过属性赋值、代理模式、通知中心等方式实现数据的传递。例如，在从商品列表视图控制器跳转到商品详情视图控制器时，需要将商品的详细信息传递过去，可以通过在商品详情视图控制器中定义属性来接收数据：
```objc
// 在商品详情视图控制器中定义属性
@interface DetailViewController : UIViewController
@property (nonatomic, strong) Product *product;
@end

// 在商品列表视图控制器中传递数据
DetailViewController *detailViewController = [[DetailViewController alloc] init];
detailViewController.product = selectedProduct;
[self.navigationController pushViewController:detailViewController animated:YES];
```





