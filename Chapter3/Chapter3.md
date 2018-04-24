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
                System.out.println("this num = "+num + " Thread  name="+Thread.currentThread().getName());
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
**发布**一个对象是指对象能够在当前作用域之外的区域的代码中使用。
当某个不应当发布的对象被发布时，这种情况称之为**逸出**。
发布对象最简单的方法是将对象的引用保存到一个公有的**静态变量**之中。
- 发布对象的方式：
1. 将对象的引用存储到公共静态域中。
2. 从非私有方法中返回引用。
3. 将一个对象传递给外部方法，相当于发布这个对象了。
4. 发布一个内部类实例。内引类实例包装了对封装类实例的隐含引用。

       class UnsafeStates{
    		private String[] states = new String[]{"AK", "AL"};
    		public String[] getStates(){
    			return states;
            }
 		}
        私有变量发布，逸出。
        public class SafeListener{
      	private final EventListener listener;
      	private safeListener(){
          listener = new EventListener(){
              public void onEvent(Event e){
                  doSomething(e);
              	}
          	}
     	}
            public static SafeListener newInstance(EventSource source){
             SafeListener safeListener = new SafeListener();
             safeListener.registerListener(safeListener.listener);

             return safeListener;
     		}
	 	}
        构造方法结束后再发布
---------------------------------------
## 线程封闭
线程封闭即是在**单线程**内访问数据。当某个对象被封闭在一个线程中时，这种用法将自动实现线程安全，即使线程封闭的对象是不安全的。
- **Ad-hoc线程封闭**
这是完全靠实现者控制的线程封闭，他的线程封闭完全靠实现者实现。Ad-hoc线程封闭非常脆弱，没有任何一种语言特性能将对象封闭到目标线程上。
- **栈封闭**
封闭是我们编程当中遇到的最多的线程封闭。什么是栈封闭呢？简单的说就是**局部变量**。多个线程访问一个方法，此方法中的局部变量都会被拷贝一分儿到线程栈中。所以局部变量是不被多个线程所共享的，也就不会出现并发问题。所以能用局部变量就别用全局的变量，全局变量容易引起并发问题。

		public class ThreadClosedDemo
		{

            public int loadNum()
            {
                int i = 0;

                if (Thread.currentThread().getName().equals("T1"))
                {
                    try
                    {
                        i = 100;
                        Thread.sleep(2000);
                    }
                    catch (InterruptedException e)
                    {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
                else if (Thread.currentThread().getName().equals("T2"))
                {
                    i = 200;
                }

                System.out.println("thread name = " + Thread.currentThread().getName() + "  i = " + i);

                return i;
            }

            public static void main(String[] args)
            {
                ThreadClosedDemo threadClosedDemo = new ThreadClosedDemo();
                new Thread(new Runnable()
                {

                    @Override
                    public void run()
                    {
                        threadClosedDemo.loadNum();
                    }
                }, "T1").start();
                new Thread(new Runnable()
                {

                    @Override
                    public void run()
                    {
                        threadClosedDemo.loadNum();
                    }
                }, "T2").start();
            }
		}
- **ThreadLocal类**
使用ThreadLocal是实现线程封闭的最好方法，其实ThreadLocal内部维护了一个Map，Map的key是每个线程的名称，而Map的值就是我们要封闭的对象。每个线程中的对象都对应着Map中一个值，也就是ThreadLocal利用Map实现了对象的线程封闭。

        public class ThreadLocalTest
        {

            private ThreadLocal<Integer> num = new ThreadLocal<Integer>()
            {
                protected Integer initialValue()
                {
                    return 0;
                }
            };

            public int getNum()
            {
                if (Thread.currentThread().getName().equals("T1"))
                {
                    try
                    {
                        num.set(100);
                        Thread.sleep(2000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                else if (Thread.currentThread().getName().equals("T2"))
                {
                    try
                    {
                        num.set(200);
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                else
                {
                    num.set(50);
                    // num.remove();
                }
                return num.get();
            }

            public static void main(String[] args)
            {
                ThreadLocalTest threadLocalTest = new ThreadLocalTest();
                System.out.println(Thread.currentThread().getName() + ":" + threadLocalTest.getNum());
                new Thread(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        System.out.println(Thread.currentThread().getName() + ":" + threadLocalTest.getNum());
                    }
                }, "T1").start();
                new Thread(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        System.out.println(Thread.currentThread().getName() + ":" + threadLocalTest.getNum());
                    }
                }, "T2").start();
                System.out.println("end:" + Thread.currentThread().getName() + ":" + threadLocalTest.getNum());
            }
    	}
	- void set(Object value)设置当前线程的线程局部变量的值。
	- public Object get()该方法返回当前线程所对应的线程局部变量。
	- public void remove()将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是JDK 5.0新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。
	- protected Object initialValue()返回该线程局部变量的初始值，该方法是一个protected的方法，显然是为了让子类覆盖而设计的。这个方法是一个延迟调用方法，在线程第1次调用get()或set(Object)时才执行，并且仅执行1次。ThreadLocal中的缺省实现直接返回一个null。
-----------------------------
## 不变性
不可变对象一定是**线程安全**的。（不可变对象：对象被创建之后，其状态不能被修改）。
当满足以下条件时，对象才是不可变的：
- 对象创建之后其状态不能修改。
- 对象的所有域都是final类型。
- 对象是正确创建的（对象创建期间，this引用没有逸出）。
--------------------
### final域
- 当在类中定义变量时，在其前面加上final关键字，这个变量一旦被初始化便不可改变，这里不可改变的意思对基本类型来说是其值不可变，而对于对象变量来说其引用不可再变
- 其初始化可以在两个地方，一是其定义处，也就是说在final变量定义时直接给其赋值，二是在构造函数中 。这两个 地方只能选其一，要么在定义时给值，要么在构造函数中给值，不能同时既在定义时给了值，又在构造函数中给另外的值。
- 引用对象 can never be changed to point to another object. However, 引用对象 can be modified;
----------------------------
### 使用Volatile类型发布不可变对象
	public class OneValueCache {
        private final BigInteger lastNumber;
        private final BigInteger[] lastFactors;

        public OneValueCache(BigInteger i, BigInteger[] factors) {
            lastNumber = i;
            lastFactors = Arrays.copyOf(factors, factors.length);
        }

        public BigInteger[] getFactors(BigInteger i) {
            if (lastNumber == null || !lastNumber.equals(i))
                return null;
            else
                return Arrays.copyOf(lastFactors, lastFactors.length);
        }
	}
    public class VolatileCachedFactorizer extends GenericServlet implements Servlet {
        private volatile OneValueCache cache = new OneValueCache(null, null);

        public void service(ServletRequest req, ServletResponse resp) {
            BigInteger i = extractFromRequest(req);//通过request获得请求的整数i
            BigInteger[] factors = cache.getFactors(i);//对该整数i进行因式分解
            if (factors == null) {
                factors = factor(i);
                cache = new OneValueCache(i, factors);//对于不同的请求生成不同的缓存
            }
            encodeIntoResponse(resp, factors);//返回response
        }
    }
--------------------------
## 安全发布
- **不正确的发布：正确的对象被破坏**：失效值。
- **不可变对象与初始化安全性**：任何线程都可以在不需要额外同步的情况下安全访问不可变对象，即使在发布这些对象时没有使用同步。（如果final类型域指向的为可变对象，在访问这些域对象时仍需同步）。
- **安全发布的常用模式**：
	- 在静态初始化函数中初始化一个对象引用
	- 将对象的引用保存到volatile类型或者Atomic对象中
	- 将对象引用保存到某个正确构造对象的final类型域中
	- 将对象引用保存到一个由锁保护的域中
- **事实不可变对象**：对象可变，在其发布后状态不会再改变。（在没有额外同步的情况下，任何线程都可以安全的使用被安全发布的事实不可变对象）
- **可变对象**（不可变对象可通过任意机制发布，事实不可变对象须通过安全方式发布，可变对象须安全发布，且需要线程安全或锁保护）
- **安全的共享对象**
在并发程序中使用或共享对象时，可使用的策略：
	- 线程封闭
	- 只读共享（不可变对象，事实不可变对象）
	- 线程安全共享（内部同步）
	- 保护对象（锁）
