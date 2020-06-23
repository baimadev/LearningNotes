

[IntentFlag启动模式](#1) 
<h3 id="1"></h3>

## IntentFlag启动模式

- Intent.FLAG\_ACTIVITY\_NEW_TASK  
使用一个新的Task来启动一个Activity。

- Intent.FLAG\_ACTIVITY\_CLEAR\_TASK  
将会使该activity所在的栈清空，它成为跟activity，该标志必须和Intent.FLAG\_ACTIVITY\_NEW_TASK一起使用。

- Intent.FLAG\_ACTIVITY\_SINGLE\_TOP  
同SingleTop,在栈顶则不创建，否则创建新的。

- Intent.FLAG\_ACTIVITY\_CLEAR_TOP  
同SingleTask,栈内若有实例，则让之上的都出栈，否则创建新的。

- Intent.FLAG\_ACTIVITY\_NO_HISTORY  
A以这种方式启动B,B启动C,则B就会从栈中移除，变为AC。