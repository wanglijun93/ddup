Android 的消息机制，也就是 Handler 机制，相信各位都已经是烂熟于心了吧。即创建一个 Message 对象，然后借助 Handler 发送出去，之后在 Handler 的 handleMessage() 方法中获取刚才发送的 Message 对象，然后在这里进行 UI 操作就不会出现崩溃了。

我们都知道子线程中进行 UI 操作会阻塞主线程，通常是怎么在子线程更新 UI的？

Handler
Activity.runOnUiThread()
View.post(Runnable r)

讲讲 Handler 机制吧
Handler 主要由以下部分组成。
Handler
Handler 是一个消息辅助类，主要负责向消息池发送各种消息事件Handler.sendMessage() 和处理相应的消息事件Handler.handleMessage()。

Message
Message 即消息，它能容纳任意数据，相当于一个信息载体。

MessageQueue
MessageQueue 如其名，消息队列。它按时序将消息插入队列，最小的时间戳将被优先处理。

Looper
Looper 负责从消息队列读取消息，然后分发给对应的 Handler 进行处理。它是一个死循环，不断地调用 MessageQueue.next() 去读取消息，在没有消息分发的时候会变成阻塞状态，在有消息可用时继续轮询。

在 Android 开发中使用 Handler 有什么需要注意的

首先自然是在工作线程中创建自己的消息队列必须要调用 Looper.prepare()，并且在一个线程中只能调用一次。当然，仅仅创建了 Looper 还不行，还必须使用 Looper.loop() 开启消息循环，要不然要 Looper 也没用。

我们平时在开发中不用调用是因为默认会调用主线程的 Looper。
此外，一个线程中只能有一个 Looper 对象和一个 MessageQueue 对象。

大概的标准写法是这样。

Looper.prepare();
Handler mHandler = new Handler() {   
   @Override
   public void handleMessage(Message msg) {
          Log.i(TAG, "在子线程中定义Handler，并接收到消息。。。");
   }
};
Looper.loop();
另外一个比较常考察的就是 Handler 可能引起的内存泄漏了。

Handler 可能引起的内存泄漏

我们经常会写这样的代码。

 private final Handler mHandler = new Handler(){        
		@Override
        public void handleMessage(Message msg) {            
		super.handleMessage(msg);
        }
    };
当你这样写的时候，你一定会收到编译器的黄色警告。

In Android, Handler classes should be static or leaks might occur, Messages enqueued on the application thread’s MessageQueue also retain their target Handler. If the Handler is an inner class, its outer class will be retained as well. To avoid leaking the outer class, declare the Handler as a static nested class with a WeakReference to its outer class

在 Java 中，非静态的内部类和匿名内部类都会隐式地持有其外部类的引用，而静态的内部类不会持有外部类的引用。

要解决这样的问题，我们在继承 Handler 的时候，要么是放在单独的类文件中，要么直接使用静态内部类。当需要在静态内部类中调用外部的 Activity 的时候，我们可以直接采用弱引用进行处理，所以我们大概修改后的代码如下。

 private static final class MyHandler extends Handler{        
 private final WeakReference<MainActivity> mWeakReference;        
 public MyHandler(MainActivity activity){
            mWeakReference = new WeakReference<>(activity);
        }        
		@Override
        public void handleMessage(Message msg) {            
		super.handleMessage(msg);
            MainActivity activity = mWeakReference.get();            
			if (activity != null){                
			// 开始写业务代码
            }
        }
    }
	private MyHandler mMyHandler = new MyHandler(this);
其实在我们实际开发中，不止一个地方可能会用到内部类，我们都需要在这样的情况下尽量使用静态内部类加弱引用的方式解决我们可能出现的内存泄漏问题。

用过 HandlerThread 吗？它是干嘛的？

HandlerThread 是 Android API 提供的一个便捷的类，使用它可以让我们快速地创建一个带有 Looper 的线程，有了 Looper 这个线程，我们又可以生成 Handler，本质上也就是一个建立了内部 Looper 的普通 Thread。

我们在上面提到的子线程建立 Handler 大概代码是这样。

new Thread(new Runnable() {    
@Override public void run() {        
// 准备一个 Looper，Looper 创建时对应的 MessageQueue 也会被创建
        Looper.prepare();        
		// 创建 Handler 并 post 一个 Message 到 MessageQueue
        new Handler().post(new Runnable() {            
		@Override 
            public void run() {
                  MLog.i("Handler in " + Thread.currentThread().getName());
            }
        });        
		// Looper 开始不断的从 MessageQueue 取出消息并再次交给 Handler 执行
        // 此时 Lopper 进入到一个无限循环中，后面的代码都不会被执行
        Looper.loop();
    }
}).start();
而采用 HandlerThread 可以直接把步骤简化为这样：

// 1. 创建 HandlerThread 并准备 LooperhandlerThread = new HandlerThread("myHandlerThread");
handlerThread.start();
// 2. 创建 Handler 并绑定 handlerThread 的 Loopernew Handler(handlerThread.getLooper()).post(new Runnable() {    @Override 
    public void run() {          
	// 注意：Handler 绑定了子线程的 Looper，这个方法也会运行在子线程，不可以更新 UI
          MLog.i("Handler in " + Thread.currentThread().getName());
    }
});
// 3. 退出@Override public void onDestroy() {    super.onDestroy();
    handlerThread.quit();
}
其中必须注意的是，workerThread.start() 是必须要执行的。

至于如何使用 HandlerThread 来执行任务，主要是调用 Handler 的 API。

使用 post 方法提交任务，postAtFrontOfQueue() 将任务加入到队列前端， postAtTime() 指定时间提交任务， postDelayed() 延后提交任务。
使用 sendMessage() 方法可以发送消息，sendMessageAtFrontOfQueue() 将该消息放入消息队列前端，sendMessageAtTime() 指定时间发送消息， sendMessageDelayed() 延后提交消息。
HandlerThread 的 quit() 和 quitSafety() 有啥区别？

两个方法作用都是结束 Looper 的运行。它们的区别是，quit() 方法会直接移除 MessageQueue 中的所有消息，然后终止 MesseageQueue，而 quitSafety() 会将 MessageQueue 中已有的消息处理完成后（不再接收新消息）再终止 MessageQueue。