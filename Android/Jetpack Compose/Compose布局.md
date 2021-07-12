
## Modifier

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