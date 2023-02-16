# flutter 底层原理

## element 的生命周期
1.Framework调用Widget.createElement 创建Element实例，element

2.FrameWork调用element.mount(parentElement,newSlot)，
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
   指向渲染书中自己的父节点
parentData
   预留变量自空间的布局信息

renderObject 类本身实现了一套布局和绘制的协议，单是并没有定义子节点模型。也没有定义坐标系统和具体的布局协议


子节点模型和坐标系和布局协议 
RenderBox RenderSliver都继承自RenderObject,布局坐标次用笛卡尔坐标系，屏幕的（top,left）是原点

RenderBox 盒模型

RenderSliver 按需加载模型