### 原型模式

这部分介绍的模式其实很简单，即原型模式，按照惯例，先看定义。

> 用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。

这是什么鬼哦，本宝宝看不懂！不必过度在意这些定义，自己心里明白就ok了。没代码你说个jb。

首先我们定义一个Person类

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52  | public class Person{ private String name; private int age; private double height; private double weight;  public Person\(\){      }  public String getName\(\) { return name;     }  public void setName\(String name\) { this.name = name;     }  public int getAge\(\) { return age;     }  public void setAge\(int age\) { this.age = age;     }  public double getHeight\(\) { return height;     }  public void setHeight\(double height\) { this.height = height;     }  public double getWeight\(\) { return weight;     }  public void setWeight\(double weight\) { this.weight = weight;     }  @Override public String toString\(\) { return "Person{" + "name='" + name + '\'' + ", age=" + age + ", height=" + height + ", weight=" + weight + '}';     } }  |
| :--- | :--- |


要实现原型模式，只需要按照下面的几个步骤去实现即可。

* 实现Cloneable接口

| 1 2 3  | public class Person implements Cloneable{  }  |
| :--- | :--- |


* 重写Object的clone方法

| 1 2 3 4  | @Override public Object clone\(\){ return null; }  |
| :--- | :--- |


* 实现clone方法中的拷贝逻辑

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14  | @Override public Object clone\(\){ 	Person person=null; try { 		person=\(Person\)super.clone\(\); 		person.name=this.name; 		person.weight=this.weight; 		person.height=this.height; 		person.age=this.age; 	} catch \(CloneNotSupportedException e\) { 		e.printStackTrace\(\); 	} return person; }  |
| :--- | :--- |


测试一下

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17  | public class Main { public static void main\(String \[\] args\){         Person p=new Person\(\);         p.setAge\(18\);         p.setName\("张三"\);         p.setHeight\(178\);         p.setWeight\(65\);         System.out.println\(p\);          Person p1= \(Person\) p.clone\(\);         System.out.println\(p1\);          p1.setName\("李四"\);         System.out.println\(p\);         System.out.println\(p1\);     } }  |
| :--- | :--- |


输出结果如下

> Person{name=’张三’, age=18, height=178.0, weight=65.0}  
> Person{name=’张三’, age=18, height=178.0, weight=65.0}  
> Person{name=’张三’, age=18, height=178.0, weight=65.0}  
> Person{name=’李四’, age=18, height=178.0, weight=65.0}

试想一下，两个不同的人，除了姓名不一样，其他三个属性都一样，用原型模式进行拷贝就会显得异常简单，这也是原型模式的应用场景之一。

> 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用，即保护性拷贝。

但是假设Person类里还有一个属性叫兴趣爱好，是一个List集合，就像这样子

| 1 2 3 4 5 6 7 8 9  | private ArrayList&lt;String&gt; hobbies=new ArrayList&lt;String&gt;\(\);  public ArrayList&lt;String&gt;getHobbies\(\) { return hobbies; }  public void setHobbies\(ArrayList&lt;String&gt; hobbies\) { this.hobbies = hobbies; }  |
| :--- | :--- |


在进行拷贝的时候要格外注意，如果你直接按之前的代码那样拷贝

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15  | @Override public Object clone\(\){ 	Person person=null; try { 		person=\(Person\)super.clone\(\); 		person.name=this.name; 		person.weight=this.weight; 		person.height=this.height; 		person.age=this.age; 		person.hobbies=this.hobbies; 	} catch \(CloneNotSupportedException e\) { 		e.printStackTrace\(\); 	} return person; }  |
| :--- | :--- |


会带来一个问题

使用测试代码进行测试

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23  | public class Main { public static void main\(String \[\] args\){         Person p=new Person\(\);         p.setAge\(18\);         p.setName\("张三"\);         p.setHeight\(178\);         p.setWeight\(65\);         ArrayList &lt;String&gt; hobbies=new ArrayList&lt;String&gt;\(\);         hobbies.add\("篮球"\);         hobbies.add\("编程"\);         hobbies.add\("长跑"\);         p.setHobbies\(hobbies\);         System.out.println\(p\);          Person p1= \(Person\) p.clone\(\);         System.out.println\(p1\);          p1.setName\("李四"\);         p1.getHobbies\(\).add\("游泳"\);         System.out.println\(p\);         System.out.println\(p1\);     } }  |
| :--- | :--- |


我们拷贝了一个对象，并添加了一个兴趣爱好进去，看下打印结果

> Person{name=’张三’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑\]}  
> Person{name=’张三’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑\]}  
> Person{name=’张三’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑, 游泳\]}  
> Person{name=’李四’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑, 游泳\]}

你会发现原来的对象的hobby也发生了变换。

其实导致这个问题的本质原因是我们只进行了浅拷贝，也就是只拷贝了引用，最终两个对象指向的引用是同一个，一个发生变化另一个也会发生变换，显然解决方法就是使用深拷贝。

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16  | @Override public Object clone\(\){ 	Person person=null; try { 		person=\(Person\)super.clone\(\); 		person.name=this.name; 		person.weight=this.weight; 		person.height=this.height; 		person.age=this.age;  		person.hobbies=\(ArrayList&lt;String&gt;\)this.hobbies.clone\(\); 	} catch \(CloneNotSupportedException e\) { 		e.printStackTrace\(\); 	} return person; }  |
| :--- | :--- |


注意**person.hobbies=\(ArrayList\)this.hobbies.clone\(\);**，不再是直接引用而是进行了一份拷贝。再运行一下，就会发现原来的对象不会再发生变化了。

> Person{name=’张三’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑\]}  
> Person{name=’张三’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑\]}  
> Person{name=’张三’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑\]}  
> Person{name=’李四’, age=18, height=178.0, weight=65.0, hobbies=\[篮球, 编程, 长跑, 游泳\]}

其实有时候我们会更多的看到原型模式的另一种写法。

* 在clone函数里调用构造函数，构造函数的入参是该类对象。

| 1 2 3 4  | @Override public Object clone\(\){ return new Person\(this\); }  |
| :--- | :--- |


* 在构造函数中完成拷贝逻辑

| 1 2 3 4 5 6 7  | public Person\(Person person\){ this.name=person.name; this.weight=person.weight; this.height=person.height; this.age=person.age; this.hobbies= new ArrayList&lt;String&gt;\(hobbies\); }  |
| :--- | :--- |


其实都差不多，只是写法不一样。

现在来挖挖android中的原型模式。

先看Bundle类，该类实现了Cloneable接口

| 1 2 3 4 5 6 7 8 9  | public Object clone\(\) { return new Bundle\(this\); }  public Bundle\(Bundle b\) { super\(b\);  	mHasFds = b.mHasFds; 	mFdsKnown = b.mFdsKnown; }  |
| :--- | :--- |


然后是Intent类，该类也实现了Cloneable接口

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28  | @Override public Object clone\(\) { return new Intent\(this\); } public Intent\(Intent o\) { this.mAction = o.mAction; this.mData = o.mData; this.mType = o.mType; this.mPackage = o.mPackage; this.mComponent = o.mComponent; this.mFlags = o.mFlags; this.mContentUserHint = o.mContentUserHint; if \(o.mCategories != null\) { this.mCategories = new ArraySet&lt;String&gt;\(o.mCategories\); 	} if \(o.mExtras != null\) { this.mExtras = new Bundle\(o.mExtras\); 	} if \(o.mSourceBounds != null\) { this.mSourceBounds = new Rect\(o.mSourceBounds\); 	} if \(o.mSelector != null\) { this.mSelector = new Intent\(o.mSelector\); 	} if \(o.mClipData != null\) { this.mClipData = new ClipData\(o.mClipData\); 	} }  |
| :--- | :--- |


用法也显得十分简单，一旦我们要用的Intent与现有的一个Intent很多东西都是一样的，那我们就可以直接拷贝现有的Intent，再修改不同的地方，便可以直接使用。

| 1 2 3 4 5 6  | Uri uri = Uri.parse\("smsto:10086"\);     Intent shareIntent = new Intent\(Intent.ACTION\_SENDTO, uri\);     shareIntent.putExtra\("sms\_body", "hello"\);      Intent intent = \(Intent\)shareIntent.clone\(\) ; startActivity\(intent\);  |
| :--- | :--- |


网络请求中一个最常见的开源库OkHttp中，也应用了原型模式。它就在**OkHttpClient**这个类中，它实现了Cloneable接口

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31  | /\*\* Returns a shallow copy of this OkHttpClient. \*/ @Override  public OkHttpClient clone\(\) { return new OkHttpClient\(this\); } private OkHttpClient\(OkHttpClient okHttpClient\) { this.routeDatabase = okHttpClient.routeDatabase; this.dispatcher = okHttpClient.dispatcher; this.proxy = okHttpClient.proxy; this.protocols = okHttpClient.protocols; this.connectionSpecs = okHttpClient.connectionSpecs; this.interceptors.addAll\(okHttpClient.interceptors\); this.networkInterceptors.addAll\(okHttpClient.networkInterceptors\); this.proxySelector = okHttpClient.proxySelector; this.cookieHandler = okHttpClient.cookieHandler; this.cache = okHttpClient.cache; this.internalCache = cache != null ? cache.internalCache : okHttpClient.internalCache; this.socketFactory = okHttpClient.socketFactory; this.sslSocketFactory = okHttpClient.sslSocketFactory; this.hostnameVerifier = okHttpClient.hostnameVerifier; this.certificatePinner = okHttpClient.certificatePinner; this.authenticator = okHttpClient.authenticator; this.connectionPool = okHttpClient.connectionPool; this.network = okHttpClient.network; this.followSslRedirects = okHttpClient.followSslRedirects; this.followRedirects = okHttpClient.followRedirects; this.retryOnConnectionFailure = okHttpClient.retryOnConnectionFailure; this.connectTimeout = okHttpClient.connectTimeout; this.readTimeout = okHttpClient.readTimeout; this.writeTimeout = okHttpClient.writeTimeout; }  |
| :--- | :--- |


正如开头的注释**Returns a shallow copy of this OkHttpClient**，该clone方法返回了一个当前对象的浅拷贝对象。

至于其他框架中的原型模式，请读者自行发现。

