# flutter 底层原理

## element 的生命周期
1.Framework调用Widget.createElement 创建Element实例，element
Element createElement();

2.FrameWork调用element.mount(parentElement,newSlot)，
void mount(Element? parent, Object? newSlot)
mount方法首先调用element所对应的widget的CreateRenderObject()方法
创建与element相关联的RenderObject对象，然后用element.attachRenderObject()
方法将element.renerObject添加到渲染树中插槽指定的位置。(这一步不是必须的，一般发
生在Element数结构发生变化时才需要重新attach)。插入到渲染树后的element就处于active状态
处于active状态就可以显示在屏幕上了(或者隐藏)

3.当有父widget的配置数据发生改变时。同时起state.build返回的Widget的结构与之前不同
此时就需要重新构建对应的Element树。为了进行Element复用，在Element重新构建之前会尝试
是够可以复用旧树相同位置的Element，element节点在更新前都会调用其对应Widget.canUPdate()
方法，如果返回true，则复用旧的Element，旧的Element会使用新的widgetde 数据更新，
如果返回false,就会创建一个新的element

canUpdate()主要判断newWidget 和 newWidget的 runtimeType和 key 是否相等，
如果相等就返回true,反之返回false
根据这个原理可以通过指定不同的key 来避免重复

4.当有祖先Element决定要移除elements时，这是该祖先Element就会调用
的activateChild方法来移除它
移除后element.rederObject也会从渲染树种移除，
然后FrameWork会调用Element.Deactive()方法
这时element的状态就变成inactive

5.inactive的element将不会再显示到屏幕上，为了避免再一次动画执行过程中反复创建，移除某个特定Element，
"inactive"态的Element在当前动画最后一帧结束前都会保留，如果动画执行结束后它还未能重新变成"active"状态，
Framework就会调用其unmount（）方法将其彻底移除，这时Element将永远不会再被插入到树中

6.如果element要重新插入到Element树的其他位置，
如Element或Element的祖先拥有一个GlobalKey（用于全局复用元素）
那么Framework会先将element从现有位置移除，然后再调用其activate方法
将其RenderObject重新attach到渲染树

## buildContext的本质
buildContext的本质是 widget对应的Element


## 不通过widget，直接用element 也可以实现UI界面
buildContext的本质是 widget对应的Element

## RenderObject
每一个element 都对应一个RenderObject
rederObject的主要职责是Layout和绘制，所有的RenderObject回组成一棵渲染树Render Tree.

作用：布局，绘制，层合成以及上屏
属性
parent
   指向渲染树中自己的父节点
parentData
   预留变量自空间的布局信息

renderObject 类本身实现了一套布局和绘制的协议，单是并没有定义子节点模型。也没有定义坐标系统和具体的布局协议


子节点模型和坐标系和布局协议 
RenderBox RenderSliver都继承自RenderObject,布局坐标次用笛卡尔坐标系，屏幕的（top,left）是原点

RenderBox 盒模型

RenderSliver 按需加载模型

## flutter 的启动过程

入口：
void runApp(Widget app) {
WidgetsFlutterBinding.ensureInitialized()
..scheduleAttachRootWidget(app)
..scheduleWarmUpFrame();
}

参数app是一个widget，他是flutter应用启动后的第一个组件。而
WidgetsFlutterBinding正是绑定widget框架和Flutter引擎的桥梁，
定义如下：
   
##WidgetsFlutterBinding
/// * [GestureBinding], which implements the basics of hit testing.
/// * [SchedulerBinding], which introduces the concepts of frames.
/// * [ServicesBinding], which provides access to the plugin subsystem.
/// * [PaintingBinding], which enables decoding images.
/// * [SemanticsBinding], which supports accessibility.
/// * [RendererBinding], which handles the render tree.
/// * [WidgetsBinding], which handles the widget tree.
class WidgetsFlutterBinding extends BindingBase with GestureBinding, SchedulerBinding, ServicesBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {
/// Returns an instance of the binding that implements
/// [WidgetsBinding]. If no binding has yet been initialized, the
/// [WidgetsFlutterBinding] class is used to create and initialize
/// one.
///
/// You only need to call this method if you need the binding to be
/// initialized before calling [runApp].
///
/// In the `flutter_test` framework, [testWidgets] initializes the
/// binding instance to a [TestWidgetsFlutterBinding], not a
/// [WidgetsFlutterBinding]. See
/// [TestWidgetsFlutterBinding.ensureInitialized].
static WidgetsBinding ensureInitialized() {
if (WidgetsBinding._instance == null)
WidgetsFlutterBinding();
return WidgetsBinding.instance;
}
}

##Window
window 正是Flutter Framework连接宿主操作的接口，我们看一下Window类的定义
class Window {

// 当前设备的DPI，即一个逻辑像素显示多少物理像素，数字越大，显示效果就越精细保真。
// DPI是设备屏幕的固件属性，如Nexus 6的屏幕DPI为3.5
double get devicePixelRatio => _devicePixelRatio;

// Flutter UI绘制区域的大小
Size get physicalSize => _physicalSize;

// 当前系统默认的语言Locale
Locale get locale;

// 当前系统字体缩放比例。  
double get textScaleFactor => _textScaleFactor;

// 当绘制区域大小改变回调
VoidCallback get onMetricsChanged => _onMetricsChanged;  
// Locale发生变化回调
VoidCallback get onLocaleChanged => _onLocaleChanged;
// 系统字体缩放变化回调
VoidCallback get onTextScaleFactorChanged => _onTextScaleFactorChanged;
// 绘制前回调，一般会受显示器的垂直同步信号VSync驱动，当屏幕刷新时就会被调用
FrameCallback get onBeginFrame => _onBeginFrame;
// 绘制回调  
VoidCallback get onDrawFrame => _onDrawFrame;
// 点击或指针事件回调
PointerDataPacketCallback get onPointerDataPacket => _onPointerDataPacket;
// 调度Frame，该方法执行后，onBeginFrame和onDrawFrame将紧接着会在合适时机被调用，
// 此方法会直接调用Flutter engine的Window_scheduleFrame方法
void scheduleFrame() native 'Window_scheduleFrame';
// 更新应用在GPU上的渲染,此方法会直接调用Flutter engine的Window_render方法
void render(Scene scene) native 'Window_render';

// 发送平台消息
void sendPlatformMessage(String name,
ByteData data,
PlatformMessageResponseCallback callback) ;
// 平台通道消息处理回调  
PlatformMessageCallback get onPlatformMessage => _onPlatformMessage;

... //其他属性及回调

}

可以看到Window类包含了当前设备和系统的一些信息以及Flutter Engine的一些回调。
现在我们再回来看看WidgetFlutterBinding混入的各种Binding.通过查看这些Binding
的源码，我们可以发现这些Binding中基本都是监听并处理Window对象的一些事件，
然后将这些事件按照Framework的模型包装，抽象然后分发。
可以看到WidgetsFlutterBinding正是粘连Flutter engine与上层Framework的"胶水"

/// * [GestureBinding], which implements the basics of hit testing.
提供了window.onPointerDataPacket回调，绑定Framework手势子系统，
是Framework事件模型和底层事件的绑定入口
/// * [SchedulerBinding], which introduces the concepts of frames.
提供了window.onBeginFrame和window.onDrawFrame回调，
监听刷新事件，绑定Framework绘制调度子系统
/// * [ServicesBinding], which provides access to the plugin subsystem.
提供了 window.onPlatformMessage回调，用于绑定平台消息通道(message channel)
主要处理原生和Flutter通信
/// * [PaintingBinding], which enables decoding images.
绑定绘制库，主要用于处理图片库
/// * [SemanticsBinding], which supports accessibility.
语义化层与Fluter engine的桥梁，找事辅助功能的底层支持
/// * [RendererBinding], which handles the render tree.
提供了window.onMetricsChanged
window.onTextScaleFactorChanged等回调。他是渲染树与fluter engine的桥梁
/// * [WidgetsBinding], which handles the widget tree.
提供了 window.onLocaleChanged
onBuildScheduled等回调。他是Flutter widget层与engine的桥梁

WidgetsFluttterBinding.ensureInitialed()负责初始化一个WidgetBinding的全局单例，
紧接着会调用WidgetBinding的attachRootWidget方法，
该方法负责将根Widget添加到RenderView上，

void attachRootWidget(Widget rootWidget) {
      final bool isBootstrapFrame = renderViewElement == null;
      _readyToProduceFrames = true;
      _renderViewElement = RenderObjectToWidgetAdapter<RenderBox>(
      container: renderView,
      debugShortDescription: '[root]',
      child: rootWidget,
      ).attachToRenderTree(buildOwner!, renderViewElement as RenderObjectToWidgetElement<RenderBox>?);
      
if (isBootstrapFrame) {
      SchedulerBinding.instance.ensureVisualUpdate();
      }
}
                
代码中有renderView和renderViewElement 两个变量，
renderView是一个RenderObject,他是渲染树的根，
而renderViewElement是RenderView对应的Element对象，
可见该方法主要完成了根widget到根RenderObject,再到跟Element
的整个关联过程。我们看看attachToRenderTree的源码实现：

RenderObjectToWidgetElement<T> attachToRenderTree(BuildOwner owner, [RenderObjectToWidgetElement<T> element]) {
  if (element == null) {
    owner.lockState(() {
      element = createElement();
      assert(element != null);
      element.assignOwner(owner);
    });
    owner.buildScope(element, () {
      element.mount(null, null);
    });
  } else {
    element._newWidget = this;
    element.markNeedsBuild();
  }
  return element;
}
该方法负责创建根Element，即RenderObjectToWidgetElement,并且将Element与widget
进行关联，即创建出widget树对应的element树。如果element已经创建过了，
则将根element中关联的widget设为新的，由此可以看出element只会创建一次，
后面会进行复用。
那么BuildOwner是什么？
其实它就是widget framework 的管理类，它跟踪哪些widget需要重新构建。

组件树在构建(build)完毕后，回到runApp的现实中，当调用完attachRootWidget后，
最后一行会调用WidgetsFlutterBinding实例的scheculeWarmUpFrame()方法，该方法的实现在SchedulerBinding中，
它被调用后立即进行一次绘制，在此绘制结束前，该方法会锁定事件分发，
也就是说在本次绘制结束之前，flutter将不会响应各种事件，这可以保证在绘制过程中不会再触发新的绘制

20230222 - 20230322
        
# Flutter计划

+ flutter 底层原理
    - 启动，绘制，渲染，更新  1周 
    - provider 原理        1周
    - getX 原理           1周
    - flutter 原生混合开发   1周
    - flutter 插件开发      1周
# go    
+ redis实战  11章 2周
+ go语言高级编程 7章 1周
  - Marker character change forces new list start:
    * Ac tristique libero volutpat at
    + Facilisis in pretium nisl aliquet
    - Nulla volutpat aliquam velit
# 操作系统
+ 汇编语言 
+ 操作系统公开课哈工大

# 计算机网络 
+ 哈工大的计算机网络课程
+ 计算机网络——自顶向下方法
+ 《图解 HTTP》
+ 《网络是怎样连接的》 
# 数据库   
+ 北京师范大学的《数据库系统原理》
  