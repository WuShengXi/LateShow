### Build模式

那么什么是Builder模式。你通过搜索，会发现大部分网上的定义都是

> 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

但是看完这个定义，并没有什么卵用，你依然不知道什么是Builder设计模式。在此个人的态度是学习设计模式这种东西，不要过度在意其定义，定义往往是比较抽象的，学习它最好的例子就是通过样例代码。

我们通过一个例子来引出Builder模式。假设有一个Person类，我们通过该Person类来构建一大批人，这个Person类里有很多属性，最常见的比如name，age，weight，height等等，并且我们允许这些值不被设置，也就是允许为null，该类的定义如下。

|  |  |
| ---: | :--- |


然后我们为了方便可能会定义一个构造方法。

| 1 2 3 4 5 6  | public Person\(String name, int age, double height, double weight\) { this.name = name; this.age = age; this.height = height; this.weight = weight; }  |
| :--- | :--- |


或许为了方便new对象，你还会定义一个空的构造方法

| 1 2  | public Person\(\) { }  |
| :--- | :--- |


甚至有时候你很懒，只想传部分参数，你还会定义如下类似的构造方法。

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14  | public Person\(String name\) { this.name = name; }  public Person\(String name, int age\) { this.name = name; this.age = age; }  public Person\(String name, int age, double height\) { this.name = name; this.age = age; this.height = height; }  |
| :--- | :--- |


于是你就可以这样创建各个需要的对象

| 1 2 3 4 5  | Person p1=new Person\(\); Person p2=new Person\("张三"\); Person p3=new Person\("李四",18\); Person p4=new Person\("王五",21,180\); Person p5=new Person\("赵六",17,170,65.4\);  |
| :--- | :--- |


可以想象一下这样创建的坏处，最直观的就是四个参数的构造函数的最后面的两个参数到底是什么意思，可读性不怎么好，如果不点击看源码，鬼知道哪个是weight哪个是height。还有一个问题就是当有很多参数时，编写这个构造函数就会显得异常麻烦，这时候如果换一个角度，试试Builder模式，你会发现代码的可读性一下子就上去了。

我们给Person增加一个静态内部类Builder类，并修改Person类的构造函数，代码如下。

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72  | public class Person { private String name; private int age; private double height; private double weight;      privatePerson\(Builder builder\) { this.name=builder.name; this.age=builder.age; this.height=builder.height; this.weight=builder.weight;     } public String getName\(\) { return name;     }  public void setName\(String name\) { this.name = name;     }  public int getAge\(\) { return age;     }  public void setAge\(int age\) { this.age = age;     }  public double getHeight\(\) { return height;     }  public void setHeight\(double height\) { this.height = height;     }  public double getWeight\(\) { return weight;     }  public void setWeight\(double weight\) { this.weight = weight;     }  static class Builder{ private String name; private int age; private double height; private double weight; public Builder name\(String name\){ this.name=name; return this;         } public Builder age\(int age\){ this.age=age; return this;         } public Builder height\(double height\){ this.height=height; return this;         }  public Builder weight\(double weight\){ this.weight=weight; return this;         }  public Person build\(\){ return new Person\(this\);         }     } }  |
| :--- | :--- |


从上面的代码中我们可以看到，我们在Builder类里定义了一份与Person类一模一样的变量，通过一系列的成员函数进行设置属性值，但是返回值都是this，也就是都是Builder对象，最后提供了一个build函数用于创建Person对象，返回的是Person对象，对应的构造函数在Person类中进行定义，也就是构造函数的入参是Builder对象，然后依次对自己的成员变量进行赋值，对应的值都是Builder对象中的值。此外Builder类中的成员函数返回Builder对象自身的另一个作用就是让它支持链式调用，使代码可读性大大增强。

于是我们就可以这样创建Person类。

| 1 2 3 4 5 6 7  | Person.Builder builder=new Person.Builder\(\); Person person=builder 		.name\("张三"\) 		.age\(18\) 		.height\(178.5\) 		.weight\(67.4\) 		.build\(\);  |
| :--- | :--- |


有没有觉得创建过程一下子就变得那么清晰了。对应的值是什么属性一目了然，可读性大大增强。

其实在Android中， Builder模式也是被大量的运用。比如常见的对话框的创建

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18  | AlertDialog.Builder builder=new AlertDialog.Builder\(this\); AlertDialog dialog=builder.setTitle\("标题"\) 		.setIcon\(android.R.drawable.ic\_dialog\_alert\) 		.setView\(R.layout.myview\) 		.setPositiveButton\(R.string.positive, new DialogInterface.OnClickListener\(\) { @Override public void onClick\(DialogInterface dialog, int which\) {  			} 		}\) 		.setNegativeButton\(R.string.negative, new DialogInterface.OnClickListener\(\) { @Override public void onClick\(DialogInterface dialog, int which\) {  			} 		}\) 		.create\(\); dialog.show\(\);  |
| :--- | :--- |


其实在java中有两个常见的类也是Builder模式，那就是StringBuilder和StringBuffer，只不过其实现过程简化了一点罢了。

我们再找找Builder模式在各个框架中的应用。

如Gson中的GsonBuilder，代码太长了，就不贴了，有兴趣自己去看源码，这里只贴出其Builder的使用方法。

| 1 2 3 4 5 6  | GsonBuilder builder=new GsonBuilder\(\); Gson gson=builder.setPrettyPrinting\(\) 		.disableHtmlEscaping\(\) 		.generateNonExecutableJson\(\) 		.serializeNulls\(\) 		.create\(\);  |
| :--- | :--- |


还有EventBus中也有一个Builder，只不过这个Builder外部访问不到而已，因为它的构造函数不是public的，但是你可以在EventBus这个类中看到他的应用。

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22  | public static EventBusBuilder builder\(\) { return new EventBusBuilder\(\); } public EventBus\(\) { this\(DEFAULT\_BUILDER\); } EventBus\(EventBusBuilder builder\) { 	subscriptionsByEventType = new HashMap&lt;Class&lt;?&gt;, CopyOnWriteArrayList&lt;Subscription&gt;&gt;\(\); 	typesBySubscriber = new HashMap&lt;Object, List&lt;Class&lt;?&gt;&gt;&gt;\(\); 	stickyEvents = new ConcurrentHashMap&lt;Class&lt;?&gt;, Object&gt;\(\); 	mainThreadPoster = new HandlerPoster\(this, Looper.getMainLooper\(\), 10\); 	backgroundPoster = new BackgroundPoster\(this\); 	asyncPoster = new AsyncPoster\(this\); 	subscriberMethodFinder = new SubscriberMethodFinder\(builder.skipMethodVerificationForClasses\); 	logSubscriberExceptions = builder.logSubscriberExceptions; 	logNoSubscriberMessages = builder.logNoSubscriberMessages; 	sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent; 	sendNoSubscriberEvent = builder.sendNoSubscriberEvent; 	throwSubscriberException = builder.throwSubscriberException; 	eventInheritance = builder.eventInheritance; 	executorService = builder.executorService; }  |
| :--- | :--- |


再看看著名的网络请求框架OkHttp

| 1 2 3 4 5  | Request.Builder builder=new Request.Builder\(\); Request request=builder.addHeader\("",""\) 	.url\(""\) 	.post\(body\) 	.build\(\);  |
| :--- | :--- |


除了Request外，Response也是通过Builder模式创建的。贴一下Response的构造函数

| 1 2 3 4 5 6 7 8 9 10 11 12  | private Response\(Builder builder\) { this.request = builder.request; this.protocol = builder.protocol; this.code = builder.code; this.message = builder.message; this.handshake = builder.handshake; this.headers = builder.headers.build\(\); this.body = builder.body; this.networkResponse = builder.networkResponse; this.cacheResponse = builder.cacheResponse; this.priorResponse = builder.priorResponse; }  |
| :--- | :--- |


可见各大框架中大量的运用了Builder模式。最后总结一下

* 定义一个静态内部类Builder，内部的成员变量和外部类一样
* Builder类通过一系列的方法用于成员变量的赋值，并返回当前对象本身（this）
* Builder类提供一个build方法或者create方法用于创建对应的外部类，该方法内部调用了外部类的一个私有构造函数，该构造函数的参数就是内部类Builder
* 外部类提供一个私有构造函数供内部类调用，在该构造函数中完成成员变量的赋值，取值为Builder对象中对应的值



