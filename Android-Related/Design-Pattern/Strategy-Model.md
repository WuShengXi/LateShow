## 策略模式

看下策略模式的定义

> 策略模式定义了一些列的算法，并将每一个算法封装起来，而且使它们还可以相互替换。策略模式让算法独立于使用它的客户而独立变换。

乍一看，也没看出个所以然来。举个栗子吧。

假设我们要出去旅游，而去旅游出行的方式有很多，有步行，有坐火车，有坐飞机等等。而如果不使用任何模式，我们的代码可能就是这样子的。

```java
public class TravelStrategy {

	enum Strategy{
		WALK,PLANE,SUBWAY
	}

	private Strategy strategy;
	public TravelStrategy(Strategy strategy){
		this.strategy=strategy;
	}

	public void travel(){
		if(strategy==Strategy.WALK){
			print("walk");
		}else if(strategy==Strategy.PLANE){
			print("plane");
		}else if(strategy==Strategy.SUBWAY){
			print("subway");
		}
	}

	public void print(String str){
		System.out.println("出行旅游的方式为："+str);
	}

	public static void main(String[] args) {
		TravelStrategy walk=new TravelStrategy(Strategy.WALK);
		walk.travel();

		TravelStrategy plane=new TravelStrategy(Strategy.PLANE);
		plane.travel();

		TravelStrategy subway=new TravelStrategy(Strategy.SUBWAY);
		subway.travel();
	}
}
```

这样做有一个致命的缺点，一旦出行的方式要增加，我们就不得不增加新的else if语句，而这违反了面向对象的原则之一，对修改封闭。而这时候，策略模式则可以完美的解决这一切。

首先，需要定义一个策略接口。

```java
public interface Strategy {
	void travel();
}
```

然后根据不同的出行方式实行对应的接口

```java
public class WalkStrategy implements Strategy{

	@Override
	public void travel() {
		System.out.println("walk");
	}

}
```

```java
public class PlaneStrategy implements Strategy{

	@Override
	public void travel() {
		System.out.println("plane");
	}

}
```

```java
public class SubwayStrategy implements Strategy{

	@Override
	public void travel() {
		System.out.println("subway");
	}

}
```

此外还需要一个包装策略的类，并调用策略接口中的方法

```java
public class TravelContext {
	Strategy strategy;

	public Strategy getStrategy() {
		return strategy;
	}

	public void setStrategy(Strategy strategy) {
		this.strategy = strategy;
	}

	public void travel() {
		if (strategy != null) {
			strategy.travel();
		}
	}
}
```

测试一下代码

```java
public class Main {
	public static void main(String[] args) {
		TravelContext travelContext=new TravelContext();
		travelContext.setStrategy(new PlaneStrategy());
		travelContext.travel();
		travelContext.setStrategy(new WalkStrategy());
		travelContext.travel();
		travelContext.setStrategy(new SubwayStrategy());
		travelContext.travel();
	}
}
```

输出结果如下

> plane
> walk
> subway

可以看到，应用了策略模式后，如果我们想增加新的出行方式，完全不必要修改现有的类，我们只需要实现策略接口即可，这就是面向对象中的对扩展开放准则。假设现在我们增加了一种自行车出行的方式。只需新增一个类即可。

```java
public class BikeStrategy implements Strategy{

	@Override
	public void travel() {
		System.out.println("bike");
	}

}
```

之后设置策略即可

```java
public class Main {
	public static void main(String[] args) {
		TravelContext travelContext=new TravelContext();
		travelContext.setStrategy(new BikeStrategy());
		travelContext.travel();
	}
}
```

而在Android的系统源码中,策略模式也是应用的相当广泛的.最典型的就是属性动画中的应用.

我们知道,在属性动画中,有一个东西叫做插值器,它的作用就是根据时间流逝的百分比来来计算出当前属性值改变的百分比.

我们使用属性动画的时候,可以通过set方法对插值器进行设置.可以看到内部维持了一个时间插值器的引用，并设置了getter和setter方法，默认情况下是先加速后减速的插值器，set方法如果传入的是null，则是线性插值器。而时间插值器TimeInterpolator是个接口，有一个接口继承了该接口，就是Interpolator这个接口，其作用是为了保持兼容

```java
private static final TimeInterpolator sDefaultInterpolator = new AccelerateDecelerateInterpolator();
private TimeInterpolator mInterpolator = sDefaultInterpolator;

@Override
public void setInterpolator(TimeInterpolator value) {
	if (value != null) {
		mInterpolator = value;
	} else {
		mInterpolator = new LinearInterpolator();
	}
}

@Override
public TimeInterpolator getInterpolator() {
	return mInterpolator;
}
```

```java
public interface Interpolator extends TimeInterpolator {
    // A new interface, TimeInterpolator, was introduced for the new android.animation
    // package. This older Interpolator interface extends TimeInterpolator so that users of
    // the new Animator-based animations can use either the old Interpolator implementations or
    // new classes that implement TimeInterpolator directly.
}
```

此外还有一个BaseInterpolator插值器实现了Interpolator接口，并且是一个抽象类

```java
abstract public class BaseInterpolator implements Interpolator {
    private int mChangingConfiguration;
    /**
     * @hide
     */
    public int getChangingConfiguration() {
        return mChangingConfiguration;
    }

    /**
     * @hide
     */
    void setChangingConfiguration(int changingConfiguration) {
        mChangingConfiguration = changingConfiguration;
    }
}
```

平时我们使用的时候，通过设置不同的插值器，实现不同的动画速率变换效果，比如线性变换，回弹，自由落体等等。这些都是插值器接口的具体实现，也就是具体的插值器策略。我们略微来看几个策略。

```java
public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {

    public LinearInterpolator() {
    }

    public LinearInterpolator(Context context, AttributeSet attrs) {
    }

    public float getInterpolation(float input) {
        return input;
    }

    /** @hide */
    @Override
    public long createNativeInterpolator() {
        return NativeInterpolatorFactoryHelper.createLinearInterpolator();
    }
}
```

```java
public class AccelerateDecelerateInterpolator extends BaseInterpolator
        implements NativeInterpolatorFactory {
    public AccelerateDecelerateInterpolator() {
    }

    @SuppressWarnings({"UnusedDeclaration"})
    public AccelerateDecelerateInterpolator(Context context, AttributeSet attrs) {
    }

    public float getInterpolation(float input) {
        return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
    }

    /** @hide */
    @Override
    public long createNativeInterpolator() {
        return NativeInterpolatorFactoryHelper.createAccelerateDecelerateInterpolator();
    }
}
```

内部使用的时候直接调用getInterpolation方法就可以返回对应的值了，也就是属性值改变的百分比。

属性动画中另外一个应用策略模式的地方就是估值器，它的作用是根据当前属性改变的百分比来计算改变后的属性值。该属性和插值器是类似的，有几个默认的实现。其中TypeEvaluator是一个接口。

```java
public interface TypeEvaluator<T> {

    public T evaluate(float fraction, T startValue, T endValue);

}
```

```java
public class IntEvaluator implements TypeEvaluator<Integer> {

    public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
        int startInt = startValue;
        return (int)(startInt + fraction * (endValue - startInt));
    }
}
```

```java
public class FloatEvaluator implements TypeEvaluator<Number> {

    public Float evaluate(float fraction, Number startValue, Number endValue) {
        float startFloat = startValue.floatValue();
        return startFloat + fraction * (endValue.floatValue() - startFloat);
    }
}
```

```java
public class PointFEvaluator implements TypeEvaluator<PointF> {

    private PointF mPoint;


    public PointFEvaluator() {
    }

    public PointFEvaluator(PointF reuse) {
        mPoint = reuse;
    }

    @Override
    public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
        float x = startValue.x + (fraction * (endValue.x - startValue.x));
        float y = startValue.y + (fraction * (endValue.y - startValue.y));

        if (mPoint != null) {
            mPoint.set(x, y);
            return mPoint;
        } else {
            return new PointF(x, y);
        }
    }
}
```

```java
public class ArgbEvaluator implements TypeEvaluator {
    private static final ArgbEvaluator sInstance = new ArgbEvaluator();

    public static ArgbEvaluator getInstance() {
        return sInstance;
    }

    public Object evaluate(float fraction, Object startValue, Object endValue) {
        int startInt = (Integer) startValue;
        int startA = (startInt >> 24) & 0xff;
        int startR = (startInt >> 16) & 0xff;
        int startG = (startInt >> 8) & 0xff;
        int startB = startInt & 0xff;

        int endInt = (Integer) endValue;
        int endA = (endInt >> 24) & 0xff;
        int endR = (endInt >> 16) & 0xff;
        int endG = (endInt >> 8) & 0xff;
        int endB = endInt & 0xff;

        return (int)((startA + (int)(fraction * (endA - startA))) << 24) |
                (int)((startR + (int)(fraction * (endR - startR))) << 16) |
                (int)((startG + (int)(fraction * (endG - startG))) << 8) |
                (int)((startB + (int)(fraction * (endB - startB))));
    }
}
```

上面的都是一些系统实现好的估值策略，在内部调用估值器的evaluate方法即可返回改变后的值了。我们也可以自定义估值策略。这里就不展开了。

当然，在开源框架中，策略模式也是无处不在的。

首先在Volley中，策略模式就能看到。

有一个重试策略接口

```java
public interface RetryPolicy {


    public int getCurrentTimeout();//获取当前请求用时（用于 Log）


    public int getCurrentRetryCount();//获取已经重试的次数（用于 Log）


    public void retry(VolleyError error) throws VolleyError;//确定是否重试，参数为这次异常的具体信息。在请求异常时此接口会被调用，可在此函数实现中抛出传入的异常表示停止重试。
}
```

在Volley中，该接口有一个默认的实现DefaultRetryPolicy，Volley 默认的重试策略实现类。主要通过在 retry\(…\) 函数中判断重试次数是否达到上限确定是否继续重试。

```java
public class DefaultRetryPolicy implements RetryPolicy {
	...
}
```

而策略的设置是在Request类中

```java
public abstract class Request<T> implements Comparable<Request<T>> {
    private RetryPolicy mRetryPolicy;
    public Request<?> setRetryPolicy(RetryPolicy retryPolicy) {
        mRetryPolicy = retryPolicy;
        return this;
    }
	public RetryPolicy getRetryPolicy() {
        return mRetryPolicy;
    }
}
```

此外，各大网络请求框架，或多或少都会使用到缓存，缓存一般会定义一个Cache接口，然后实现不同的缓存策略，如内存缓存，磁盘缓存等等，这个缓存的实现，其实也可以使用策略模式。直接看Volley，里面也有缓存。

定义了一个缓存接口

```java
/**
 * An interface for a cache keyed by a String with a byte array as data.
 */
public interface Cache {
    /**
     * Retrieves an entry from the cache.
     * @param key Cache key
     * @return An {@link Entry} or null in the event of a cache miss
     */
    public Entry get(String key);

    /**
     * Adds or replaces an entry to the cache.
     * @param key Cache key
     * @param entry Data to store and metadata for cache coherency, TTL, etc.
     */
    public void put(String key, Entry entry);

    /**
     * Performs any potentially long-running actions needed to initialize the cache;
     * will be called from a worker thread.
     */
    public void initialize();

    /**
     * Invalidates an entry in the cache.
     * @param key Cache key
     * @param fullExpire True to fully expire the entry, false to soft expire
     */
    public void invalidate(String key, boolean fullExpire);

    /**
     * Removes an entry from the cache.
     * @param key Cache key
     */
    public void remove(String key);

    /**
     * Empties the cache.
     */
    public void clear();

    /**
     * Data and metadata for an entry returned by the cache.
     */
    public static class Entry {
        /** The data returned from cache. */
        public byte[] data;

        /** ETag for cache coherency. */
        public String etag;

        /** Date of this response as reported by the server. */
        public long serverDate;

        /** The last modified date for the requested object. */
        public long lastModified;

        /** TTL for this record. */
        public long ttl;

        /** Soft TTL for this record. */
        public long softTtl;

        /** Immutable response headers as received from server; must be non-null. */
        public Map<String, String> responseHeaders = Collections.emptyMap();

        /** True if the entry is expired. */
        public boolean isExpired() {
            return this.ttl < System.currentTimeMillis();
        }

        /** True if a refresh is needed from the original data source. */
        public boolean refreshNeeded() {
            return this.softTtl < System.currentTimeMillis();
        }
    }

}
```

它有两个实现类**NoCache**和**DiskBasedCache**，使用的时候设置对应的缓存策略即可。

在android开发中，ViewPager是一个使用非常常见的控件，它的使用往往需要伴随一个Indicator指示器。**如果让你重新实现一个ViewPager，并且带有Indicator**，这时候，你会不会想到用策略模式呢？在你自己写的ViewPager中（不是系统的，当然你也可以继承系统的）持有一个策略接口Indicator的变量，通过set方法设置策略，然后ViewPager滑动的时候调用策略接口的对应方法改变指示器。默认提供几个Indicator接口的实现类，比如圆形指示器CircleIndicator、线性指示器LineIndicator、Tab指示器TabIndicator、图标指示器IconIndicator 等等等等。有兴趣的话自己去实现一个吧。
