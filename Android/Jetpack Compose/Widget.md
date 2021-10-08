## Icon
`Icon(Icons.Outlined.Analytics,contentDescription = null,tint = Color.Black)`

## Card

```kotlin
   Card(
        modifier = modifier,
        border = BorderStroke(color = Color.Black, width = Dp.Hairline),
        shape = RoundedCornerShape(8.dp)
    ){


    }
```
## Spacer
空白Widget

## Surface

```kotlin
 Surface(elevation = 2.dp,color = MaterialTheme.colors.surface) {
            Text(
                text = text,
                modifier = modifier
            )
        }
```
设置子组件的背景颜色，elevation颜色高度。

## shape
设置组件形状

```kotlin
       modifier = Modifier.clip(shape = MaterialTheme.shapes.large)
```
