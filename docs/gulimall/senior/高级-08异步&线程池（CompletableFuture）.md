## 一、线程基础

### 1、初始化线程的4种方式

- 继承 Thread
- 实现 Runnable 接口
- 实现 Callable 接口 + FutureTask
- 线程池

#### 1.1 继承Thread

```java
class Thread_1 extends Thread{
    @Override
    public void run() {
        System.out.println("当前线程" + Thread.currentThread().getName());
    }
}
new Thread_1().start();
```



#### 1.2 实现 Runnable 接口

```java
class Run implements Runnable{
    @Override
    public void run() {
        System.out.println("当前线程" + Thread.currentThread().getName());
    }
}
new Thread(new Run()).start();
```



#### 1.3 实现 Callable 接口 + FutureTask

> 可以拿到返回结果，可以处理异常

```java
class Call implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println("当前线程" + Thread.currentThread().getName());
        return 2;
    }
}
FutureTask<Integer> futureTask = new FutureTask<>(new Call());
new Thread(futureTask).start();

Integer integer = futureTask.get();// 阻塞，抛出异常
```

***主进程可以获取线程的运算结果，但是不利于控制服务器中的线程资源。可能导致服务器资源耗尽。***



#### 1.4 线程池

> 业务代码中，之前三种启动线程方式都不会用。
>
> 而是【将所有的多线程异步任务交给线程池去执行】

***通过线程池控制资源，性能稳定，也可以获取执行结果，并捕获异常。***

***但是，在业务复杂情况下，一个异步调用可能会依赖于另一个异步调用的结果。***

```java
// 最好保证当前系统中只有一两个线程池
public static ExecutorService service = Executors.newFixedThreadPool(10);
// submit可以获取返回值
Future<Integer> submit = service.submit(new Call());
Integer in = submit.get();
// execute没有返回值
service.execute(new Run());
```

> 自定义线程池

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201113115450.png)

### 2、开发中为什么使用线程池

- 降低资源的消耗
  - 通过重复利用已经创建好的线程降低线程的创建和销毁带来的損耗。
- 提高响应速度
  - 因为线程池中的线程数没有超过线程池的最大上限时，有的线程处于等待分配任务的状态，当任务来时无需创建新的线程就能执行。
- 提高线程的可管理性
  - 线程池会根据当前系统特点对池内的线程进行优化处理，减少创建和销毁线程带来的系统开销。无限的创建和销毁线程不仅消耗系统资源，还降低系统的稳定性，使用线程池进行统一分配。





## 二、CompletableFuture 异步编排

### 1、创建异步对象

```java
public static ExecutorService executor = Executors.newFixedThreadPool(10);
// 1、runAsync没有返回值
CompletableFuture.runAsync(() -> {
	System.out.println("Runnable 的 run 方法");
}, executor);
// 2、supplyAsync可以获取返回值
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
	return "返回值";
}, executor);
completableFuture.get(); // 接收返回值，阻塞
```

### 2、计算完成时回调方法

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201113091047.png)

```java
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
	return "返回值"; // 可以返回值
}, executor).whenComplete((res, exception) -> {
    // 感知异常，但无法修改数据
    System.out.println("异步任务成功完成了");
    System.out.println("异步任务结果：" + res + ";异常为：" + exception);
}).exceptionally(throwable -> {
    // 感知异常，并且返回默认值
    return "默认返回值";
});
completableFuture.get();
```

> 方法不以 Async 结尾，意味着 Action 使用相同的线程执行，而 Async 可能会使用其他线程执行（如果是使用相同的线程池，也可能会被同一个线程选中执行）

### 3、handle

> 任务执行完的处理方法

```java
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
    return "返回值"; // 可以返回值
}, executor).handle((res,exception)->{
    if(res == null){
        return "";
    }
    return res + "01";
});
completableFuture.get();
```

### 4、线程串行化方法

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201113092344.png)

### 5、两任务组合-都要完成

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201113101536.png)

```java
// 定义两个异步对象
CompletableFuture<String> future01 = CompletableFuture.supplyAsync(() -> {
    return "返回值1";
}, executor);
CompletableFuture<String> future02 = CompletableFuture.supplyAsync(() -> {
    return "返回值2";
}, executor);
// 两任务组合
future01.runAfterBothAsync(future02,()->{
    System.out.println("run 线程03");
},executor);
future01.thenAcceptBothAsync(future02,(res1,res2)->{
    System.out.println("之前结果" + res1 + "》》" + res2);
},executor);
future01.thenCombineAsync(future02,(res1,res2)->{
    return res1 + res2;
},executor);
```

### 6、两任务组合-一个完成

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201113102949.png)

### 7、多任务组合

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201113103157.png)

> allOf 等待所有任务完成
>
> anyOf 只要有一个任务完成

```java
CompletableFuture<Void> all = CompletableFuture.allOf(future01, future02, future03);
all.get(); // 阻塞，不加这个的话主线程会继续走
// 分别获取各个线程的返回值
String s = future01.get();

CompletableFuture<Object> any = CompletableFuture.anyOf(future01, future02, future03);
Object o = any.get(); // 获取最先完成线程的返回值
```

