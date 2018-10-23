# RxJS #

RxJS是一个库，它通过使用observable序列来编写异步和基于事件的程序。

在RxJS中用来解决异步事件管理的基本概念是：

    * Observable(可观察对象):表示一个可调用的未来值或事件的集合。
    * Observer(观察者):一个回调函数的集合，它知道如何去监听由Observable提供的值。
    * Subscription(订阅者):表示Observable的执行，主要用于取消Observable执行。
    * Operators(操作符):采用函数式编程风格的纯函数，使用像map、filter、concat、flatMap等操作符来处理集合。
    * Subject(主体):相当于EventEmitter，并且是将值或事件多路推送给多个Observer的唯一方式。
    * Schedulers(调度器):用来控制并发并且是中央集权的调度员，允许在发生计算时进行协调。

## Observable(可观察对象) ##

Observalbe是使用Rx.Observable.create或创建操作符创建的，并使用观察者来订阅它，然后执行它并发送next / error / complete通知给观察者，而且执行可能会被清理。

Observable的核心关注点：

    * 创建
    * 订阅
    * 执行
    * 清理

### 创建Observable ###

Rx.Observable.create是Observable构造函数的别名，它接收一个参数：subscribe函数。

让我们来创建一个例子：

    var observable = Rx.Observable.create(function subscribe(observer){
      var id = setInterval(() => {
        observer.next('hello');
      },1000);
    });

上面例子会每隔1秒发送字符串'hello'。

> Observable可以使用create创建，也可以使用操作符（of、form、interval等）来创建。

### 订阅Observable ###

上面例子创建的observable对象可以订阅：

    observable.subscribe(...);

对于同一个Observable，不同的观察者订阅是不共享的。

> todo!!!

订阅Observable就像是调用函数，并提供了接收函数返回值的回调函数。

subscribe订阅Observable，会让Observable开始执行，并将值或者事件传递给本次执行的观察者。

### 执行Observable ###

Observalbe的执行表示`Rx.Observable.create(function subscribe(observer){...}`中的`...`代码开始运行。

它是惰性运算，只有在观察者订阅之后才会执行。

随着时间的推移会以同步或者异步的方式产生多个值。

Observable执行可以传递三种类型的值：

    * "Next"通知：发送一个值，比如数字、字符串、对象等。
    * "Error"通知：发送一个JavaScript错误或异常。
    * "Complete"通知：不再发送任何值。

Observalbe可以随着时间的推移**返回**多个值，这是函数所做不到的。

    function foo(){
      console.log("Hello");
      return 42;
      return 2; // 这行是永远不会执行的
    }

函数只能返回一个值，但Observable可以这样：

    var foo = Rx.Observable.create(function(observer){
      console.log("Hello");
      observer.next(1); // 第一次返回1
      observer.next(2); // 随着时间推移返回2
      observer.next(3); // 最后返回3
    })

    console.log('before');
    foo.subscribe(function(x){
      console.log(x);
    })
    console.log('after');

同步输出：

    'before'
    'Hello'
    1
    2
    3
    'after'

> Observable传递值可以是同步的，也可以是异步的。

所以可以这样：

    var foo = Rx.Observable.create(function(observer){
      console.log("Hello");
      observer.next(1);
      observer.next(2);
      observer.next(3);
      setTimeout(() => {
        observer.next(4); // 异步返回4
      },300)
    })

    console.log('before');
    foo.subscribe(function(x){
      console.log(x);
    })
    console.log('after');

查看输出

    'before'
    'Hello'
    1
    2
    3
    'after'
    4 // 异步输出