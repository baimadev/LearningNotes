## LocalContentColor
改变Icon alpha。

```kotlin
    Icon(
            imageVector = todo.icon.imageVector,
            tint = LocalContentColor.current.copy(alpha = iconAlpha),
            contentDescription = stringResource(id = todo.icon.contentDescription)
        )
```


## CompositionLocalProvider
改变子组件alpha。

```kotlin
  CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
        Text(
            text = text,
            modifier = modifier
        )
    }
```

## Span
给字体设置不同的颜色、大小。

```kotlin

val tagStyle = MaterialTheme.typography.overline.toSpanStyle().copy(
   background = MaterialTheme.colors.primary.copy(alpha = 0.1f)
)

     Text(text = buildAnnotatedString {
                append("N")
                withStyle(SpanStyle(color = Color.Yellow)){
                    append("E")
                }
                withStyle(tagStyle){
                    append("W")
                }
                withStyle(SpanStyle(color = Color.Blue)){
                    append("S")
                }

            })
```

## 资源获取

```kotlin
dimensionResource(id)
stringResource(id)
```
## 获取context

`LocalContext.current.resources`

## Compose与Android View互调

1.View中调用Compose
（1）、布局中添加ComposeView；
（2）、调用composeView.setContent()；

## Text
居中 textAlign = TextAlign.Center
