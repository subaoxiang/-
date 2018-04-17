# 对象的共享
## 可见性
可见性是指在读操作和写操作不同的线程中执行时，读取的值与写入的值不一致。通常，我们无法确保执行读操作的线程能适时看到**其他线程**写操作的值。为了确保多个线程之间对内存写入操作的可见性，必须使用**同步**机制。
- **失效数据**：除非每次在访问变量时都加同步，否则则可能获取到的变量是一个失效值。失效值不一定会**同时出现**，有可能某变量为最新值，某变量为失效值。
		public class InvalidThread
		{
    		private int num;

            public int getNum()
            {
                return num;
            }

            public void setNum()
            {
        		this.num++;
                System.out.println("this num = "+num++ + " Thread  name="+Thread.currentThread().getName());
    		}

            public static void main(String[] args)
            {
                InvalidThread invalidThread = new InvalidThread();
                new Thread(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        invalidThread.setNum();
                    }
                }).start();
                System.out.println(invalidThread.getNum());
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                System.out.println(invalidThread.getNum());
            }
		}
        该段代码中set方法中打印的数字顺序不正常，证明有失效数据存在。
- **非原子的64位操作**
当线程在没有**同步**的情况下读取变量时，有可能会读取到失效值，但是该值为线程设置的值，并非随机值，这种安全保证被称为最低安全性。
最低安全性不适于64位的非volatile类型数值变量（long，double），JVM会将他们的读写操作分成两个32位的单独操作，即变量的读写并非**单线程**。因此即使不考虑失效数据问题，long与double也是不安全的，需要用**volatile**去声明或**加锁**。
- **加锁与可见性**
加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程可以看到共享变量的最新值，所有读写线程都必须在同一个锁上同步。
- **Volatile变量**
加锁机制既可以即可以确保可见性又可以确保原子性，而volatile变量只能确保可见性。
当且仅当满足以下条件时才应该使用volatile变量：
	- 对变量的**写入**操作不依赖变量的**当前值**，或者能确定只有**单线程**更新变量的值
	- 该变量不会与其他**状态变量**一起纳入不变性条件中
	- 在访问变量时不需要**加锁**
--------------------------------
## 发布与溢出
