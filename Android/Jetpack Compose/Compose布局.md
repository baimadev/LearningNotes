### modifier的顺序很重要

```kotlin

  Row(modifier
        .padding(16.dp)
        .clickable(onClick = { /* Ignoring onClick */ })
    ) {
        ...
    }

```
先写padding，并不是整个区域都可点击。

### Align

```kotlin
 modifier = Modifier.align(Alignment.CenterVertically)
```

### then
`Modifier.then`可组合另一个modifier。

### Intrinsics

```kotlin

          Row (modifier = Modifier.height(IntrinsicSize.Min)){
                Text(
                    text = "text1",
                    modifier = Modifier
                        .weight(1f)
                        .fillMaxHeight()
                        .padding(start = 4.dp)
                        .wrapContentWidth(Alignment.CenterHorizontally)
                )

                Divider(color = Color.Black, modifier = Modifier
                    .fillMaxHeight()
                    .width(1.dp))


                Text(
                    modifier = Modifier
                        .weight(1f)
                        .padding(end = 4.dp)
                        .height(100.dp)
                        .wrapContentWidth(Alignment.CenterHorizontally),
                    text = "text2"
                )
            }

```
查询子view，强制高度为最小那个,text1和divider的高度将是100dp。

### layout

(1).对单个子view的限制

```kotlin
fun Modifier.firstBaselineToTop(distance:Dp) = this.then(
    layout{ measurable, constraints ->
        //只能测量子view一次
        val placeable = measurable.measure(constraints = constraints)

        //检查是否有基线
        check(placeable[FirstBaseline] != AlignmentLine.Unspecified)
        val firstBaseLine = placeable[FirstBaseline]

        val placeableY = distance.roundToPx() - firstBaseLine
        val height = placeable.height + placeableY
        //重新放置子view的位置
        layout(placeable.width,height = height) {
            placeable.placeRelative(0,placeableY)
        }
    }
)

```

(2).自定义布局

```kotlin
@Composable
fun MyOwnColumn(modifier: Modifier = Modifier, content: @Composable () -> Unit) {
    Layout(modifier = modifier, content = content) { measurables, constrants ->
        val placeables = measurables.map {
            it.measure(constraints = constrants)
        }

        var yPos = 0

        layout(constrants.maxWidth, constrants.maxHeight) {
            placeables.forEach {
                it.placeRelative(x = 0, y = yPos)
                yPos += it.height
            }
        }
    }
}

```
网格布局

```kotlin
@Composable
fun StaggeredGrid(modifier: Modifier = Modifier,rows:Int = 3,content: @Composable () -> Unit){
    Layout(content = content, modifier = modifier ){ measurables,constrant ->
        val rowWidths = IntArray(rows){0}
        val rowHeights = IntArray(rows){0}

        val placeables = measurables.mapIndexed { index, measurable ->
            val placeable = measurable.measure(constraints = constrant)
            val row = index % rows
            rowWidths[row] += placeable.width
            rowHeights[row] = max(rowHeights[row],placeable.height)
            placeable
        }

        val width = rowWidths.maxOrNull()?.coerceIn(constrant.minWidth.rangeTo(constrant.maxWidth))?:constrant.minWidth
        val height = rowHeights.sumOf { it }.coerceIn(constrant.minHeight.rangeTo(constrant.maxHeight))
        val rowY = IntArray(rows){0}
        for (i in 1 until rows){
            rowY[i] = rowY[i-1] + rowHeights[i-1]
        }


        layout(width,height){
            val rowX = IntArray(rows){0}
            placeables.forEachIndexed{ index, placeable ->
                val row = index%rows
                placeable.placeRelative(x=rowX[row],y=rowY[row])
                rowX[row] += placeable.width
            }
        }
    }
}

```

### constraintLayout

(1)、创建索引
`val (button,text,image,icon,ii) = createRefs()`

(2)、关联引用

```kotlin
Modifier.constrainAs(button){ top.linkTo(parent.top,margin = 16.dp)}

Modifier.constrainAs(text) { top.linkTo(button.bottom,16.dp)  }
```

(3)、居中

```kotlin
Modifier.constrainAs(button){
top.linkTo(parent.top,margin = 16.dp)
centerHorizontallyTo(parent)
}
```

(4)、创建barrier和GuidLine

```kotlin
//创建一个text和icon中最右侧的参考位置
val barrier = createEndBarrier(text,icon)

//创建一个平分基准线
val guideline = createGuidelineFromStart(fraction = 0.5f)
```

(5)、限置宽高

```kotlin
   Text(" LONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONGLONG",
                modifier = Modifier.constrainAs(text){
                    linkTo(start = guideline,end = parent.end)
                    width = Dimension.fillToConstraints
                })
```

- preferredWrapContent 被constraints限制

- wrapContent 无视constraints

- fillToConstraints 充满constraints中的宽\高

- preferredValue 固定值，但是不会超过constraints

- value 固定值，可以超过constraints

`width = Dimension.preferredWrapContent.atLeast(100.dp)`

(6)、抽象解耦

```kotlin
fun decoupled(margin:Dp):ConstraintSet{
    return ConstraintSet {
        val button = createRefFor("button")
        val text = createRefFor("text")

        constrain(button){
            top.linkTo(parent.top,margin)
        }

        constrain(text){
            top.linkTo(button.bottom,margin)
        }
    }
}

ConstraintLayout(decoupled(16.dp)) {
                    Button(onClick = { /*TODO*/ },modifier = Modifier.layoutId("button")) {
                        Text(text = "bt3")
                    }
                    Text(text = "tv3",modifier = Modifier.layoutId("text"))
                }
```