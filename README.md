# element 

## element 的生命周期

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