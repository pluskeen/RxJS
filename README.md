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
> 案例演示不共享

订阅Observable就像是调用函数，并提供了接收函数返回值的回调函数。

subscribe订阅Observable，会让Observable开始执行，并将值或者事件传递给本次执行的观察者。

### 执行Observable ###

Observalbe的执行表示`Rx.Observable.create(function subscribe(observer){...}`中的`...`代码开始运行。

它是惰性运算，只有在观察者订阅之后才会执行。

随着时间的推移会以同步或者异步的方式产生多个值。

Observable的执行过程是运行可以传递值的三种通知类型：

    * "Next"通知：发送一个值，比如数字、字符串、对象等。
    * "Error"通知：发送一个JavaScript错误或异常。
    * "Complete"通知：不再发送任何值。

"Next"通知是最常用的类型，表示传递数据给观察者。

"Error"通知和"Complete"通知可能只会在Observable执行期间发生一次，并且只会执行其中一个。

在Observable执行期间，可能会发送无数个"Next"通知。

如果发送的是"Error"或者"Complete"通知的话，那么之后就不会再发送任何通知了。

    var observable = Rx.Observable.create(function subscribe(observer){
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.complete();
        observer.next(4); // 不会发送
    })

使用try/catch来捕获异常，如果捕获到异常的话，会发送"Error"通知。

    var observable = Rx.Observable.create(function subscribe(observer){
        try{
            observer.next(1);
            observer.next(2);
            observer.next(3);
            observer.complete();
        } catch{
            observer.error();
        }
    })

### 清理Observable ###

观察者可以将其对应的Observable执行中止。

当调用了observable.subscribe后，会返回一个Subscription对象：

    var subscription = observable.subscribe(x => console.log(x));

subscription表示进行中的执行，使用`subscription.unsubscribe()`可以取消进行中的执行：

    var observable = Rx.Observable.form([1,2,3]); // 使用创建操作符
    var subscription = observable.subscribe(x => console.log(x));
    subscription.unsubscribe();

如果使用create()方法创建Observable时，**Observable必须定义如何清理执行**，可以通过在function subscribe()中返回一个自定义的unsubscribe函数。

    var observable = Rx.Observable.create(fucntion subscribe(observer){
        var intervalId = setInterval(() => {
            observer.next('hi');
        },1000);

        // 提供清理interval方法
        return function unsubscribe(){
            clearInterval(intervalId);
        };
    });

### Observable与函数 ###

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

## observer(观察者) ##

观察者会消费Observable发送的值，观察者只是一组回调函数的集合，每个回调函数对应一种Observable发送的通知类型：next、error和complete。

一个典型的观察者对象：

    var observer = {
        next: (x) => console.log('Observer got a next value: ' + x),
        error: (error) => console.log('Observer got an error: ' + error),
        complete: () => console.log('complete notification')
    }

使用时，将它提供给Observable的subscribe方法：

    observable.subscribe(observer);

观察者对象是由三个回调函数组成的对象，也可以不满三个。如果没有提供某个回调函数，Observable也会成功执行，只是某些通知类型会被忽略。

也可以直接将回调函数当作参数传递给observable.subscribe()方法：

    observable.subscribe(
        x => console.log(x),
        err => console.log(err),
        () => console.log('complete')
    )

第一个回调函数对应接收"Next"通知，第二个回调函数接收"Error"通知，第三个回调函数接收"Complete"通知。

## Subscription(订阅) ##

Subscription表示可清理资源的对象，通常Observable的执行会返回此对象。

它有一个重要的方法，unsubscribe，不需要任何参数，只是用来清理由Subscription占用的资源，即取消Observable的执行。

    var observable = Rx.Observable.interval(100);
    var subcription = observable.subscribe(
        x => console.log(x)
    );
    subscription.unsubscribe(); // 取消observable的执行

多个Subscription可以合并在一起，这样对于合并的Subscription调用一次unsubscribe方法就可以取消多个Observable的执行。

    var observable1 = Rx.Observable.interval(400);
    var observable2 = Rx.Observalbe.interval(300);

    var subscription1 = observable1.subscribe(x => console.log(x));
    var subscription2 = observable2.subscribe(x => console.log(x));

    subscription1.add(subscription2); // subscription2添加到subscription1

    setTimeout(() => {
        subscription1.unsubscribe(); // subscription1和subscription2都会取消
    },1000)

相对add的添加方法，有remove移除方法，用来撤销一个已添加的Subscription

    subscription1.remove(subscription2);

## Subject(主体) ##

Subject是一种特殊的Observable，可以将值多播给多个观察者，普通的Observable是单播的。