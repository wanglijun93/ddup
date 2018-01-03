Android ����Ϣ���ƣ�Ҳ���� Handler ���ƣ����Ÿ�λ���Ѿ������������˰ɡ�������һ�� Message ����Ȼ����� Handler ���ͳ�ȥ��֮���� Handler �� handleMessage() �����л�ȡ�ղŷ��͵� Message ����Ȼ����������� UI �����Ͳ�����ֱ����ˡ�

���Ƕ�֪�����߳��н��� UI �������������̣߳�ͨ������ô�����̸߳��� UI�ģ�

Handler
Activity.runOnUiThread()
View.post(Runnable r)

���� Handler ���ư�
Handler ��Ҫ�����²�����ɡ�
Handler
Handler ��һ����Ϣ�����࣬��Ҫ��������Ϣ�ط��͸�����Ϣ�¼�Handler.sendMessage() �ʹ�����Ӧ����Ϣ�¼�Handler.handleMessage()��

Message
Message ����Ϣ�����������������ݣ��൱��һ����Ϣ���塣

MessageQueue
MessageQueue ����������Ϣ���С�����ʱ����Ϣ������У���С��ʱ����������ȴ���

Looper
Looper �������Ϣ���ж�ȡ��Ϣ��Ȼ��ַ�����Ӧ�� Handler ���д�������һ����ѭ�������ϵص��� MessageQueue.next() ȥ��ȡ��Ϣ����û����Ϣ�ַ���ʱ���������״̬��������Ϣ����ʱ������ѯ��

�� Android ������ʹ�� Handler ��ʲô��Ҫע���

������Ȼ���ڹ����߳��д����Լ�����Ϣ���б���Ҫ���� Looper.prepare()��������һ���߳���ֻ�ܵ���һ�Ρ���Ȼ������������ Looper �����У�������ʹ�� Looper.loop() ������Ϣѭ����Ҫ��ȻҪ Looper Ҳû�á�

����ƽʱ�ڿ����в��õ�������ΪĬ�ϻ�������̵߳� Looper��
���⣬һ���߳���ֻ����һ�� Looper �����һ�� MessageQueue ����

��ŵı�׼д����������

Looper.prepare();
Handler mHandler = new Handler() {   
   @Override
   public void handleMessage(Message msg) {
          Log.i(TAG, "�����߳��ж���Handler�������յ���Ϣ������");
   }
};
Looper.loop();
����һ���Ƚϳ�����ľ��� Handler ����������ڴ�й©�ˡ�

Handler ����������ڴ�й©

���Ǿ�����д�����Ĵ��롣

 private final Handler mHandler = new Handler(){        
		@Override
        public void handleMessage(Message msg) {            
		super.handleMessage(msg);
        }
    };
��������д��ʱ����һ�����յ��������Ļ�ɫ���档

In Android, Handler classes should be static or leaks might occur, Messages enqueued on the application thread��s MessageQueue also retain their target Handler. If the Handler is an inner class, its outer class will be retained as well. To avoid leaking the outer class, declare the Handler as a static nested class with a WeakReference to its outer class

�� Java �У��Ǿ�̬���ڲ���������ڲ��඼����ʽ�س������ⲿ������ã�����̬���ڲ��಻������ⲿ������á�

Ҫ������������⣬�����ڼ̳� Handler ��ʱ��Ҫô�Ƿ��ڵ��������ļ��У�Ҫôֱ��ʹ�þ�̬�ڲ��ࡣ����Ҫ�ھ�̬�ڲ����е����ⲿ�� Activity ��ʱ�����ǿ���ֱ�Ӳ��������ý��д����������Ǵ���޸ĺ�Ĵ������¡�

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
			// ��ʼдҵ�����
            }
        }
    }
	private MyHandler mMyHandler = new MyHandler(this);
��ʵ������ʵ�ʿ����У���ֹһ���ط����ܻ��õ��ڲ��࣬���Ƕ���Ҫ������������¾���ʹ�þ�̬�ڲ���������õķ�ʽ������ǿ��ܳ��ֵ��ڴ�й©���⡣

�ù� HandlerThread �����Ǹ���ģ�

HandlerThread �� Android API �ṩ��һ����ݵ��࣬ʹ�������������ǿ��ٵش���һ������ Looper ���̣߳����� Looper ����̣߳������ֿ������� Handler��������Ҳ����һ���������ڲ� Looper ����ͨ Thread��

�����������ᵽ�����߳̽��� Handler ��Ŵ�����������

new Thread(new Runnable() {    
@Override public void run() {        
// ׼��һ�� Looper��Looper ����ʱ��Ӧ�� MessageQueue Ҳ�ᱻ����
        Looper.prepare();        
		// ���� Handler �� post һ�� Message �� MessageQueue
        new Handler().post(new Runnable() {            
		@Override 
            public void run() {
                  MLog.i("Handler in " + Thread.currentThread().getName());
            }
        });        
		// Looper ��ʼ���ϵĴ� MessageQueue ȡ����Ϣ���ٴν��� Handler ִ��
        // ��ʱ Lopper ���뵽һ������ѭ���У�����Ĵ��붼���ᱻִ��
        Looper.loop();
    }
}).start();
������ HandlerThread ����ֱ�ӰѲ����Ϊ������

// 1. ���� HandlerThread ��׼�� LooperhandlerThread = new HandlerThread("myHandlerThread");
handlerThread.start();
// 2. ���� Handler ���� handlerThread �� Loopernew Handler(handlerThread.getLooper()).post(new Runnable() {    @Override 
    public void run() {          
	// ע�⣺Handler �������̵߳� Looper���������Ҳ�����������̣߳������Ը��� UI
          MLog.i("Handler in " + Thread.currentThread().getName());
    }
});
// 3. �˳�@Override public void onDestroy() {    super.onDestroy();
    handlerThread.quit();
}
���б���ע����ǣ�workerThread.start() �Ǳ���Ҫִ�еġ�

�������ʹ�� HandlerThread ��ִ��������Ҫ�ǵ��� Handler �� API��

ʹ�� post �����ύ����postAtFrontOfQueue() ��������뵽����ǰ�ˣ� postAtTime() ָ��ʱ���ύ���� postDelayed() �Ӻ��ύ����
ʹ�� sendMessage() �������Է�����Ϣ��sendMessageAtFrontOfQueue() ������Ϣ������Ϣ����ǰ�ˣ�sendMessageAtTime() ָ��ʱ�䷢����Ϣ�� sendMessageDelayed() �Ӻ��ύ��Ϣ��
HandlerThread �� quit() �� quitSafety() ��ɶ����

�����������ö��ǽ��� Looper �����С����ǵ������ǣ�quit() ������ֱ���Ƴ� MessageQueue �е�������Ϣ��Ȼ����ֹ MesseageQueue���� quitSafety() �Ὣ MessageQueue �����е���Ϣ������ɺ󣨲��ٽ�������Ϣ������ֹ MessageQueue��