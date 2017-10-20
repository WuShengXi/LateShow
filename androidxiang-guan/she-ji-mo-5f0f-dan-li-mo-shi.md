### 单例模式 {#单例模式}

首先了解一些单例模式的概念。

> 确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

这样做有以下几个优点

* 对于那些比较耗内存的类，只实例化一次可以大大提高性能，尤其是在移动开发中。
* 保持程序运行的时候该中始终只有一个实例存在内存中

其实单例有很多种实现方式，但是个人比较倾向于其中1种。可以见单例模式

代码如下

```java
public class Singleton {
    private static volatile Singleton instance = null;

    private Singleton(){
    }
 
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

要保证单例，需要做一下几步

* 必须防止外部可以调用构造函数进行实例化，因此构造函数必须私有化。
* 必须定义一个静态函数获得该单例
* 单例使用volatile修饰
* 使用synchronized 进行同步处理，并且双重判断是否为null，我们看到synchro nized \(Singleton.class\)里面又进行了是否为null的判断，这是因为一个线程 进入了该代码，如果另一个线程在等待，这时候前一个线程创建了一个实例出 来完毕后，另一个线程获得锁进入该同步代码，实例已经存在，没必要再次创 建，因此这个判断是否是null还是必须的。

至于单例的并发测试，可以使用CountDownLatch，使用await\(\)等待锁释放，使用countDown\(\)释放锁从而达到并发的效果。可以见下面的代码

```java
public static void main(String[] args) {
	final CountDownLatch latch = new CountDownLatch(1);
	int threadCount = 1000;
	for (int i = 0; i < threadCount; i++) {
		new Thread() {
			@Override
			public void run() {
				try {
					latch.await();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Singleton.getInstance().hashCode());
			}
		}.start();
	}
	latch.countDown();
}
```

看看打印出来的hashCode会不会出现不一样即可，理论上是全部都一样的。

而在Android中，很多地方用到了单例。

比如Android-Universal-Image-Loader中的单例

```java
private volatile static ImageLoader instance;
/** Returns singleton class instance */
public static ImageLoader getInstance() {
	if (instance == null) {
		synchronized (ImageLoader.class) {
			if (instance == null) {
				instance = new ImageLoader();
			}
		}
	}
	return instance;
}
```

比如EventBus中的单例

```java
private static volatile EventBus defaultInstance;
public static EventBus getDefault() {
	if (defaultInstance == null) {
		synchronized (EventBus.class) {
			if (defaultInstance == null) {
				defaultInstance = new EventBus();
			}
		}
	}
	return defaultInstance;
}
```

上面的单例都是比较规规矩矩的，当然实际上有很多单例都是变了一个样子，但本质还是单例。

如InputMethodManager 中的单例

```java
static InputMethodManager sInstance;
public static InputMethodManager getInstance() {
	synchronized (InputMethodManager.class) {
		if (sInstance == null) {
			IBinder b = ServiceManager.getService(Context.INPUT_METHOD_SERVICE);
			IInputMethodManager service = IInputMethodManager.Stub.asInterface(b);
			sInstance = new InputMethodManager(service, Looper.getMainLooper());
		}
		return sInstance;
	}
}
```

AccessibilityManager 中的单例，看代码这么长，其实就是进行了一些判断，还是一个单例

```java
private static AccessibilityManager sInstance;
public static AccessibilityManager getInstance(Context context) {
	synchronized (sInstanceSync) {
		if (sInstance == null) {
			final int userId;
			if (Binder.getCallingUid() == Process.SYSTEM_UID
					|| context.checkCallingOrSelfPermission(
							Manifest.permission.INTERACT_ACROSS_USERS)
									== PackageManager.PERMISSION_GRANTED
					|| context.checkCallingOrSelfPermission(
							Manifest.permission.INTERACT_ACROSS_USERS_FULL)
									== PackageManager.PERMISSION_GRANTED) {
				userId = UserHandle.USER_CURRENT;
			} else {
				userId = UserHandle.myUserId();
			}
			IBinder iBinder = ServiceManager.getService(Context.ACCESSIBILITY_SERVICE);
			IAccessibilityManager service = IAccessibilityManager.Stub.asInterface(iBinder);
			sInstance = new AccessibilityManager(context, service, userId);
		}
	}
	return sInstance;
}
```

当然单例还有很多种写法，比如恶汉式，有兴趣的自己去了解就好了。

最后，我们应用一下单例模式。典型的一个应用就是管理我们的Activity，下面这个可以作为一个工具类，代码也很简单，也不做什么解释了。

```java
public class ActivityManager {

	private static volatile ActivityManager instance;
	private Stack<Activity> mActivityStack = new Stack<Activity>();
	
	private ActivityManager(){
		
	}
	
	public static ActivityManager getInstance(){
		if (instance == null) {
		synchronized (ActivityManager.class) {
			if (instance == null) {
				instance = new ActivityManager();
			}
		}
		return instance;
	}
	
	public void addActicity(Activity act){
		mActivityStack.push(act);
	}
	
	public void removeActivity(Activity act){
		mActivityStack.remove(act);
	}
	
	public void killMyProcess(){
		int nCount = mActivityStack.size();
		for (int i = nCount - 1; i >= 0; i--) {
        	Activity activity = mActivityStack.get(i);
        	activity.finish();
        }
		
		mActivityStack.clear();
		android.os.Process.killProcess(android.os.Process.myPid());
	}
}
```

这个类可以在开源中国的几个客户端中找到类似的源码

* Git@OSC中的AppManager
* android-app中的AppManager

以上两个类是一样的，没区别。

