#### 1. 概述JavaFx多线程
在javaFX程序中，有多个线程

1. main
2. JavaFX-Launcher
3. JavaFX Application Thread

一、多线程中组件元素的修改

其中JavaFX Application Thread被称为用户线程，负责初始化Scene Graph。界面上一些组件的修改和操作需要放到此线程中进行，如果不是在此线程下进行的组件操作，可能会报出：Not on FX application thread;


```
 void runToolkit() {
        Thread user = Thread.currentThread();

        if (!toolkitRunning.getAndSet(true)) {
            user.setName("JavaFX Application Thread");
```

当我们在使用多线程时，如果在自己实现的线程中需要修改组件元素，需要在线程中调用如下方法实现

```
Platform.runLater(() -> {
                    // 组件操作
});
```
二、多线程中后台线程的实现

1、为什么使用后台线程

对于一些需要较长时间操作的业务逻辑如果放到用户进程中进行的话，前端视图将在执行业务逻辑的过程中失去响应，影响到用户体验，因此对于需要较长时间的操作通常放到后台进程中进行。

2、后台线程的实现方式

    a、task类
```
Task task = new Task() {
            @Override
            protected Object call() throws Exception {
                // 业务逻辑
                return null;
            }
        };
 
 Thread thread = new Thread(task);
 thread.setDaemon(true);
 thread.start();
        
```
