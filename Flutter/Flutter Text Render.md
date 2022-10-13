[TOC]

RenderObjectWidget是一个蓝图。 它包含RenderObject的配置信息，RenderObject完成了所有艰难的hit testing 和绘制UI的工作。

![](https://tva1.sinaimg.cn/large/008vxvgGly1h72sb778pzj310m0fedh4.jpg)



下面的图展示一些RenderObject的子类.最常见的一个是RenderBox, RenderBox 定义了一个屏幕上的矩形区域用来进行绘制。RenderBox的众多子类之一，RenderParagraph，是Flutter用来绘制文本的工具。

![](https://koenig-media.raywenderlich.com/uploads/2019/07/renderobject_types.png)

猜猜？再过一会，你将从头开始创建你自己的Render Paragraph。我知道，我也等不及了!



如您所知，Flutter中的布局是通过将widgets组合到树中来实现的。在底层实现中有一个相对应的渲染树（render object  tree）。但是widgets和render objects不知道如何进行交互。Widgets不能生成一个render object 树，render objects也不清楚widget 树何时改变。



这就是elements的作用。有一棵对应的element树，对于widget树上的每一个widget有相应的Element。elements包含widgets和对应的render objects的引用。elements是widgets和render objects(呈现对象) 之间的管理器或者说是中介者。elements知道什么时候去创建render objects，将它们放到树上的什么位置，当有变化时候去更新它们，以及何时从子部件膨胀(创建)新elements。



下图显示了主要的Element子类。每一个widget有一个对应的element，所以你会注意到名字和Widget子类的相似之处。



![](https://koenig-media.raywenderlich.com/uploads/2019/07/element_types.png)



> 有趣的事实：你一直都在和elements直接互动，但是你没有意识到这一点。你知道BuildContext么？这其实只是Element的别名。或者更专业的说，BuildContext是Element实现的一个抽象类。



上面的解释是我在去目的地的路上在旅游车上说的话。让我们现在下车，走下去看看一个真实的例子。

### Stepping in：The Text Widget

现在，你将深入Flutter的源码，看看widgets、elements和render objects是如何实际使用的。你将跟随Text widget的渲染对象(render object)的创造轨迹，也就是说，`RenderParagraph` 

不用担心，我会与你一路相伴。

在Android Studio 3.4 或者更高的版本中，打开您的starter项目，并运行。您需要首先运行 `flutter pub get` 来获取项目的依赖项。在Android Studio中，你可以使用打开项目后弹出的Get dependencies来实现这一点。

APP运行起来后，你应该会看到一个名为Steppe Up 的旅游应用程序的欢迎页面，底部的文字是`travel mongolian` 。

<img src="https://koenig-media.raywenderlich.com/uploads/2019/07/app_travel_mongolia.png" style="zoom:50%;" />



现在停止运行APP。

在 ` lib/main.dart `  ,滚动到底部，找到  TODO: Start your journey here line:

```dart
child: Align(
  alignment: Alignment.bottomCenter,
  child: Text( // TODO: Start your journey here
    Strings.travelMongolia,
```

这棵widget树有一个孩子是Text widget 的 Align widget 。当你逐步浏览源代码时，你可以参考下面的图表:

![](https://koenig-media.raywenderlich.com/uploads/2019/07/tree.png)

请执行如下步骤：

1. Command-click(或者 Control-click) `Text`  跳转到它的的源码定义. 注意，  `Text`是一个stateless widget.
2. 向下滚动的 `build`方法。这个方法返回了什么？Surprise！它返回了一个 `RichText`  widget。 事实证明，`Text` 只是 `RichText` 的伪装。

3. Command-click `RichText` 进入到它的源码定义。注意到`RichText` 是一个 `MultiChildRenderObjectWidget` . 为什么是*multi*-child ? 在1.7之前的Flutter以前版本中，它实际上曾经是一个`LeafRenderObjectWidget` ，它没有子组件，但现在RichText支持具有[widget spans](https://api.flutter.dev/flutter/widgets/WidgetSpan-class.html)的内联小部件。
4. 向下滚动到`createRenderObject` 方法, 在这里，这是它创建`RenderParagraph`的地方。
5. 添加一个断点到 *return RenderParagraph* 这一行。
6. 在debug 模式下再一次运行程序。

在Android Studio中，如果你选择了Debug和Variables选项卡，你应该会看到类似以下内容:

![](https://koenig-media.raywenderlich.com/uploads/2019/07/debug_richtext.png)



您还应该得到下面的堆栈跟踪，其中顶部有这些行. 我在括号中添加了widget或element类型. 最右边的数字参考了下面的评论. 

```dart
RichText.createRenderObject             (RichText)    // 8
RenderObjectElement.mount               (RichText)    // 7
MultiChildRenderObjectElement.mount     (RichText)
Element.inflateWidget                   (Text)        // 6
Element.updateChild                     (Text)
ComponentElement.performRebuild         (Text)        // 5
Element.rebuild                         (Text)
ComponentElement._firstBuild            (Text)
ComponentElement.mount                  (Text)        // 4
Element.inflateWidget                   (Align)       // 3
Element.updateChild                     (Align)       // 2
SingleChildRenderObjectElement.mount    (Align)       // 1
```

让我们跟随`RenderParagraph` 是如何创建的。你不需要点击每一行，但可以从上述的第12行开始:

1. 点击 <u>SingleChildRenderObjectElement.mount.</u> 您处于`Align ` widget的Element中。在您的布局中 `Align` 的child是 `Text` widget。所以被传递到 `updateChild` 的 `widget.child` 是 `Text` widget.

   ```dart
   	// SingleChildRenderObjectElement
   	@override
     void mount(Element? parent, Object? newSlot) {
       super.mount(parent, newSlot);
       // _child 是 Element
       // widget.child 是 Text widget.
       _child = updateChild(_child, (widget as SingleChildRenderObjectWidget).child, null);
     }
   ```

   

2. 点击 <u>Element.updateChild.</u> 在一个很长的方法末尾，您的`Text` widget，形参名为 `newWidget` ，被传入到inflateWidget中

```dart
Element? updateChild(Element? child, Widget? newWidget, Object? newSlot) {
		··· ···
		 newChild = inflateWidget(newWidget, newSlot);
		··· ···
}
```

3. 点击 <u>Element.inflateWidget</u>. 膨胀一个widget意味着从它`创建一个element.` 正如您所看到的发生在元素`newChild = newWidget.createElement()`中。此时，您仍然在 `Align` element中，但是你将要进入刚刚膨胀的 `Text` element的`mount`方法。

```dart
 @protected
  @pragma('vm:prefer-inline')
  Element inflateWidget(Widget newWidget, Object? newSlot) {
  		··· ···
  		final Element newChild = newWidget.createElement();
      newChild.mount(this, newSlot);
  		··· ···
  }
```

4. 单击<u>ComponentElement.mount.</u> 您现在位于Text element中。Component elements<组件元素>(像是`StatelessElement`) 不能够直接创建渲染对象<render objects>. 但是它们创建其他的element，这些element最终将创建渲染对象<render objects>.

5. 下一个令人兴奋的事情是堆栈跟踪上的几个方法。点击 <u>ComponentElement.performRebuild</u> 。找到`built = build()`这一行。 伙计们，这就是调用Text widget 的构建<build>方法的地方。[`StatelessElement`](https://github.com/flutter/flutter/blob/38f849015ce9d38362b41feb8163d33f6c94a7af/packages/flutter/lib/src/widgets/framework.dart#L3974)使用setter方法来添加对自身的引用(点链接看)，作为`BuildContext`参数。构建变量<built variable>是你的`RichText`小部件。

   

   ![](https://tva1.sinaimg.cn/large/008vxvgGly1h73rbfbpmlj31600m041d.jpg)

6. 点击 Element.inflateWidget. 这时newWidget是RichText，同时它被用来创建一个`MultiChildRenderObjectElement`.您仍然停留在Text element，但是你将要进入到RichText element的mount方法中。

7. 点击 RenderObjectElement.mount. 你将看到什么？多么美丽的景象：`widget.createRenderObject(this)`.最后，这里是创建 `RenderParagraph` 的地方.参数`this` 是您当前所在的 `MultiChildRenderObjectElement`。

8. 点击 RichText.createRenderObject. 在这里，你在另一边。请注意，`MultiChildRenderObjectElement` 已重新命名为 `BuildContext`。

有人累了吗?既然你处在断点，为什么不休息一下，喝点水呢?仍然有很多伟大的事情要看和做。

### Stepping Down: Text Rendering Objects

您已经见过Flutter架构的图表，[如下](https://github.com/flutter/flutter/wiki/The-Engine-architecture)所示:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/flutter_overview.png)

上一节所做的工作是在Widgets层进行的。在本章节，你将逐步进入渲染、绘制和基础层。即使您正在深入研究，但在Flutter框架的较低级别上，事情实际上更简单，因为不需要处理多个树。您还处于添加的断点吗? Command-click *RenderParagraph* 查看里面的内容。

花几分钟上下滚动 `RenderParagraph` 类. 以下是一些需要注意的事情:

- `RenderParagraph` 继承自 `RenderBox`. 这意味着这个渲染对象的形状是矩形的，并且基于内容有一些固有的宽度和高度。对于render paragraph, 内容是text
- 它处理点击事件（hit testing）。嘿，孩子们，别打对方!如果你要点击什么，点击render Paragraph。它能承受。
- `performLayout` 和 `paint` 方法也很有趣。

你是否注意到`RenderParagraph`把它的文本绘制工作交给了一个叫做`TextPainter` 的东西? 在类的顶部附近找到`_textPainter`的定义。让我们离开渲染层<Rendering layer>，到下面的绘画层<Painting layer>。Command-click `TextPainter`。

花点时间看看风景。

* 有一个重要的成员变量，名为`_paragraph` ，类型为`ui.Paragraph`。ui部分是为来自dart:ui库(Flutter框架的最低级别)的类添加前缀的常用方法。
*  `layout` 方法非常有趣. 你不能直接实例化`paragraph`. 必须需要使用`ParagraphBuilder` 类去做。它采用适用于整个段落<paragraph>的默认段落样式。这可以通过包含在`TextSpan`树中的样式进一步修改。调用`TextSpan.build()`将这些样式添加到`ParagraphBuilder`对象中。
* 你可以看到`paint`方法在这里非常简单。`TextPainter`只是把paragraph交给<u>canvas.drawParagraph()</u>. 如果你Control-click它，你能看到它调用了<u>paragraph._paint</u>方法。



您已经来到Flutter框架的基础层。在TextPainter类中，Control-click以下两个类:

* `ParagraphBuilder` : 它添加了文本和push以及pop的样式，但实际工作交给了原生图层<native layer>。
* `Paragraph` : 这里没什么可看的。所有内容都被传递到原生层<native layer>。

现在停止正在运行的应用程序。

下面是一个图表来总结你上面看到的:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/text_render_low_level.png)

### Way Down: Flutter’s Text Engine

离开你的祖国去一个你不会说当地语言的地方可能会有点害怕。但也是刺激的。您将离开Dart之地，访问原生的文字引擎。他们使用C和C++。幸运的是，这里面有很多的英文标志。

你不能在你的IDE中再Command-click，但代码都在GitHub上，作为Flutter仓库<repository>的一部分。该文本引擎称为LibTxt。现在从这个[链接](https://github.com/flutter/engine/tree/master/third_party/txt)去那里。

我们不会在这里呆太久，但是如果你喜欢探索，稍后查看一下src文件夹。不过现在，让我们都转到原生类中。`Paragraph.dart`将它的工作传递到[txt/paragraph_txt.cc](https://github.com/flutter/engine/blob/master/third_party/txt/src/txt/paragraph_txt.cc).点击这个链接。

你可能喜欢在你的空闲时间检查`Layout`and `Paint` 方法，但现在只向下滚动一点，看看导入:

```cpp
#include "flutter/fml/logging.h"
#include "font_collection.h"
#include "font_skia.h"
#include "minikin/FontLanguageListCache.h"
#include "minikin/GraphemeBreak.h"
#include "minikin/HbFontCache.h"
#include "minikin/LayoutUtils.h"
#include "minikin/LineBreaker.h"
#include "minikin/MinikinFont.h"
#include "third_party/skia/include/core/SkCanvas.h"
#include "third_party/skia/include/core/SkFont.h"
#include "third_party/skia/include/core/SkFontMetrics.h"
#include "third_party/skia/include/core/SkMaskFilter.h"
#include "third_party/skia/include/core/SkPaint.h"
#include "third_party/skia/include/core/SkTextBlob.h"
#include "third_party/skia/include/core/SkTypeface.h"
#include "third_party/skia/include/effects/SkDashPathEffect.h"
#include "third_party/skia/include/effects/SkDiscretePathEffect.h"
#include "unicode/ubidi.h"
#include "unicode/utf16.h"
```

从这里，您可以(稍微挖掘一下)了解 `LibTxt` 是如何工作的。它基于许多其他库。这里有一些有趣的花边新闻:


