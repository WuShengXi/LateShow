前面介绍了单例模式和Builder模式，这部分着重介绍一下观察者模式。先看下这个模式的定义。

> 定义对象间的一种一对多的依赖关系，当一个对象的状态发送改变时，所有依赖于它的对象都能得到通知并被自动更新

还是那句话，定义往往是抽象的，要深刻的理解定义，你需要自己动手实践一下。

先来讲几个情景。

* 情景1

  > 有一种短信服务，比如天气预报服务，一旦你订阅该服务，你只需按月付费，付完费后，每天一旦有天气信息更新，它就会及时向你发送最新的天气信息。

* 情景2

  > 杂志的订阅，你只需向邮局订阅杂志，缴纳一定的费用，当有新的杂志时，邮局会自动将杂志送至你预留的地址。

观察上面两个情景，有一个共同点，就是我们无需每时每刻关注我们感兴趣的东西，我们只需做的就是订阅感兴趣的事物，比如天气预报服务，杂志等，一旦我们订阅的事物发生变化，比如有新的天气预报信息，新的杂志等，被订阅的事物就会即时通知到订阅者，即我们。而这些被订阅的事物可以拥有多个订阅者，也就是一对多的关系。当然，严格意义上讲，这个一对多可以包含一对一，因为一对一是一对多的特例，没有特殊说明，本文的一对多包含了一对一。

现在你反过头来看看观察者模式的定义，你是不是豁然开朗了。

然后我们看一下观察者模式的几个重要组成。

* 观察者，我们称它为Observer，有时候我们也称它为订阅者，即Subscriber
* 被观察者，我们称它为Observable，即可以被观察的东西，有时候还会称之为主题，即Subject

至于观察者模式的具体实现，这里带带大家实现一下场景一，其实java中提供了**Observable**类和**Observer接口**供我们快速的实现该模式，但是为了加深印象，我们不使用这两个类。

场景1中我们感兴趣的事情是天气预报，于是，我们应该定义一个Weather实体类。

```java
public class Weather {
    private String description;

    public Weather(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    @Override
    public String toString() {
        return "Weather{" +
                "description='" + description + '\'' +
                '}';
    }
}
```

然后定义我们的被观察者，我们想要这个被观察者能够通用，将其定义成泛型。内部应该暴露register和unregister方法供观察者订阅和取消订阅，至于观察者的保存，直接用ArrayList即可，此外，当有主题内容发送改变时，会即时通知观察者做出反应，因此应该暴露一个notifyObservers方法，以上方法的具体实现见如下代码。

```java
public class Observable<T> {
    List<Observer<T>> mObservers = new ArrayList<Observer<T>>();

    public void register(Observer<T> observer) {
        if (observer == null) {
            throw new NullPointerException("observer == null");
        }
        synchronized (this) {
            if (!mObservers.contains(observer))
                mObservers.add(observer);
        }
    }

    public synchronized void unregister(Observer<T> observer) {
        mObservers.remove(observer);
    }

    public void notifyObservers(T data) {
        for (Observer<T> observer : mObservers) {
            observer.onUpdate(this, data);
        }
    }

}
```

而我们的观察者，只需要实现一个观察者的接口Observer，该接口也是泛型的。其定义如下。

```java
public interface Observer<T> {
    void onUpdate(Observable<T> observable,T data);
}
```

一旦订阅的主题发送变换就会回调该接口。

我们来使用一下，我们定义了一个天气变换的主题，也就是被观察者，还有两个观察者观察天气变换，一旦变换了，就打印出天气信息，注意一定要调用被观察者的register进行注册，否则会收不到变换信息。而一旦不敢兴趣了，直接调用unregister方法进行取消注册即可

```java
public class Main {
    public static void main(String [] args){
        Observable<Weather> observable=new Observable<Weather>();
        Observer<Weather> observer1=new Observer<Weather>() {
            @Override
            public void onUpdate(Observable<Weather> observable, Weather data) {
                System.out.println("观察者1："+data.toString());
            }
        };
        Observer<Weather> observer2=new Observer<Weather>() {
            @Override
            public void onUpdate(Observable<Weather> observable, Weather data) {
                System.out.println("观察者2："+data.toString());
            }
        };

        observable.register(observer1);
        observable.register(observer2);


        Weather weather=new Weather("晴转多云");
        observable.notifyObservers(weather);

        Weather weather1=new Weather("多云转阴");
        observable.notifyObservers(weather1);

        observable.unregister(observer1);

        Weather weather2=new Weather("台风");
        observable.notifyObservers(weather2);

    }
}
```

最后的输出结果也是没有什么问题的，如下

> 观察者1：Weather{description=’晴转多云’}  
> 观察者2：Weather{description=’晴转多云’}  
> 观察者1：Weather{description=’多云转阴’}  
> 观察者2：Weather{description=’多云转阴’}  
> 观察者2：Weather{description=’台风’}

接下来我们看看观察者模式在android中的应用。我们从最简单的开始。还记得我们为一个Button设置点击事件的代码吗。

```java
Button btn=new Button(this);
btn.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.e("TAG","click");
    }
});
```

其实严格意义上讲，这个最多算是回调，但是我们可以将其看成是一对一的观察者模式，即只有一个观察者。

其实只要是set系列的设置监听器的方法最多都只能算回调，但是有一些监听器式add进去的，这种就是观察者模式了，比如RecyclerView中的addOnScrollListener方法

```java
private List<OnScrollListener> mScrollListeners;
public void addOnScrollListener(OnScrollListener listener) {
    if (mScrollListeners == null) {
        mScrollListeners = new ArrayList<OnScrollListener>();
    }
    mScrollListeners.add(listener);
}
public void removeOnScrollListener(OnScrollListener listener) {
    if (mScrollListeners != null) {
        mScrollListeners.remove(listener);
    }
}
public void clearOnScrollListeners() {
    if (mScrollListeners != null) {
        mScrollListeners.clear();
    }
}
```

然后有滚动事件时便会触发观察者进行方法回调

```java
public abstract static class OnScrollListener {
    public void onScrollStateChanged(RecyclerView recyclerView, int newState){}
    public void onScrolled(RecyclerView recyclerView, int dx, int dy){}
}

void dispatchOnScrolled(int hresult, int vresult) {
    //...
    if (mScrollListeners != null) {
        for (int i = mScrollListeners.size() - 1; i >= 0; i--) {
            mScrollListeners.get(i).onScrolled(this, hresult, vresult);
        }
    }
}
void dispatchOnScrollStateChanged(int state) {
    //...
    if (mScrollListeners != null) {
        for (int i = mScrollListeners.size() - 1; i >= 0; i--) {
            mScrollListeners.get(i).onScrollStateChanged(this, state);
        }
    }
}
```

类似的方法很多很多，都是add监听器系列的方法，这里也不再举例。

还有一个地方就是Android的广播机制，其本质也是观察者模式，这里为了简单方便，直接拿本地广播的代码说明，即LocalBroadcastManager。

我们平时使用本地广播主要就是下面四个方法

```java
LocalBroadcastManager localBroadcastManager = LocalBroadcastManager.getInstance(this);
localBroadcastManager.registerReceiver(BroadcastReceiver receiver, IntentFilter filter);
localBroadcastManager.unregisterReceiver(BroadcastReceiver receiver);
localBroadcastManager.sendBroadcast(Intent intent)
```

调用registerReceiver方法注册广播，调用unregisterReceiver方法取消注册，之后直接使用sendBroadcast发送广播，发送广播之后，注册的广播会收到对应的广播信息，这就是典型的观察者模式。具体的源代码这里也不贴。

android系统中的观察者模式还有很多很多，有兴趣的自己去挖掘，接下来我们看一下一些开源框架中的观察者模式。一说到开源框架，你首先想到的应该是EventBus。没错，EventBus也是基于观察者模式的。

观察者模式的三个典型方法它都具有，即注册，取消注册，发送事件

```java
EventBus.getDefault().register(Object subscriber);
EventBus.getDefault().unregister(Object subscriber);

EventBus.getDefault().post(Object event);
```

内部源码也不展开了。接下来看一下重量级的库，它就是RxJava，由于学习曲线的陡峭，这个库让很多人望而止步。

创建一个被观察者

```java
Observable<String> myObservable = Observable.create(  
    new Observable.OnSubscribe<String>() {  
        @Override  
        public void call(Subscriber<? super String> sub) {  
            sub.onNext("Hello, world!");  
            sub.onCompleted();  
        }  
    }  
);
```

创建一个观察者，也就是订阅者

```java
Subscriber<String> mySubscriber = new Subscriber<String>() {  
    @Override  
    public void onNext(String s) { System.out.println(s); }  

    @Override  
    public void onCompleted() { }  

    @Override  
    public void onError(Throwable e) { }  
};
```

观察者进行事件的订阅

```java
myObservable.subscribe(mySubscriber);
```

具体源码也不展开，不过RxJava这个开源库的源码个人还是建议很值得去看一看的。

总之，在Android中观察者模式还是被用得很频繁的。

