Title: 使用Swift开发iOS8应用程序
Date: 2015-02-18 20:20
Modified: 2015-02-18 20:20
Slug: swift
Authors: Joey Huang
Summary: 本文是读书笔记
Status: draft

[TOC]

## 使用Swift开发iOS8应用程序

Stanford公开课：[iTunes Stanford CS193p][1]

### 前言

前三课从头开发一个计算器 demo 应用，对使用 Swift 来开发 iOS 应用做了较全面的介绍。特别是作者对编程思路的讲解，大量 xcode 使用技巧和最佳实践的说明都很有价值。学完这三课结合这三课的作业和阅读内容。基本能对使用 Swift 来开发 iOS 应用所需要的一些最基础的概念有较好的理解。


### 第一课

* UI: StoryBoard, Sence
* Controler: UIViewController
* Outlets: 把 UI 控件和 Swift 代码关联起来
* Actions: 设置 UI 控件的事件处理函数
* 访问文档: 按住 ALT 键，鼠标点击相应的类或变量
* Optionals: 这个是 Swift 引入的一个非常重要的概念

!!! note "Optional"
    关于 Optionals 可以参阅 The Swift Programming Language 里的 `Optional Chaining` 一节的内容。里面有一个详细的示例程序。

### 第二课

* AutoLayout: iOS 控件排版系统
* Contraints: 排版规则
* Document Outline: StoryBoard 树形结构
* Computed Properties: 计算属性，包含 get, set 回调函数
* Closure: 闭包，特别是闭包的简化写法，包括省略参数使用 $0, $1 代替；省略 return 关键字；把最后一个参数放在最后函数拨号之后等。
* MVC: Controller 可以直接和 Model 和 View 通信。View 可以发送 Action 到 Controller 里的某个 Target，View 可以通过 delegate 在 Controller 里设置代理。Model 不能直接和 Controller 通信，他们之间使用 subscriber 模式进行通信，Model 设置一个通知中心，Controller 通过向通知中心注册事件通知的方式来获取 Model 的变化信息。

*关于 Closure，视频教程里最终把一段复杂的代码简化为如下简单的代码*

```swift
	performOperatoer() { $0 * $1 }
```	
### 第三课

* Class v.s Structure
	* 类可以继承，但结构体不行
	* 类是引用传递，但结构体是值传递
	* 在 Swift 里，Array, Dictionary 是结构体，不是类，所以当他们作为参数时是值传递，而不是引用传递。最终导致的结果是当 Array, Dictionary 作为参数传递给函数时，是只读的。因为这些值是拷贝过去的，函数里对这个 Array, Dictionary 所作的修改不会返回给调用者。
* Swift 习惯用法
	* `var ops = Array<Double>()` -> `var ops = [Double]()`
	* `var ops = Dictionary<String, Double>()` -> `var ops = [String: Double]()`
* Enumulation: Swift 提供了强大的枚举类型，结合 switch 语句，可以实现很灵活的功能。具体参阅本课54分的视频。另可参阅 The Swift Programming Language 里 Swift Tour 里的 `Enumerations and Structures`
	* 枚举的值：可以是传统的数字，也可以是字符串或其他任意自定义的类型
	* 枚举可以有自己的成员函数
* 函数可以返回 Tuple，以便返回多个参数，调用者可以结合 `_` 来选择 Tuple 里感兴趣的返回值
* Printable: 在类，结构体，枚举等命名类型掭添加 `: Printable`，然后在命名类型里添加 `description: String` 的只读的计算属性，即可重定义这个类型的字符串表达形式，即可以通过 `println()` 来把这个类型的值打印出来。

### 第四课

主要针对第三课的 reading: Project 1 的内容进行讲解。

*  `NSNumber`: 调用 Object-C 代码时需要用到
*  `NSDate`: 表达时间日期的数据结构
*  `NSObject`: 所有 Object-C 的在 Swift 里映射的类的子类
*  `NSData`: 字节流类
*  Class v.s Struct: 大部分时候，我们使用 class 而不使用 struct。struct 是给 Foundation 使用的。

####函数参数

在 Swift 里的函数参数全部是命名参数。默认情况下，第一个参数调用者可以不用指定名字，而第二个起，需要指定参数名字。

####Properties Observer

可以通过 `willSet`, `didSet` 来监控参数的改变。

```swift
var prop: Int = 4 {
	willSet {
		println("prop will set to \(newValue)")
	}
	didSet {
		println("prop did set, the old value is \(oldValue)")
	}
}

// for inherit properties observer
override var prop {
	willSet {
	}
	didSet {
	}
}
// 问题: 给继承的属性添加监听器后，父亲类的 willSet, didSet 会自动被调用吗？
```

####Lazy init

Lazy init的属性只有在第一次被访问时才会被初始化，对那些开销比较大的属性这是个非常好的机制，可以让类快速创建。

```swift
lazy var brain = CalculatorBrain()

lazy var prop: Type = {
	// execute a closure to init a expensive prop
	return Type()
}
 
// call a function to init a property
lazy var prop = self.initSomeExpensiveProp()
```

####构造函数

阅读构造函数的几个规则，用代码来演示。

####AnyObject

AnyObject 可以指向任意的类。使用时可以使用 as 或 as? 来转化，也可以使用 is 关键字来判断是不是属于一个特定的类。

```swift
var item: AnyObject = ...

// force to convert, will crash if item is not a UIBarButtonItem
let toolBar = item as UIBarButtonItem

if let toolBar = item as? UIBarButtonItem {
	// do something with toolBar, it's now a UIBarButtonItem
}

if item is UIBarButtonItem {
	let toolBar = item as UIBarButtonItem
}

```

####Array<T>

数组提供了一些很有意思的方法。`sort`，`replace`, `insert`等。另外还有一些可以批量处理数组里的元素的方法。

```swift
// 过滤一个数组，返回满足条件的一个新数组
filter(includeElement: (T) -> Bool) -> [T]

// 把一个数组映射到另外一个数组，比如把一个按钮全部转化为其对应的字符串数组
map(transform: (T) -> U) -> [U]
// 利用 map 把数组转化为字符串数组
let strs: [String] = [1, 2, 3].map { "\($0)" }

// 对数组里的元素进行运算，返回一个值
reduce(initial: U, combine: (U, T) -> U) -> U
// 利用 reduce 对数组求各
let sum: Int = [1, 2, 3].reduce(0) { $0 + $1 }
```

####关于教学方式

这一节里讲解了大量比较枯燥的语法和基础知识。从这里也可以看到教学的脉络：先用一个合适的 demo 来给学生建立成就感和兴趣。然后再回到基础性的和原理性的内容上。

使用示例代码来讲解复杂概念，比如针对 Optional 的讲解：

```swift
// Optional 的定义
enum Optional<T> {
	case None
	case Some(T)
}

let n: Double? = nil
// 等价于
let n = Optional<Double>.None

let n: Double? = 5
// 等价于
let n = Optional<Double>.Some(5)

n!
// 等价于
switch Optional<T> {
	case .None:
		// raise a runtime error
		return nil
	case .Some(let v):
		return v
}
```
类似这种伪代码的解释，非常清晰地解释了一些复杂的概念。

### 第五课

### 第六课

####Protocols

前面主要介绍了 protocol，其概念类似 Java 里面的 interface。其和 Java 的 interface 有一个区别就是，swift 的 protocol 支持 optional metnod。

```swift
@objc protocol Car {
	optional func fillGas()
	optional func fillWater() -> Bool
}
```

有几个关键点：

* optional 协议必须添加 `@objc` 前缀
* 添加了 `@objc` 前缀的 protocol 有以下几个特性：
	* 只能由类来实现，不能由结构体和枚举实现
	* 可以使用 `is`, `as`, `as?` 来判断是否实现特定协议
	* 只能使用和 Object-C 兼容的数据类型，比如不能使用 swfit 特有的 optional

####Delegate

由 `protocol` 引入了一个 MVC 的设计理念。View 和 Controller 通信时，View 不能直接引用 Controller，view 通过 protocol 来设置一个 Controller 的委托代理来实现通信。主要步骤如下：

* 创建一个代理协议 (delegation protocol)，描述用到这个 View 的 Controller 需要具备哪些特性，需要实现哪些协议
* 在 View 里创建一个委托代理属性 (delegate property)，其数据类型就是这个 protocol
* 在 View 里使用这个 protocol 实例来和使用这个 View 的 Controller 通信
* Controller 声明它支持这个协议
* Controller 把自己设置成 View 的委托代理属性 (步骤 2 里创建的)
* Controller 实现这个协议

```swift
/**
 * MARK: code in View 
 */

// define delegation protocol, it can only implement by class
protocol HappinessDataSource: class {
	func smilinessForFaceView(sender: UIView) -> Double?
}

// define delegate property as weak reference
weak var dataSource: HappinessDataSource? = nil

// use delegate property to do/get things not belong to View
let smiliness = dataSource?.smilinessForFaceView(self) ?? 0.5

/**
 * MARK: code in Controller 
 */

// declare that this will implement HappinessDataSource protocol
HappinessViewController: UIViewController, HappinessDataSource {

    @IBOutlet weak var faceView: FaceView! {
        didSet {
        	// set myself as View's delegate property
            faceView.dataSource = self
        }
    }

	// implement delegation protocol
    func smilinessForFaceView(sender: FaceView) -> Double? {
        return ((Double)(happiness - 50)) / 50
    }
}
```

!!! note "week reference"
    `dataSource` 定义为 `weak var` 这个涉及到内存管理，它不会阻止 dataSource 指向的实例被回收。另外由于 swift 里只有类才涉及到引用传递，所以这里的 dataSource 必须指向一个类，而不能是 struct 或 enum。所以，这里的 protocol 定义为 class，即只有类才能实现这个协议。

####TIPS

* `??` 运算符：`let v = dataSource?.getData(i, self) ?? "empty"`
* Command + Shift + o: 快速打开文件或变量

####Gesture

使用 iOS 内置的手势识别类来处理用户操作事件。关键有两个动作。

* 给 View 添加动作识别
* 提供一个函数来处理这个动作

```swift
@IBOutlet weak var pannableView: UIView {
    didSet {
        let recognizer = UIPanGestureRecognizer(target: self, action: “pan:”)
        pannableView.addGestureRecognizer(recognizer)
    }
}

func pan(gesture: UIPanGestureRecognizer) {
	switch gesture.state {
	case .Changed: fallthrough
	case .Ended:
		let translation = gesture.translationInView(pannableView)
		// update anything that depends on the pan gesture using translation.x and .y
		gesture.setTranslation(CGPointZero, inView: pannableView)
	default: break
	} 
}

```

[1]: https://itunesu.itunes.apple.com/WebObjects/LZDirectory.woa/ra/directory/courses/961180099/feed
