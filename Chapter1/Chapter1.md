# 并发简史
## 并发简史
为了在计算机中加入操作系统实现多程序同时执行的原因：
- **资源利用率**（在等待时运行另一个程序，提高资源利用率）
- **公平性**（不同的用户和程序对于计算机上的资源有着同等使用权）
- **便利性**（计算多个任务时，多程序运行，程序之间相互通信比只编写一个程序来计算所有任务容易实现）
----------------------------------
## 线程的优势
- **发挥处理器强大能力**
- **建模的简单性**
- **异步事件的简化处理**
- **响应更灵敏的用户界面**
----------------------------------
## 线程带来的风险
- **安全性问题**
    	private int x = 4;

		@Override
		public void run() {
		//synchronized (this) {
			if (x > 0) {
				try {
					Thread.sleep(5);
				} catch (Exception e) {
					System.out.println(e);
				}
				--x;
				System.out.println(x);
			}
		//}
		}
    	public static void main(String[] args) {
		ThreadSafety threadSafety = new ThreadSafety();
		Thread t1 = new Thread(threadSafety);
		Thread t2 = new Thread(threadSafety);
		Thread t3 = new Thread(threadSafety);
		Thread t4 = new Thread(threadSafety);
		Thread t5 = new Thread(threadSafety);
		t1.start();
		t2.start();
		t3.start();
		t4.start();
		t5.start();
		}
根据运行结果发现，在不添加synchronized (this)时，线程存在安全问题，运行结果有可能会错乱或小于0的数字，在添加锁之后可解决线程安全性问题。
- **活跃性问题**
例如线程A在等待线程B释放其持有的资源，而线程B永远都不释放该资源，那么A就会永久等待下去。导致线程活跃性问题是难以分析的，因为它们依赖于不同的线程事件发生时序，开发或测试中并不能总是重现。（死锁，活锁，饥饿）
		public int flag = 1;
	
		private static Object o1 = new Object();
	
		private static Object o2 = new Object();
	
		@Override
		public void run() {
		if (flag == 1) {
			synchronized (o1) {
				try {
					System.out.println("in flag 1");
					Thread.sleep(500);
				} catch (Exception e) {
					e.printStackTrace();
				}
				synchronized (o2) {
					System.err.println("1");
				}
			}
		}
		if (flag == 2) {
			synchronized (o2) {
				try {
					System.out.println("in flag 2");
					Thread.sleep(500);
				} catch (Exception e) {
					e.printStackTrace();
				}
				synchronized (o1) {
					System.out.println("2");
				}
			}
		}
		}
	
		public static void main(String[] args) {
		DeadLockThread deadLockThread1 = new DeadLockThread();
		DeadLockThread deadLockThread2 = new DeadLockThread();
		deadLockThread1.flag = 1;
		deadLockThread2.flag = 2;
		Thread t1 = new Thread(deadLockThread1);
		Thread t2 = new Thread(deadLockThread2);
		t1.start();
		t2.start();
		}
当DeadLockThread类的对象flag==1时（t1），先锁定o1,睡眠500毫秒；而t1在睡眠的时候另一个flag==2的对象（t2）线程启动，先锁定o2,睡眠500毫秒；t1睡眠结束后需要锁定o2才能继续执行，而此时o2已被t2锁定；t2睡眠结束后需要锁定o1才能继续执行，而此时o1已被t1锁定；t1、t2相互等待，都需要得到对方锁定的资源才能继续执行，从而死锁。
- **性能问题**
性能问题包括多个方面，例如服务时间过长，响应不灵敏，吞吐率过低，资源消耗过高或可伸缩性较低等。
--------------------------------
## 线程无处不在
每个java应用程序都会使用线程。
