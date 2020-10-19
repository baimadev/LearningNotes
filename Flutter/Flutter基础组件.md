#Flutter基础组件

[跳转目录](#1) 
<h3 id="1"></h3>

## 布局类组件
### Row

- textDirction 水平方向子组件的布局顺序（默认从左到右）。
- mainAxisSize 主轴方向占用的空间，默认MainAxisSize.Max，表示尽可能多的占用水平方向的空间。纵轴所占用的空间为最大子组件的高度。
- mainAxisAlignment 主轴的对齐方式。  
...

### Column

...

### Flex弹性布局
允许子组件按照一定的比例分配父容器空间。Row和Column都继承自Flex.

- direction 弹性布局的方向（Axis.horizontal）

#### Expanded
可以按比例扩伸Row、Column、Flex的子组件所占用的空间。

#### Spacer
Expanded的包装类，占用指定比例的空间，不显示内容。

- flex 弹性系数

```dart
Flex(
	direction:Axis.horizontal,
	children:<Widget>[
		Expanded(
			flex:1,
			child:...
		),
		Expanded(
			flex:2,
			child:...
		)
	],
)

```
横向按1：2分配空间。

### Wrap
流式布局，超出范围会折行显示。

- spacing:主轴方向的间距
- runSpacing:纵轴方向的间距
- direction
- alginment

### Flow
较复杂，需要自己实现Widget的位置转换

### Stack
层叠布局，允许子组件堆叠。（类似Frame）

- alginment:用于没有定位或部分定位（只在一个轴上有定位）的子组件。
- fit：用于没有定位的子组件如何适应Stack的大小。
- overflow:决定如何显示超出Stack显示空间的子组件。

#### Positioned
用于根据Stack四个角确定子组件的位置。

- l、t、r、b、w、h
只能指定l、r、w中的两个

```dart
Stack(
	alignment:Alignment.center,
	childrem:<Widget>[
		Positioned(
			left:18.0,
			right:20.0,
			child:...
		),
		Positioned(
			right:20.0,
			child:...
		),
		Container(
			child:...
		)
	],
)

```

### Align
调整子组件在父元素中的位置。

 - alignment:子组件在父组件中的起始位置（常用Alignment和FractionalOffset）
 - widthFactor和heightFactor:确定Align组件本身的宽高，它们会分别乘上子组件的宽高，如果值为null，则尽可能多的占用空间。

```dart
Algin(
	widthFactor:2.0,
	heightFactor:2.0,
	alignment:Alignment.topRight,
	child:FlutterLogo(
		size:60.0
	)
)

```
Alignment.topRight以矩形中心点为原点，值为（1，-1）

#### FractionalOffset
以矩形左侧顶点为原点（同Android）。  
`alignment:FractionalOffset(0.2,0.6)`  
实际偏移=（FractionalOffset.x\*childWidth,FractionalOffset.y*childHeight）

### Center
继承自Align，对齐方式是Alignment.center。widthFactor和heightFactor如果值为null，则尽可能多的占用空间。


## 容器类组件
### Padding
EdgeInsets
  
- fromLTRB
- all
- only
- symmetric

### ConstrainedBox
用于对子组件添加额外的约束。  
constraints:BoxConstraints() 可设置最小、最大宽高。  
BoxConstraints.tight(Size size) 生成给定大小  
BoxConstraints.expand()	  尽可能大填充另一个容器

#### 多重限制
如果某个组件有多个父级ConstrainedBox限制，对于minValue，生效的是值最大的那一个；对于maxValue，生效的是值最小的那一个。

### SizedBox
用于给子元素指定固定的宽高。
实际上内部实现是BoxConstraints.tightFor(w,h)。

### UnconstrainedBox
去掉多重限制。

### DecoratedBox
绘制装饰内容，如背景、边框、渐变等。  
decoration：通常使用BoxDecoration。

### Transform
应用矩阵变换来实现特效。  
Transform.translate(offset,child) //平移  
Transform.rotate(angle,child) //旋转  
Transform.scale(scale,child) //缩放  
组件所占用的空间不会变，依然在原来的位置。

### RotatedBox
对子组件进行变换，会影响子组件的大小和位置。

### Container
组合类容器，它是DecoratedBox、ConstrainedBox、Transform、Padding、Algin等组件组合的一个多功能容器。

### AppBar
leading：导航栏最左侧的widget  
actions：导航栏最右侧菜单  
bottom：导航栏底部菜单  
centerTitle：标题是否居中

#### TabBar
AppBar导航栏底部Tab。  
`TabController _controller = TabController(length,vsync)`

```dart
 appBar:AppBar(
 	bottom:TabBar(
 		controller:_controller,
 		tabs:<Widget>[]
 	)
 )
```
tabs属性接收一个Widget数组，也可以使用Tab组件（text,icon,child）。

#### TabBarView
实现Tab页面，与导航栏tab按钮联动。

```dart
	Scaffold(
		appBar:AppBar(
			bottom:TabBar(
				controller:_controller,
				tabs:...
			),
		),
		body:TabBarView(
			controller:_controller,
			children:<Widget>[],
		),
	)
```

### BottomNavigationBar
Scaffold的底部导航栏。

```dart
      BottomNavigationBar(
        items: <BottomNavigationBarItem>[
          BottomNavigationBarItem(icon: Icon(Icons.home),title: Text("Home")),
          BottomNavigationBarItem(icon: Icon(Icons.business),title: Text("Business")),
          BottomNavigationBarItem(icon: Icon(Icons.school),title: Text("school")),
        ],
        currentIndex: _selectedIndex,
        fixedColor: Colors.blue,
        onTap: _onItemTapped,
      ),

```
打洞效果

```dart
      bottomNavigationBar:BottomAppBar(
        color: Colors.white,
        shape: CircularNotchedRectangle(),//底部导航栏打出圆形的洞
        child: Row(
          children: <Widget>[
            IconButton(icon: Icon(Icons.map)),
            SizedBox(),//中间位置空出
            IconButton(icon: Icon(Icons.dashboard)),
          ],
          mainAxisAlignment: MainAxisAlignment.spaceAround,//均分底部导航栏横向空间
        ),
      ),

```
还需要在Scaffold下指定floatingActionButton的位置：
` floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked`

### Clip
对组件进行裁剪。  

- ClipOval//圆形
- ClipRRct//圆角矩形
- ClipRect//剪裁子组件实际占用的矩形大小，将溢出部分剪掉
- CustomClipper//自定义裁剪形状

## 可滚动组件

### SingleChildScrollView
只能接收一个子组件，未采用Sliver模型。item较少的时候使用。

### ListView

- itemExtent:指定子组件的高度，如果没有指定，系统在每次构建子组件时都重新计算一次，尤其在滚动位置频繁变化的时候。
- addAutomaticKeepAlives:为true，则滑出屏幕的item不会被GC，会使用KeepAliveNotification保存状态。
- addRepaintBoundary:为true，则可以避免item重绘。

使用ListView默认构造参数接受一个Widget数组时，不会使用Slver模型。
 
 ```dart
 	   ListView.builder(
        itemCount: 5,
        itemBuilder: (context, index) {
          return Row();
        });
  });
 ```
 添加分割线。
 
 ```dart
   ListView.separated(
        itemBuilder: (context, index) {
          return Row();
        },
        separatorBuilder: (context, index) {
          return Divider(
            color: Colors.blue,
          );
        },
        itemCount: null);
 
 ```
 
 
### GridView

- gridDelegate 控制GridView子组件如何排列。

#### SliverGridDElegateWithFixedCrossAxisCount
横轴为固定数量子元素。

- crossAxisCount 横轴子元素数量
- mainAxisSpacing 主轴方向的间距
- crossAxisSpacing
- childAspectRatio 宽高比

可使用GridView.count达到相同效果。
 
#### SliverGridDElegateWithMaxCrossExtent
- maxcrossAxisExtent 指定横轴子元素的最大值

可使用GridView.extent达到相同效果。

#### GridView.builder

```dart
    GridView.builder(
        gridDelegate:
            SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 5),
        itemBuilder: (context, index) {
          return Row();
        });

```

### CustomScrollView
CustomScrollView相当于一个胶水，将多个Sliver粘在一起，它子组件必须是Sliver。

```dart
 Widget customScrollWidget = Material(
      child: CustomScrollView(
        slivers: <Widget>[
          SliverAppBar(
            pinned: true,
            expandedHeight: 250.0,
            flexibleSpace: FlexibleSpaceBar(
              title: Text("DEMO"),
              background: Image.asset("./images/back.png",fit: BoxFit.cover),
            ),
          ),
          SliverPadding(
            padding: const EdgeInsets.all(16.0),
            sliver: SliverGrid(
              gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3,
                crossAxisSpacing: 5.0,
                mainAxisSpacing: 10.0,
                childAspectRatio: 4
              ),
              delegate: SliverChildBuilderDelegate((context,index){
                return Container(
                  alignment: Alignment.center,
                  child: Text("item$index"),
                  color: Colors.cyan[100*(index%9)],
                );
              },
                childCount: 20,
              ),
            ),
          ),
          SliverFixedExtentList(
            itemExtent: 50.0,
            delegate: SliverChildBuilderDelegate((context,index){
              return Container(
                alignment: Alignment.center,
                color: Colors.blue[100*(index%9)],
                child: Text("item$index"),
              );
            },
            childCount:50
            ),
          )
        ],
      ),
    );

```

### 滚动监听及控制

#### ScrollController

- offset 当前的滚动位置（像素）
- jumpTo()、animatedTo()
- addListener()监听滚动

#### 滚动位置恢复
PageStorage是一个用于保存页面（路由）相关数据的组件，子树中的widget可以通过指定不同的PageStorageKey来存储各自的数据状态。  
`ListView(key:PageStroageKey(1)...);`

#### ScrollerPosition
用来保存滚动组件的滚动位置。

#### 滚动监听
NotificationListener在从可滚动组件到widget树根之间的任意都能监听。

```dart
Widget fragment2 = NotificationListener<ScrollNotification>(
      onNotification: (notification){
        setState(() {
          _progress = "${(notification.metrics.pixels/notification.metrics.maxScrollExtent*100).toInt()}%";
        });
      },
      child:Stack(
        alignment: Alignment.center,
        children: <Widget>[
          ListView.separated(itemBuilder:(BuildContext context,int index){
            return ListTile(
              title: Text("$index "),
            );
          },
            controller: _scrollController,
            itemCount: 100,
            key: _pageStorageKey,
            separatorBuilder: (BuildContext context,int index){
              return index%2==0?Divider(color: Colors.black,):Divider(color:Colors.blue);
            },
          ) ,
          CircleAvatar(
            radius: 30.0,
            child: Text(_progress),
          )
        ],
      )

    );

```
在接收到滚动组件时，参数为ScrollNotification，它包含一个metrics属性，类型是ScrollMetrics，包含当前ViewPort及滚动位置等信息。

- pixel当前滚动位置
- maxScrollExtent 最大可滚动长度
- extentBefor 滑出ViewPort顶部的长度
- extentInside ViewPort内部的长度
- extentAfter 列表中未滑入ViewPort部分的长度
- atEdge 是否到边界

## 功能型组件

### willPopScope
导航返回拦截.

```dart
WillPopScope(
        onWillPop: ()async{
          if(_lastTime==null|| DateTime.now().difference(_lastTime)>Duration(seconds: 2)){
            _lastTime = DateTime.now();
            return false;
          }
          return true;
        },

        child:...
        )
```
onWillPop是一个回调函数,当用户点击返回按钮式被调用（包括导航返回按钮和Android物理返回按钮）。该回调函数需要返回一个Future对象，true则出栈，false不出栈。

### InheriedWidget
提供了数据在Widget中从上到下的传递。 如果子Widget使用了父Widget中的InheritedWidget的数据，则代表子Widget依赖于InheritedWidget。子Widget会在“依赖”发生变化时，回调didChangeDependencies方法。

#### 子Widget和InheritedWidget建立依赖
`context.inheritFromWithWidgetOfExactType()`

==此方法不会建立依赖关系。==  
`context.ancestorInheritedElementForWidgetOfExactType()`  

InheritedWidget每次更新则会调用`updateShouldNotify`方法，该方法如果返回true，则会==通知与他有依赖关系的子Widget==，调用子widget的`didChangeDependencies`和`build`方法。

InheritedWidget==更新的方法==则是重新构建它，及使用`setState()`。


