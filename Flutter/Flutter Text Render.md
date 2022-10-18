[TOC]

你是否曾骑着骆驼走过寂静的戈壁，或在游牧民族的帐篷里品茶?请点击下面的链接，享受这个千载难逢的机会。这是限时优惠，所以现在就行动，门票售完!

只是开玩笑。:]

这不是一个旅游网站，但我将带您踏上激动人心的旅程，到Flutter框架的一个奇异的角落:文本渲染<*text rendering*>。乍一看似乎很简单。只是ABC,对吗?然而，背后隐藏着难以言喻的复杂性。

在本教程结束时，您将:

* 看到widget、element和render objects之间的关系。
* 探索`Text`和`RichText`小部件<widgets>背后的内容。
* 制作自己的自定义文本小部件。

### Getting Started

通过点击页面顶部或底部的下载材料按钮来下载启动项目。这次我不是开玩笑说要点击链接。你认为你是通过看电视上的旅游节目，还是通过坐飞机去那里学到更多?最好的办法是下载初始项目，然后继续执行。



### A Journey Through the Framework

作为Flutter开发人员，您已经非常熟悉`stateless`和`stateful` widgets，但它们不是惟一的。今天您将了解第三种类型`RenderObjectWidget`，以及支持它的Flutter框架的底层类。

下图显示了`Widget`子类，其中蓝色的是我在这节课中最想关注的:

![](https://koenig-media.raywenderlich.com/uploads/2019/07/widget_types.png)



RenderObjectWidget是一个蓝图。 它包含RenderObject的配置信息，RenderObject完成了所有艰难的hit testing 和绘制UI的工作。

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

* [Minikin](https://github.com/flutter/engine/tree/master/third_party/txt/src/minikin) 负责测量和布局文本。
* [ICU](https://icu.unicode.org/home) 可以帮助Minikin完成一些事情，比如将文本分成几行。
* [HarfBuzz](https://www.freedesktop.org/wiki/Software/HarfBuzz/) 帮助Minikin从字体中选择正确的字形。
* [Skia](https://skia.org/) 将文字和文字装饰画在画布上。

你看得越多，你就越能意识到正确渲染文本的重要性。我甚至没有时间提及行间间距< [interline spacing](https://stackoverflow.com/questions/56799068/what-is-the-strutstyle-in-the-flutter-text-widget)>、grapheme集群<[grapheme clusters](https://stackoverflow.com/questions/54483177/handling-grapheme-clusters-in-dart)>和bidi< [bidi](https://en.wikipedia.org/wiki/Bidirectional_Text)>文本等问题。

您已经深入了解了框架和文本呈现引擎。现在是时候退一步，把这些知识派上用场了。

### Stepping Up Your Game: Building a Custom Text Widget

你现在要做一件你以前可能从未做过的事。将创建一个自定义text widget，但不是像通常那样通过合成，而是通过创建一个渲染对象<render object>，该对象使用可用的最低级别的Flutter绘制文本。

Flutter最初设计的目的不是为了允许开发人员进行自定义文本布局，但Flutter团队反应迅速，愿意做出更改。请密切关注[this GitHub issue](https://github.com/flutter/flutter/issues/35994) 的进展更新。

到目前为止，Steppe Up旅游应用程序看起来还不错，但如果它能支持蒙古语脚本就更好了。传统蒙古语是独特的。是写垂直的。标准的Flutter文本小部件支持水平布局，但我们需要垂直布局，其中行从右向左环绕。

![](https://koenig-media.raywenderlich.com/uploads/2019/08/horiz_vert.png)



#### 自定义渲染对象

为了更专注于底层的文字布局上，我已经在starter project中集成了widget、render object和一个帮助类。

![](https://koenig-media.raywenderlich.com/uploads/2019/08/included_classes.png)

让我简单解释一下我是怎么做的，以防将来你想制作一个不同的自定义渲染对象<render object>。

* **vertical_text.dart**: 这是`VerticalText` 小部件。我从RichText源代码开始制作它。我剥离了几乎所有内容，并将其更改为LeafRenderObjectWidget，它没有子对象。它创建了一个RenderVerticalText对象。
* **render_vertical_text.dart**: 我是通过剥离RenderParagraph并交换宽度和高度来实现这一点的。它使用 `VerticalTextPainter` 代替 `TextPainter`.
* **vertical_text_painter.dart**: 我从`TextPainter`开始并且去掉了我不需要的一切。我还交换了宽度和高度计算，并删除了对`TextSpan`树复杂样式的支持。
* **vertical_paragraph_constraints.dart**: 我使用高度而不是宽度作为约束条件。
* **vertical_paragraph_builder.dart**: 我从`ParagraphBuilder`入手，删除了我不需要的一切，添加默认样式并且使`build`方法返回`VerticalParagraph`而不是`Paragraph`
* **line_breaker.dart**: 这个类是Minikin [LineBreaker](https://github.com/flutter/engine/blob/a4abfb2333afe16675db0709d18a6e918b0123da/third_party/txt/src/minikin/LineBreaker.cpp)的替代物，Minikin LineBreaker不暴露在Dart。

在接下来的小节中，您将通过测量单词、将它们排成直线并将它们绘制到画布上来完成`VerticalParagraph class`的创建。

### Calculating and Measuring Text Runs

文本需要换行。为此，您需要在字符串中找到可以断行的适当位置。正如我前文所说，在撰写本文时，Flutter还没有公开Minikin/ICU `LineBreaker`类，但是一个可以接受的替代方法是在空格和单词之间中断。

这是应用程序的Unicode欢迎字符串:

```
ᠤᠷᠭᠡᠨ ᠠᠭᠤᠳᠠᠮ ᠲᠠᠯ᠎ᠠ ᠨᠤᠲᠤᠭ ᠲᠤ ᠮᠢᠨᠢ ᠬᠦᠷᠦᠯᠴᠡᠨ ᠢᠷᠡᠭᠡᠷᠡᠢ
```

以下是可能的断裂位置:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/runs.png)

我将把中断之间的子字符串称为文本的“运行”。你会用一个`TextRun`类来表示它，你现在会创建它。

在**lib/model**文件夹中，创建以**text_run.dart**命名的文件夹，并黏贴下面的代码:

```dart
import 'dart:ui' as ui show Paragraph;

class TextRun {
  TextRun(this.start, this.end, this.paragraph);

  // 1
  int start;
  int end;

  // 2
  ui.Paragraph paragraph;
}
```



注释：

1. 这些是文本run子字符串的索引，其中包含`start`包含，不包括`end`。
2. 您将为每次运行创建一个“段落<paragraph>”，以便获得其测量大小。

在**dartui/vertical_paragraph.dart** 文件中，将下面的代码添加打到**VerticalParagraph**类，记得引用`TextRun`:

```dart
// 1
List<TextRun> _runs = [];

void _addRun(int start, int end) {

  // 2
  final builder = ui.ParagraphBuilder(_paragraphStyle)
    ..pushStyle(_textStyle)
    ..addText(_text.substring(start, end));
  final paragraph = builder.build();

  // 3
  paragraph.layout(ui.ParagraphConstraints(width: double.infinity));

  final run = TextRun(start, end, paragraph);
  _runs.add(run);
}
```



值得一提的是:

1. 您将分别存储字符串中的每个单词。
2. 在构建段落<building the paragraph>之前添加样式和文本。
3. 在获得测量值之前，必须调用`layout`方法。我将`width`限制设置为无穷大<infinity>，以确保这次运行只有一行。

然后在**_calculateRuns** 方法中，添加如下代码：

```dart
// 1
if (_runs.isNotEmpty) {
  return;
}

// 2
final breaker = LineBreaker();
breaker.text = _text;
final int breakCount = breaker.computeBreaks();
final breaks = breaker.breaks;

// 3
int start = 0;
int end;
for (int i = 0; i < breakCount; i++) {
  end = breaks[i];
  _addRun(start, end);
  start = end;
}

// 4
end = _text.length;
if (start < end) {
  _addRun(start, end);
}
```

解释每个部分:

1. 如果已经执行过，则不需要重新计算运行。

2. 这是我在util文件夹中包含的简单的换行符类。`breaks`变量是可能出现换行符的索引位置的列表(数组)，在本例中是在空格和非空格字符之间。

3. 在每个停顿(break)之间从文本中创建一个运行。

4. 捕捉字符串的最后一个词。

测试到目前为止你已经做了什么。您还没有足够的内容在屏幕上显示任何内容，但是在`_layout`方法的末尾添加一个print语句。

```dart
print("There are ${_runs.length} runs.");
```

正常运行应用。您应该在Run控制台中看到以下打印输出:

```dart
There are 8 runs.
```

很好。这就是你所期望的:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/runs_8.png)

### Laying Out Runs in Lines

现在您需要看看每一行可以容纳多少这样的runs。假设这条线的最大长度可以和下图中的绿色条一样长:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/runs_line.png)



您可以看到，前三次运行将适合，但第四次需要在新的行上。

要以编程的方式做到这一点，您需要知道每个run的长度。值得庆幸的是，该信息存储在`TextRun`的`paragraph`属性中。

您将创建一个类来保存关于每一行的信息。在lib/model文件夹中，创建一个名为`line_info.dart`的文件。粘贴以下代码:

```dart
import 'dart:ui';

class LineInfo {
  LineInfo(this.textRunStart, this.textRunEnd, this.bounds);

  // 1
  int textRunStart;
  int textRunEnd;

  // 2
  Rect bounds;
}
```



对这些属性的注释:

1. 这些`indexes`告诉您这一行包含的`runs`的范围。
2. 这是这条线的像素大小。您可以使用[TextBox代替，它包含文本方向(从左到右或从右到左)。这个应用程序不使用bidi文本，所以一个简单的`Rect`就足够了。

回到 **dartui/vertical_paragraph.dart**， 在`VerticalParagraph`类中,添加以下代码，记住要导入`LineInfo`:

```dart
// 1
List<LineInfo> _lines = [];

// 2
void _addLine(int start, int end, double width, double height) {
  final bounds = Rect.fromLTRB(0, 0, width, height);
  final LineInfo lineInfo = LineInfo(start, end, bounds);
  _lines.add(lineInfo);
}
```

回顾这两个部分：

1. 这个列表的长度将是行数。
2. 在这一点上，你没有旋转任何东西，所以width和height指的是水平方向。

然后在 **_calculateLineBreaks** 方法中添加以下内容:

```dart
// 1
if (_runs.isEmpty) {
  return;
}

// 2
if (_lines.isNotEmpty) {
  _lines.clear();
}

// 3
int start = 0;
int end;
double lineWidth = 0;
double lineHeight = 0;
for (int i = 0; i < _runs.length; i++) {
  end = i;
  final run = _runs[i];

  // 4
  final runWidth = run.paragraph.maxIntrinsicWidth;
  final runHeight = run.paragraph.height;

  // 5
  if (lineWidth + runWidth > maxLineLength) {
    _addLine(start, end, lineWidth, lineHeight);
    start = end;
    lineWidth = runWidth;
    lineHeight = runHeight;
  } else {
    lineWidth += runWidth;

    // 6
    lineHeight = math.max(lineHeight, run.paragraph.height);
  }
}

// 7
end = _runs.length;
if (start < end) {
  _addLine(start, end, lineWidth, lineHeight);
}

```



按顺序解释不同部分:

1. 此方法必须在计算运行之后调用。
2. 用不同的约束重新布局这些行是可以的。
3. 遍历每一次运行，检查测量值。
4. `paragraph`也有一个`width`属性，但是它是约束宽度，不是测量宽度。由于传入`doble.infinity`作为约束，width是无穷大。调用`maxIntrinsicWidth`或者`longestLine`将会传给你run的测量宽度。更多信息请参见此[链接](https://stackoverflow.com/questions/57083632/what-is-the-meaning-flutters-width-metrics-for-the-paragraph-class)。
5. 求宽度之和。如果它超过了最大长度，那么开始一个新的行。
6. 目前，高度总是相同的，但在未来，如果你使用不同的风格为每一个`run`，采取最大将允许所有需要适配的。
7. 将最后的`runs`添加为最后一行。

通过在`_layout`方法末尾添加另一个`print`语句来测试到目前为止你所做的:

```dart
print("There are ${_lines.length} lines.");
```

进行热重启(或者如果需要重新启动应用程序)。您应该看到:

```dart
There are 3 lines.
```



这就是你所期望的，因为在**main.dart**中。这个`VerticalText widget`有300个逻辑像素的限制，这是绿色条的大约长度:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/runs_line_3.png)



### Setting the size

系统想要知道widget的size，但是您之前没有足够的信息。既然已经测量了线条，那么就可以计算size了。

在**VerticalParagraph**类中，添加下面的代码到**_calculateWidth**方法中：

```dart
double sum = 0;
for (LineInfo line in _lines) {
  sum += line.bounds.height;
}
_width = sum;
```

为什么我说height相加得到width? width是一个对外公开的值。外部用户想到的是旋转(垂直)线。另一方面，height变量是您在内部用于非旋转(水平)线的变量。

固有高度是指如果widget有足够的空间，它希望达到的高度。将以下代码添加到`_calculateintrinsic`方法中:

```dart
double sum = 0;
double maxRunWidth = 0;
for (TextRun run in _runs) {
  final width = run.paragraph.maxIntrinsicWidth;
  maxRunWidth = math.max(width, maxRunWidth);
  sum += width;
}

// 1
_minIntrinsicHeight = maxRunWidth;

// 2
_maxIntrinsicHeight = sum;
```

下面解释这些编号注释:

1. 如之前所说，由于旋转，height和width进行了交换。您不想任何单词被剪除，所以widget希望达到的最小高度是最长run的长度。
2. 如果这个widget把所有东西都放在一条长长的垂直线上，这就是它想要达到的高度。

将以下内容添加到`_layout`方法的末尾:

```dart
print("width=$width height=$height");
print("min=$minIntrinsicHeight max=$maxIntrinsicHeight");
```

重新启动APP，你应该会看到类似的内容:

```dart
width=123.0 height=300.0
min=126.1953125 max=722.234375
```

如果这条线是垂直的，那么最小和最大固有高度就是你所期望的:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/runs_line_min_max.png)

### Painting Text to the Canvas

几乎要完成了。现在所剩下的就是绘制<painting>runs.将下面的代码拷贝到**draw**方法中：

```dart
canvas.save();

// 1
canvas.translate(offset.dx, offset.dy);

// 2
canvas.rotate(math.pi / 2);

for (LineInfo line in _lines) {

  // 3
  canvas.translate(0, -line.bounds.height);

  // 4
  double dx = 0;
  for (int i = line.textRunStart; i < line.textRunEnd; i++) {

    // 5
    canvas.drawParagraph(_runs[i].paragraph, Offset(dx, 0));
    dx += _runs[i].paragraph.longestLine;
  }
}

canvas.restore();

```

按顺序解释各部分:

1. 移动到起始位置。
2. 将画布旋转90度。旧的顶部现在在右边。
3. 移到line开始的位置。`y`值是负的，所以它会向上移动每一行，也就是说，在旋转的画布上向右移动。
4. 每次画一个run(word)
5. 偏移量是直线上运行的起始位置。

下图显示了文本在三行中运行的顺序:

![](https://koenig-media.raywenderlich.com/uploads/2019/08/runs_line_painted.png)

再次运行应用程序。

Tadaa !漂亮的垂直脚本为我们的旅行应用程序增添了完美的色彩。

<img src="https://koenig-media.raywenderlich.com/uploads/2019/08/final_screenshot.png" style="zoom:50%;" />

### Where to Go From Here?



