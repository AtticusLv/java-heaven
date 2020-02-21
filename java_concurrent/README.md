<!-- TOC -->

- [一、使用线程](#一使用线程)
    - [实现Runnable接口](#实现runnable接口)
    - [实现Callable接口](#实现callable接口)
- [](#)

<!-- /TOC -->
# 一、使用线程
使用线程方法：
- 实现Runnable接口
- 实现Callable接口
- 继承Thread类

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以理解为任务是通过线程驱动从而执行的。
## 实现Runnable接口
需要实现接口中的run()方法

## 实现Callable接口
与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。
```
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```
```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```
# 