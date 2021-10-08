## animateColorAsState
颜色渐变

`val backgroundColor by animateColorAsState(color = Purple100) `

返回一个`State<Color>`变化时有动画渐变效果。

## AnimatedVisibility
动画显隐

```kotlin
    AnimatedVisibility(visible = extended) {
                    Text(
                        text = stringResource(R.string.edit),
                        modifier = Modifier
                            .padding(start = 8.dp, top = 3.dp)
                    )
                }
```

自定义动画参数

```kotlin
    AnimatedVisibility(
        visible = shown,
        enter = slideInVertically(
            initialOffsetY = {fullHeight -> -fullHeight },
            animationSpec = tween(durationMillis = 150,easing = LinearOutSlowInEasing)
        ),
        exit = slideOutVertically(
            targetOffsetY = {fullHeight -> - fullHeight },
            animationSpec = tween(durationMillis = 250,easing = FastOutLinearInEasing)
        )
    )
```

## animateContentSize

动画改变大小

```kotlin
 Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
                .animateContentSize()

        ) {
            Row {
                Icon(
                    imageVector = Icons.Default.Info,
                    contentDescription = null
                )
                Spacer(modifier = Modifier.width(16.dp))
                Text(
                    text = topic,
                    style = MaterialTheme.typography.body1
                )
            }
            if (expanded) {
                Spacer(modifier = Modifier.height(8.dp))
                Text(
                    text = stringResource(R.string.lorem_ipsum),
                    textAlign = TextAlign.Justify
                )
            }
        }
```

## InfiniteTransition
估值器？

```kotlin

val infiniteTransition = rememberInfiniteTransition()
val alpha by infiniteTransition.animateFloat(
    initialValue = 0f,
    targetValue = 1f,
    animationSpec = infiniteRepeatable(
        animation = keyframes {
            durationMillis = 1000 //关键帧持续时间
            0.7f at 500  //500帧的时候值为0.7f
        },
        repeatMode = RepeatMode.Reverse
    )
)

```
创建一个不断变化的值
