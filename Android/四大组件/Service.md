## 生命周期

![](../img/service.png)

- onStartCommond可以执行多次，onCreate、onBind只会执行一次。
- 启动且绑定一个Service，若在无解绑的前提下调用stopService是无法停止服务的。
- 多个客户端绑定到同一个服务，若都解除绑定，服务会自行销毁。
