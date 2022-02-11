# WorkManager

## What is workmanager

处理后台任务，保证任务一定会执行。无论app处于什么状态，退出到后台、app被杀掉、重新打开······该任务都会被执行。  

- 该任务应是可延时执行的，不一定要马上执行。
- 不适用于立即执行或需要在准确时间执行的任务。（应该使用AlarmManager）

## why use workmanager
