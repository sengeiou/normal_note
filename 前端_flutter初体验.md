> flutter 自学随记 

#### 重要角色

- Widget
树节点的描述信息. 
StatelessWidget: 只是一个视图.一旦build完成,就不会改变, 不接受事件和动作; 
StatefullWidget: 数据是动态的, 会变化的. 

- context 
  
简单的可以理解成一棵Widget的配置数,从这棵树上,可以向上或向下找到对应类型的widget 
**每一个widget 都单独对应一个 context**

- State 
定义StatefullWidget 的"行为",包含behavior 和 layout, state 的变化会引发widget的 rebuild 
state 和 context 存在关联关系, 即使context 改变了,起关联关系也不会改变. 这种关联关系也叫做挂载(mounted )


- Element 

负责维护树节点的增删改查, 
google的定义: 特定位置的Widget实例. 如果key 和 runtimeType 相同替换. 否则创建新的element实例插入树中 


- RenderObject 

负责渲染,布局,位置调整

#### statfullWidget 的生命周期

- initState 

框架最初开始调用, 此时state不能访问context 因为还没有mounted, 可用于初始化一些固定数据. 控制器和动画 

- didChangeDependencies 
  
  如果widget 关联夜歌 inheritWidget, 每次rebuild之前都会调用这个方法. 掉用这个方法时,context已经初始化成功.
  可以在该方法内初始化一些与context相关的listener, 覆盖该方法时,应先调用父类方法

- build 
  每次state发生改变,build 方法都会被调用.或者InheritedWidget 需要通知注册的widget 更新时都会被调用. 
  setState 强制执行 build 方法 

- dispose 
  移除listener 等一些clear 工作


**注意** 

StatefullWidget 的状态分为两个部分. 一部分是构造体传入的,这部分应该是不变的. 

#### 常用组件
- ListView 

1. itemExtent: 指定列表项高度, 在总高度确定的情况下指定有利于提高性能
2. shrinkwrap: 列表项是否收缩, 再外部容器高度是无限的情况,下必须要指定为true
3. addAutomaticKeepAlives: 列表项是否自动添加状态保持, 如果是的话,列表项滑出viewport 外也会自动保存状态

- GridView

二维表格

1. SliverGridDelegate: 用于指定布局策略

1.1 SliverGridDelegateWithFixedCrossAxisCount:  横轴固定个数(crossAxisCount)并指定长宽比(childAspectRatio )


- 布局

最终的ui布局对象未 renderObject, widget 可以理解为只是 renderObject 的配置信息. 

1. row / column 布局. 
2. wrap 流式布局, 超出可以折叠, 内部组件需指定固定尺寸. 
3. stack/positioned 配合使用, positioned 指定相对于stack上下左右4个角的偏移位置. 另stack 布局是堆叠的, 位于后面的positioned 组件会覆盖前面的组件
4. flex 弹性布局: 设置划分比例. 剩余区域按照flex 设置的比例来划分; row / column 就是确定划分方向的flex布局
flex 配合 expanded 设置比例. 每个比例部分用Expanded 包装

- 填充

1. padding: 填充
2. ConstrainedBox、SizedBox、UnconstrainedBox、AspectRatio 尺寸限制

- TabBar  &  TabBarView  

一般两者配合使用, TabBar 定义 tab 项,  TabBarView 定义每个tab 项下面对应的view 


- InheritedWidget 
  常见使用套路: 将需要共享通知的数据状态(state)放在 InheritedWidget 中. 子类再构建的时候通过 context.dependOnInheritedWidgetOfExactType 
  获取共享状态,同时注册自己到InheritedWidget 的通知列表中. 操作状态引起 状态重新构建InheritedWidget, 构建InheritedWidget 的时候
  会回调updateShouldNotify, 如果是true 的话, 按照InheritedWidget 的语义, 注册的子widget 会重新执行build 方法. 从而完成部分更新. 



#### 常见问题解决

- 屏幕适配

思路: 根据context获取媒体查询,以此获取针对于当前屏幕尺寸的合适像素

物理像素: 出厂即决定,不可改变,通常所说的像素指的就是物理像素

独立像素: 因为不同手机的物理像素是不一样的,为编写程序方便,需要定义出一个逻辑单位, 如 iPhone 3GS 和 iPhone 4/4s 都是 320 个虚拟像素(独立设备像素)

设备像素比: 等于 物理像素/独立像素 , 代表一个独立像素代表几个物理像素 

- 状态管理

1. InheritedWidget: 共享状态放在该widget, child 属性如果引用了 InheritedWidget 的状态, 当InheritedWidget的状态更改时, child 组件会被rebuild (didChangeDependencies 方法会被调用), child 组件通过
context.getElementForInheritedWidgetOfExactType 获取共享状态组件 (继承于InheritedWidget 的类)


- 异步加载问题

针对于有状态的widget, initstate 初始化future, build 依赖于 futureBuilder 来构建widget 

- hot reload 不生效

hot reload 并不会重新按照 state 的生命周期方法调用. 需要 hot restart 

#### 采坑记录

1. ListView 除滚动方向外,另一个方向默认会填充父容器,如报 unlimited, 需要指定父容器尺寸 
2. ConstrainedBox 转 sliver 需要用SliverToBoxAdapter适配. 
3. 使用 InheritedWidget 时, 遇到可重用的widget 时, 用 statelessWidget 替代方法生成widget, refer: https://stackoverflow.com/questions/59404622/dependoninheritedwidgetofexacttype-returns-null

