---
layout: post
title: 创建自定义动画按钮
description: 独家翻译
image: assets/images/pic02.jpg
---

![](https://camo.githubusercontent.com/1455975236a17dabfd7ba2117a087a95b0210fd6/687474703a2f2f72616d6f736d61636861646f2e6e65742f77702d636f6e74656e742f75706c6f6164732f323031352f30322f73746172626f756e63652e676966)

[被追波设计的启发](https://dribbble.com/shots/1555501-Star-Bounce-Animation),我们将要去创造一个自定义的动画按钮。我希望通过这个教程，当说到去编码动效，你能至少理解一些重要的步骤。
### 理解动效
首先，在我们打开Xcode，跟变换（transform），图层（CALayer）的子类们等等玩耍之前，我们需要弄懂这个我们将要去构造实际动画模型是什么，这是非常重要的。盯着这个模型看一会儿，然后思考你将提供说明来简单的描述发生了什么。[EZGIF](http://ezgif.com/speed)提供一些非常好用，用于gif图的工具，例如，改变动画速度，然后在慢速度中查看动效。
下面的慢速度的图可能有帮助：
![](https://s3.amazonaws.com/swiftcast.tv/custom_uploads/Creating+Custom+Animated+Buttons/star-slow.gif)   
让我们开始打破动效图层，然后命名他们来获得一个更方便的引用：
![1:star 2: Fill 3: Ring](https://s3.amazonaws.com/swiftcast.tv/custom_uploads/Creating+Custom+Animated+Buttons/start-breakdown.png)
好了，现在我们已经标记了这些图层，我们可以用帧分解来描述每一层的发生情况。下面是一个动效步骤的简单描述：
1.星图层和环图层几乎同步增长。
2.星图层和环图层停止增长，然后用比第一步更快的速度开始缩小到星图层的中心。
3.填充圆从星图层中心开始增长。
4.星图层增长到初始大小，并且填充色改变了。
5.填充圆增长到步骤二里的大小，然后缩小到初始大小。
### 创建路径
在iOS中CGPaths代表的路径并在代码中创建他们是一个痛苦而且耗时的任务。幸运的是的这里有个工具叫[PaintCode](http://www.paintcodeapp.com)使这个任务变得很容易。
所有你需要做的就是创建基于软件[Adobe lllustrator](http://www.adobe.com/br/products/illustrator.html)或者[Sketch](http://www.sketchapp.com)的矢量图形,导出为SVG文件，导入PaintCode,然后PaintCode会提供你这个图形OC和Swift的代码。

这下面的代码代表星图层：

~~~  
    var star = UIBezierPath()
    star.moveToPoint(CGPointMake(112.79, 119))
    star.addCurveToPoint(CGPointMake(107.75, 122.6),controlPoint1: CGPointMake(113.41, 122.8), controlPoint2:       CGPointMake(111.14, 124.42))
    star.addLineToPoint(CGPointMake(96.53, 116.58))
    star.addCurveToPoint(CGPointMake(84.14, 116.47), controlPoint1: CGPointMake(93.14, 114.76), controlPoint2: CGPointMake(87.56, 114.71))
    star.addLineToPoint(CGPointMake(72.82, 122.3))
    star.addCurveToPoint(CGPointMake(67.84, 118.62), controlPoint1: CGPointMake(69.4, 124.06), controlPoint2: CGPointMake(67.15, 122.41))
    star.addLineToPoint(CGPointMake(70.1, 106.09))
    star.addCurveToPoint(CGPointMake(66.37, 94.27), controlPoint1: CGPointMake(70.78, 102.3), controlPoint2: CGPointMake(69.1, 96.98))
    star.addLineToPoint(CGPointMake(57.33, 85.31))
    star.addCurveToPoint(CGPointMake(59.29, 79.43), controlPoint1: CGPointMake(54.6, 82.6), controlPoint2: CGPointMake(55.48, 79.95))
    star.addLineToPoint(CGPointMake(71.91, 77.71))
    star.addCurveToPoint(CGPointMake(81.99, 70.51), controlPoint1: CGPointMake(75.72, 77.19), controlPoint2: CGPointMake(80.26, 73.95))
    star.addLineToPoint(CGPointMake(87.72, 59.14))
    star.addCurveToPoint(CGPointMake(93.92, 59.2), controlPoint1: CGPointMake(89.46, 55.71), controlPoint2: CGPointMake(92.25, 55.73))
    star.addLineToPoint(CGPointMake(99.46, 70.66))
    star.addCurveToPoint(CGPointMake(109.42, 78.03), controlPoint1: CGPointMake(101.13, 74.13), controlPoint2: CGPointMake(105.62, 77.44))
    star.addLineToPoint(CGPointMake(122, 79.96))
    star.addCurveToPoint(CGPointMake(123.87, 85.87), controlPoint1: CGPointMake(125.81, 80.55), controlPoint2: CGPointMake(126.64, 83.21))
    star.addLineToPoint(CGPointMake(114.67, 94.68))
    star.addCurveToPoint(CGPointMake(110.75, 106.43), controlPoint1: CGPointMake(111.89, 97.34), controlPoint2: CGPointMake(110.13, 102.63))
    star.addLineToPoint(CGPointMake(112.79, 119))
~~~
我们可以在iOS中手动创建简单的图形，例如圆图层：

~~~
    let circle = UIBezierPath(ovalInRect: inFrame)
~~~

现在我们可以创建自定义的按钮了。
### 连接图层
Xcode6 带来一个叫做[活动视图(live views）](https://developer.apple.com/videos/wwdc/2014/#411)的新科技,这使得处理自定义布局代码更容易，并且不用运行或者构建视图就能提供立即的可视化的反馈。所以让我们使用活动视图！为你的类使用活动视图非常简单：
1.放置关键词***@IBDesignable*** 在类声明之上
2.放置关键词***@IBInspectable*** 在一个你想在故事版的属性检查器上改变的变量上。
3.重写 ***layoutSubviews()***.这是一个你将添加子视图和子图层的地方。
为了我们的星星按钮，我们将创建一个UIButton 的子类并且遵循下面的步骤：

~~~
    @IBDesignable
    class StarButton: UIButton
    {
      private var starShape: CAShapeLayer!
      private var outerRingShape: CAShapeLayer!
      private var fillRingShape: CAShapeLayer!

      @IBInspectable
      var lineWidth: CGFloat = 1 {
        didSet {
        updateLayerProperties()
       }
    }

    @IBInspectable
    var favoriteColor: UIColor = UIColor(hex:"eecd34") {
      didSet {
        updateLayerProperties()
      }
    }

    @IBInspectable
    var notFavoriteColor: UIColor = UIColor(hex:"9e9b9b") {
      didSet {
        updateLayerProperties()
      }
    }

    @IBInspectable
    var starFavoriteColor: UIColor = UIColor(hex:"9e9b9b") {
      didSet {
        updateLayerProperties()
      }
    }

    var isFavorite: Bool = false {
      didSet {
        return self.isFavorite ? favorite() : notFavorite()
      }
    }

    private func updateLayerProperties()
    {
      if fillRingShape != nil
      {
        fillRingShape.fillColor = favoriteColor.CGColor
      }

      if outerRingShape != nil
      {
        outerRingShape.lineWidth = lineWidth
        outerRingShape.strokeColor = notFavoriteColor.CGColor
      }

      if starShape != nil
      {
        starShape.fillColor = isFavorite ? starFavoriteColor.CGColor :   notFavoriteColor.CGColor
      }
    }

    override func layoutSubviews()
    {
      super.layoutSubviews()
      updateLayerProperties()
     }
    }
~~~

转到故事版，创建一个按钮，把它设置成自定义，去掉上面默认的文本，然后设置成我们的StarButton类。

![](https://s3.amazonaws.com/swiftcast.tv/custom_uploads/Creating+Custom+Animated+Buttons/custom-class.png)

在属性检查器上设置可视化属性。

![](https://s3.amazonaws.com/swiftcast.tv/custom_uploads/Creating+Custom+Animated+Buttons/ibinspectables.png)

然后看！

![](https://s3.amazonaws.com/swiftcast.tv/custom_uploads/Creating+Custom+Animated+Buttons/storyboard.png)

### 动效
创建动画总是一点点尝试和失败。当然，即使你有动画背景，你也不会流畅的构建动效。为了节约时间，我会直接跳向代码。为了创建动效，我已经把这个动效分成五个步骤。我们要处理的主要属性是用来转换大小的CATransform3D，用来改变阿尔法值的opacity,用来改变颜色的fillColor.
我们假定我们用这些按钮来点赞，然后这样命名函数。


~~~

    private func favorite()
    {
    // 1. Star grows
    var starGoUp = CATransform3DIdentity
    starGoUp = CATransform3DScale(starGoUp, 1.5, 1.5, 1.5)

    // 2. Star stop growing and starts shrinking
    var starGoDown = CATransform3DIdentity
    starGoDown = CATransform3DScale(starGoDown, 0.01, 0.01, 0.01)

    // Configure a keyframe animation with both transforms (grow and shrink)
    let starKeyFrames = CAKeyframeAnimation(keyPath: "transform")
    starKeyFrames.values = [
      NSValue(CATransform3D:CATransform3DIdentity),
      NSValue(CATransform3D:starGoUp),
      NSValue(CATransform3D:starGoDown)
    ]
    starKeyFrames.keyTimes = [0.0,0.4,0.6]
    starKeyFrames.duration = 0.4
    starKeyFrames.beginTime = CACurrentMediaTime() + 0.05
    starKeyFrames.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)

    // This is VERY important when you're working with relative time, remove and odd things will happen
    starKeyFrames.fillMode =  kCAFillModeBackwards
    starKeyFrames.setValue(favoriteKey, forKey: starKey)

    // Let the notification tell us when it's over
    starKeyFrames.delegate = self
    starShape.addAnimation(starKeyFrames, forKey: favoriteKey)
    starShape.transform = starGoDown

    // 1. Ring grows
    var grayGoUp = CATransform3DIdentity
    grayGoUp = CATransform3DScale(grayGoUp, 1.5, 1.5, 1.5)

    // 2. Ring stop growing and starts shrinking
    var grayGoDown = CATransform3DIdentity
    grayGoDown = CATransform3DScale(grayGoDown, 0.01, 0.01, 0.01)

    let outerCircleAnimation = CAKeyframeAnimation(keyPath: "transform")
    outerCircleAnimation.values = [
      NSValue(CATransform3D:CATransform3DIdentity),
      NSValue(CATransform3D:grayGoUp),
      NSValue(CATransform3D:grayGoDown)
    ]
    outerCircleAnimation.keyTimes = [0.0,0.4,0.6]
    outerCircleAnimation.duration = 0.4
    outerCircleAnimation.beginTime = CACurrentMediaTime() + 0.01
    outerCircleAnimation.fillMode =  kCAFillModeBackwards
    outerCircleAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)

    outerRingShape.addAnimation(outerCircleAnimation, forKey: "Gray circle Animation")
    outerRingShape.transform = grayGoDown

    // 3. Fill Circle grows from Star's center.
    var favoriteFillGrow = CATransform3DIdentity
    favoriteFillGrow = CATransform3DScale(favoriteFillGrow, 1.5, 1.5, 1.5)

    // 5. Fill Circle grows until reach the size of step 2 and shrink back to the initial size.
    let fillCircleAnimation = CAKeyframeAnimation(keyPath: "transform")

    fillCircleAnimation.values = [
      NSValue(CATransform3D:fillRingShape.transform),
      NSValue(CATransform3D:favoriteFillGrow),
      NSValue(CATransform3D:CATransform3DIdentity)
    ]
    fillCircleAnimation.keyTimes = [0.0,0.4,0.6]
    fillCircleAnimation.duration = 0.4
    fillCircleAnimation.beginTime = CACurrentMediaTime() + 0.22
    fillCircleAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)
    fillCircleAnimation.fillMode =  kCAFillModeBackwards

    let favoriteFillOpacity = CABasicAnimation(keyPath: "opacity")
    favoriteFillOpacity.toValue = 1
    favoriteFillOpacity.duration = 1
    favoriteFillOpacity.beginTime = CACurrentMediaTime()
    favoriteFillOpacity.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)
    favoriteFillOpacity.fillMode =  kCAFillModeBackwards

    fillRingShape.addAnimation(favoriteFillOpacity, forKey: "Show fill circle")
    fillRingShape.addAnimation(fillCircleAnimation, forKey: "fill circle Animation")
    fillRingShape.transform = CATransform3DIdentity
    }
~~~
    
当第一部分的动效结束，第二部分的动效会触发

~~~
    private func endFavorite()
    {
    // just a helper to run this piece of code with default actions disabled
    executeWithoutActions {
    self.starShape.fillColor = self.starFavoriteColor.CGColor
    self.starShape.opacity = 1
    self.fillRingShape.opacity = 1
    self.outerRingShape.transform = CATransform3DIdentity
    self.outerRingShape.opacity = 0
    }

    // 4. Star grows to it's initial size, and the filling color is changed.
    let starAnimations = CAAnimationGroup()
    var starGoUp = CATransform3DIdentity
    starGoUp = CATransform3DScale(starGoUp, 2, 2, 2)

    let starKeyFrames = CAKeyframeAnimation(keyPath: "transform")
    starKeyFrames.values = [
    NSValue(CATransform3D: starShape.transform),
    NSValue(CATransform3D:starGoUp),
    NSValue(CATransform3D:CATransform3DIdentity)
    ]
    starKeyFrames.keyTimes = [0.0,0.4,0.6]
    starKeyFrames.duration = 0.2
    starKeyFrames.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionLinear)

    starShape.addAnimation(starKeyFrames, forKey: nil)
    starShape.transform = CATransform3DIdentity
    }  
~~~

对于取消收藏，这个动效没有秘密：

~~~
    private func notFavorite()
    {
    let starFillColor = CABasicAnimation(keyPath: "fillColor")
    starFillColor.toValue = notFavoriteColor.CGColor
    starFillColor.duration = 0.3

    let starOpacity = CABasicAnimation(keyPath: "opacity")
    starOpacity.toValue = 0.5
    starOpacity.duration = 0.3

    let starGroup = CAAnimationGroup()
    starGroup.animations = [starFillColor, starOpacity]

    starShape.addAnimation(starGroup, forKey: nil)
    starShape.fillColor = notFavoriteColor.CGColor
    starShape.opacity = 0.5

    let fillCircle = CABasicAnimation(keyPath: "opacity")
    fillCircle.toValue = 0
    fillCircle.duration = 0.3
    fillCircle.setValue(notFavoriteKey, forKey: starKey)
    fillCircle.delegate = self

    fillRingShape.addAnimation(fillCircle, forKey: nil)
    fillRingShape.opacity = 0

    let outerCircle = CABasicAnimation(keyPath: "opacity")
    outerCircle.toValue = 0.5
    outerCircle.duration = 0.3

    outerRingShape.addAnimation(outerCircle, forKey: nil)
    outerRingShape.opacity = 0.5
    }
~~~

### 一个附赠品
因为Swift在iOS社区是一个真正的明星，我们庆祝一下！
按照下面这个CGPath修改这个星图层路径：

~~~
    var swiftPath = UIBezierPath()
    swiftPath.moveToPoint(CGPointMake(376.2, 283.2))
    swiftPath.addCurveToPoint(CGPointMake(349.8, 238.4),controlPoint1: CGPointMake(367.4, 258.4), controlPoint2: CGPointMake(349.8, 238.4))
    swiftPath.addCurveToPoint(CGPointMake(236.5, 0), controlPoint1: CGPointMake(349.8, 238.4), controlPoint2: CGPointMake(399.7, 105.6))
    swiftPath.addCurveToPoint(CGPointMake(269, 180.8), controlPoint1: CGPointMake(303.7, 101.6), controlPoint2: CGPointMake(269, 180.8))
    swiftPath.addCurveToPoint(CGPointMake(181.29, 117.07), controlPoint1: CGPointMake(269, 180.8), controlPoint2: CGPointMake(211.4, 140.8))
    swiftPath.addCurveToPoint(CGPointMake(85, 33.6), controlPoint1: CGPointMake(151.18, 93.35), controlPoint2: CGPointMake(85, 33.6))
    swiftPath.addCurveToPoint(CGPointMake(145, 117.07), controlPoint1: CGPointMake(85, 33.6), controlPoint2: CGPointMake(128.15, 96.31))
    swiftPath.addCurveToPoint(CGPointMake(185.78, 163.66), controlPoint1: CGPointMake(161.85, 137.84), controlPoint2: CGPointMake(185.78, 163.66))
    swiftPath.addCurveToPoint(CGPointMake(136.36, 129.42), controlPoint1: CGPointMake(185.78, 163.66), controlPoint2: CGPointMake(161.07, 147.39))
    swiftPath.addCurveToPoint(CGPointMake(34.6, 50.4), controlPoint1: CGPointMake(111.65, 111.46), controlPoint2: CGPointMake(34.6, 50.4))
    swiftPath.addCurveToPoint(CGPointMake(133.8, 169.2), controlPoint1: CGPointMake(34.6, 50.4), controlPoint2: CGPointMake(82.69, 119.24))
    swiftPath.addCurveToPoint(CGPointMake(214.6, 244), controlPoint1: CGPointMake(184.91, 219.16), controlPoint2: CGPointMake(214.6, 244))
    swiftPath.addCurveToPoint(CGPointMake(129.8, 264.8), controlPoint1: CGPointMake(214.6, 244), controlPoint2: CGPointMake(196.2, 263.2))
    swiftPath.addCurveToPoint(CGPointMake(0, 221), controlPoint1: CGPointMake(63.4, 266.4), controlPoint2: CGPointMake(0, 221))
    swiftPath.addCurveToPoint(CGPointMake(206.6, 339.2), controlPoint1: CGPointMake(0, 221), controlPoint2: CGPointMake(62.5, 339.2))
    swiftPath.addCurveToPoint(CGPointMake(325, 304.8), controlPoint1: CGPointMake(270.6, 339.2), controlPoint2: CGPointMake(288.93, 304.8))
    swiftPath.addCurveToPoint(CGPointMake(383.3, 339.2), controlPoint1: CGPointMake(361.07, 304.8), controlPoint2: CGPointMake(381.7, 340))
    swiftPath.addCurveToPoint(CGPointMake(376.2, 283.2), controlPoint1: CGPointMake(384.9, 338.4), controlPoint2: CGPointMake(385, 308))
    return swiftPath.CGPath
~~~
    
这就可以了！偶也！Swift!

![](https://s3.amazonaws.com/swiftcast.tv/custom_uploads/Creating+Custom+Animated+Buttons/swift.gif)

就是这个！我希望你能理解多一点关于创建动效。去我的[Github](https://github.com/rakaramos/StarButton)找到完整的工程。当我添加一些新图形的时候，我会升级代码，所以注意这个仓库！如果你喜欢这个项目，请分享并且粉一下这个git仓库！


