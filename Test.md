#### 1. ����JavaFx���߳�
��javaFX�����У��ж���߳�

1. main
2. JavaFX-Launcher
3. JavaFX Application Thread

һ�����߳������Ԫ�ص��޸�

����JavaFX Application Thread����Ϊ�û��̣߳������ʼ��Scene Graph��������һЩ������޸ĺͲ�����Ҫ�ŵ����߳��н��У���������ڴ��߳��½��е�������������ܻᱨ����Not on FX application thread;


```
 void runToolkit() {
        Thread user = Thread.currentThread();

        if (!toolkitRunning.getAndSet(true)) {
            user.setName("JavaFX Application Thread");
```

��������ʹ�ö��߳�ʱ��������Լ�ʵ�ֵ��߳�����Ҫ�޸����Ԫ�أ���Ҫ���߳��е������·���ʵ��

```
Platform.runLater(() -> {
                    // �������
});
```
�������߳��к�̨�̵߳�ʵ��

1��Ϊʲôʹ�ú�̨�߳�

����һЩ��Ҫ�ϳ�ʱ�������ҵ���߼�����ŵ��û������н��еĻ���ǰ����ͼ����ִ��ҵ���߼��Ĺ�����ʧȥ��Ӧ��Ӱ�쵽�û����飬��˶�����Ҫ�ϳ�ʱ��Ĳ���ͨ���ŵ���̨�����н��С�

2����̨�̵߳�ʵ�ַ�ʽ

    a��task��
```
Task task = new Task() {
            @Override
            protected Object call() throws Exception {
                // ҵ���߼�
                return null;
            }
        };
 
 Thread thread = new Thread(task);
 thread.setDaemon(true);
 thread.start();
        
```
