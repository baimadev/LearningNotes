## Compose与ViewModel

```kotlin
@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel){
    val items : List<TodoItem> by todoViewModel.todoItems.observeAsState(listOf())
    TodoScreen(items = items, onAddItem = todoViewModel::addItem,onRemoveItem = todoViewModel::removeItem)
}

```
Compose函数观察viewmodel的值，事件发生时调用viewmodel方法更新数据，数据再推动UI更新。
像这样形式的Compose是无状态的(由外部传入状态)，stateless composables。

## 有状态的Composables

` val iconAlpha = remember(todo.id){ randomTint() }`

将id作为key存储对象的值，Compose下次重组（recomposition）时将返回上次的值。
remember存储值是存在对应的Composition中，当composeable被移除时，**remember中的值也会不存在**。

## mutableState
`val (text,setText) = remember { mutableStateOf("")} `
为Compose创建一个可观察的状态。其中setText = `(String) -> Unit`，当调用setText状态发生变化时，composable也会更新UI。

也可以向写成`val state = remember { mutableStateOf(default) }`。

## 状态提升
编写通用widget时应该将状态提出来，使其成为stateless compose

