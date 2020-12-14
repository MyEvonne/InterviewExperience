#UIKit相关问题
## 1.UIView与CALayer的关系和区别
1. CALayer继承自NSObject，而UIView继承自UIRespond，所以UIView可以进行事件传递响应。
1. CALayer是UIView的一个属性，用来显示UIView上面的布局。

## 2.UIView与CALayer在动画上的区别
在做 iOS 动画的时候，修改非 RootLayer的属性（譬如位置、背景色等）会默认产生隐式动画，而修改UIView则不会。  
对于每一个 UIView 都有一个 layer,把这个 layer 且称作RootLayer,而不是 View 的根 Layer的叫做 非 RootLayer。我们对UIView的属性修改时时不会产生默认动画，而对单独 layer属性直接修改会，这个默认动画的时间缺省值是0.25s.  
在 Core Animation 编程指南的 “How to Animate Layer-Backed Views” 中，对为什么会这样做出了一个解释：  
UIView 默认情况下禁止了 layer 动画，但是在 animation block 中又重新启用了它们  
是因为任何可动画的 layer 属性改变时，layer 都会寻找并运行合适的 'action' 来实行这个改变。在 Core Animation 的专业术语中就把这样的动画统称为动作 (action，或者 CAAction)。  
layer 通过向它的 delegate 发送 actionForLayer:forKey: 消息来询问提供一个对应属性变化的 action。delegate 可以通过返回以下三者之一来进行响应：  
它可以返回一个动作对象，这种情况下 layer 将使用这个动作。  
它可以返回一个 nil， 这样 layer 就会到其他地方继续寻找。  
它可以返回一个 NSNull 对象，告诉 layer 这里不需要执行一个动作，搜索也会就此停止。
当 layer 在背后支持一个 view 的时候，view 就是它的 delegate；  
总接来说就是如下几点：  
每个 UIView 内部都有一个 CALayer 在背后提供内容的绘制和显示，并且 UIView 的尺寸样式都由内部的 Layer 所提供。两者都有树状层级结构，layer 内部有 SubLayers，View 内部有 SubViews.但是 Layer 比 View 多了个AnchorPoint
在 View显示的时候，UIView 做为 Layer 的 CALayerDelegate,View 的显示内容由内部的 CALayer 的 display  
CALayer 是默认修改属性支持隐式动画的，在给 UIView 的 Layer 做动画的时候，View 作为 Layer 的代理，Layer 通过 actionForLayer:forKey:向 View请求相应的 action(动画行为)  
layer 内部维护着三分 layer tree,分别是 presentLayer Tree(动画树),modeLayer Tree(模型树), Render Tree (渲染树),在做 iOS动画的时候，我们修改动画的属性，在动画的其实是 Layer 的 presentLayer的属性值,而最终展示在界面上的其实是提供 View的modelLayer
https://www.cnblogs.com/xujinzhong/p/11155084.html
## 3.frame和bounds在什么情况下是不相等的？bounds的x，y一定是0，0么？为什么？bounds改成（50，50，width，height）会发生什么？view本身与子view？


## 4.手指触摸屏幕以后系统都做了那些事情
屏幕->application->find best responder.
responde chain
https://mp.weixin.qq.com/s/SWcCXL0o05tFfLbSjS49Ag
https://gsl201600.github.io/2019/12/25/iOS%E4%B8%AD%E4%BA%8B%E4%BB%B6%E7%9A%84%E5%93%8D%E5%BA%94%E9%93%BE%E5%92%8C%E4%BC%A0%E9%80%92%E9%93%BE/

## 5.Frame与Bounds的区别，如果bounds的origin不是00会怎样
frame指的是view在父view的坐标系内的origin，bounds是view自身内部的坐标系。
https://www.jianshu.com/p/b59f0b69c52f
