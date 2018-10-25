# RxJS #

RxJS是一个库，它通过使用observable序列来编写异步和基于事件的程序。

在RxJS中用来解决异步事件管理的基本概念是：

* [Observable(可观察对象)][Observable]:表示一个可调用的未来值或事件的集合。
* [Observer(观察者)][Observer]:一个回调函数的集合，它知道如何去监听由Observable提供的值。
* [Subscription(订阅者)][Subscription]:表示Observable的执行，主要用于取消Observable执行。
* [Operators(操作符)][Operators]:采用函数式编程风格的纯函数，使用像map、filter、concat、flatMap等操作符来处理集合。
* [Subject(主体)][Subject]:相当于EventEmitter，并且是将值或事件多路推送给多个Observer的唯一方式。
* [Schedulers(调度器)][Schedulers]:用来控制并发并且是中央集权的调度员，允许在发生计算时进行协调。

[Observable]: #Observable(可观察对象)
[Observer]: #Observer(观察者)
[Subscription]: #Subscription(订阅者)
[Operators]: #Operators(操作符)
[Subject]: #Subject(主体)
[Schedulers]: #Schedulers(调度器)

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

> Observable可以使用create创建，也可以使用操作符（of、from、interval等）来创建。

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

    var observable = Rx.Observable.from([1,2,3]); // 使用创建操作符
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

## Subscription(订阅者) ##

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

每个Subject都是Observable，可以对其使用subscribe方法。对于观察者而言，无法判断Observable的执行是来自普通的Observable还是Subject。在Subject内部，subscribe不会开启新执行，它只是将给定的观察者注册到观察者列表中。

    var subject = new Rx.Subject();

    subject.subscribe({
        next: (x) => console.log('A' + x)
    });
    subject.subscribe({
        next: (x) => console.log('B' + x)
    })

    subject.next(1);
    subject.next(2);

控制台打印

    A 1
    B 1
    A 2
    B 2

每个Subject都是观察者，Subject中有next(x)、error(e)和compelet()方法。要给Subject提供新值可以调用next(value)方法，它会将值多播给已注册监听该Subject的观察者们。

    var subject = new Rx.Subject();

    // 观察者订阅subject
    subject.subscribe({
        next: (x) => console.log('A' + x)
    });
    subject.subscribe({
        next: (x) => console.log('B' + x)
    })

    var observable = new Rx.Observable.from([1,2]);
    observable.subscribe(subject); // 提供一个subject进行订阅

控制台打印

    A 1
    B 1
    A 2
    B 2

Subject是将任意Observable执行共享给多个观察者的唯一方式。

Subject还有一些特殊类型：BehaviorSubject、ReplaySubject、AsyncSubject。

### 多播的Observable ###

多播Observable是通过Subject来发送通知，这个Subject可能有多个订阅者，然而普通的单播Observalbe只发送通知给单个观察者。

在底层，观察者订阅一个基础的Subject，然后Subject订阅源Observable。

使用多播操作符multicast改写上面的例子：

    var source = new Rx.Observable.from([1,2,3]);
    var subject = new Rx.Subject();
    var multicasted = source.multicast(subject);

    // 在底层使用了subject.subscribe({...})
    multicasted.subscribe({
        next: (x) => console.log('A' + x)
    });
    multicasted.subscribe({
        next: (x) => console.log('B' + x)
    });

    // 在底层使用了source.subscribe(subject)
    multicasted.connect();

multicast操作符返回ConnectableObservable，它是有connect()方法的Observable。

connect()方法十分重要，它决定了何时启动共享的Observable执行，因为connect()方法在底层执行了source.subscribe(subject)，所以它返回的是Subscription。

通常，当第一个观察者订阅时应该让其自动连接，当最后一个观察者取消订阅时应该自动取消连接（取消共享执行）。

可以这样概述Subscription发生的经过：

1. 第一个观察者订阅了多播Observable
2. 多播Observable已连接，开始执行
3. next值0发送给第一个观察者
4. 第二个观察者订阅了多播Observable
5. next值1发送给第一个观察者
6. next值1发送给第二个观察者
7. 第一个观察者取消订阅了多播的Observable
8. next值2发送给第二个观察者
9. 第二个观察者取消订阅了多播的Observable
10. 多播Observable的连接中断，取消订阅

要实现上面的过程，需要显示调用connect()：

    var source = new Rx.Observable.interval(500);
    var subject = new Rx.Subject();
    var multicasted = source.multicast(subject);
    var subscription1, subscription2, subscriptionConnect;

    subscription1 = multicasted.subscribe({
        next: (x) => console.log('A' + x)
    })

    // 第一个观察者订阅后，调用connect()连接
    subscriptionConnect = multicasted.connect();

    // 第二个观察者订阅
    setTimeout(() => {
        subscription2 = multicasted.subscribe({
            next: (x) => console.log('B' + x)
        });
    },600)

    // 第一个观察者取消订阅
    setTimeout(() => {
        subscription1.unsubscribe();
    },1300)

    // 第二个观察者取消订阅
    // 要取消multicasted的执行，因为此后没有订阅者了
    setTimeout(() => {
        subscription2.unsubscribe();
        subscriptionConnect.unsubscribe();
    },3000)

上面例子中显式调用了connect()，并不是最优解。

multicast操作符返回的ConnectObservable中有个refCount()方法，这个方法返回Observable，这个Observable会追踪有多少个订阅者。当订阅者数量从0变成1，它会调用connect()开启共享执行，当订阅者数量从1变成0时，它会完全取消订阅，停止进一步执行。

> refCount方法的作用是，当有第一个订阅者时，多播Observable会自动启动执行，而当最后一个订阅者离开，多播Observable会自动停止执行。

改进上面的例子：

    var source = new Rx.Observable.interval(500);
    var subject = new Rx.Subject();
    var refCounted = source.multicast(subject).refCount();
    var subscription1, subscription2;

    // 共享的Observable开始执行
    subscription1 = refCounted.subscribe({
        next: (x) => console.log('A' + x)
    })

    // 第二个观察者订阅
    setTimeout(() => {
        subscription2 = refCounted.subscribe({
            next: (x) => console.log('B' + x)
        });
    },600)

    // 第一个观察者取消订阅
    setTimeout(() => {
        subscription1.unsubscribe();
    },1300)

    // 第二个观察者取消订阅
    // 这里共享的Observable会停止执行
    setTimeout(() => {
        subscription2.unsubscribe();
    },3000)

需要注意的是，refCount()只存在与ConnectObservable中，它返回的是Observable。

### BehaviorSubject ###

BehaviorSubject保存了发送给消费者的最新值，并且当有新的观察者订阅时，会立即从BehaviorSubject那接收到当前值，即最后发送给消费者的值。

使用BehaviorSubject时，需要提供初始值：

    var subject = new Rx.BehaviorSubject(0); // 0是初始值

    subject.subscribe({
        next: (x) => console.log('A' + x)
    })

    subject.next(1);
    subject.next(2);

    subject.subscribe({
        next: (x) => console.log('B' + x)
    })

    subject.next(3);

控制台打印

    A 0
    A 1
    A 2
    B 2 // B订阅后，立即获取到当前值2
    A 3
    B 3

### ReplaySubject ###

ReplaySubject可以发送旧值给新的订阅者，还可以记录Observable执行的一部分。
它将记录Observable执行中的多个值并将其回放给新的订阅者。

当创建ReplaySubject时，可以指定回放多少个值：

    var subject = new Rx.Observable.ReplaySubjec(3);

    subject.subscribe({
        next: (x) => console.log('A' + x)
    })

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
        next: (x) => console.log('B' + x)
    })

    subject.next(5);

控制台打印

    A 1
    A 2
    A 3
    A 4
    B 2 // B订阅后，立即获取回放的最后3个值
    B 3
    B 4
    A 5
    B 5

ReplaySubject还接收第二个参数，用来确定多久之前的值可以记录，单位毫秒。

    var subject = new Rx.ReplaySubject(100,500);

    subject.subscribe({
        next: (x) => console.log('A' + x)
    })

    var i = 1;
    setInterval(() => subject.next(i++),400);

    setTimeout(() => {
        subject.subscribe({
            next: (x) => console.log('B' + x)
        })
    },1000)

控制台打印

    A 1 // 延迟400ms
    A 2 // 延迟400ms
    B 2 // 延迟1000ms，B订阅了，立即获取前500ms内产生的值2
    A 3
    B 3
    ...

### AsyncSubject ###

当Observable执行完，也就是complete()执行后，会将最后一个值发送给观察者。

    var subject = new Rx.AsyncSubject();

    subject.subscribe({
        next: (x) => console.log('A' + x)
    })

    subject.next(1);
    subject.next(2);
    subject.next(3);

    subject.subscribe({
        next: (x) => console.log('B' + x)
    })

    subject.next(4);
    subject.complete();

控制台打印

    A 4
    B 4

AsyncSubject和last()操作符类似，都是等待complete()通知，再发送一个单值。

## Operators(操作符) ##

操作符是Observable上的方法，比如map、filter、merge等等。当操作符被调用时，他们不会改变原来的Observable实例，而是返回新的Observable，它的Subscription逻辑基于第一个Observable。

操作符的本质其实就是纯函数，它接收一个Observable作为输入，并生成一个新的Observable作为输出。订阅输出的Observable会自动订阅到作为输入的Observable。

可以自定义操作符函数：

    function multiplyByTen(input){
        var output = Rx.Observable.create(function subscribe(observer){
            input.subscribe({
                next: (x) => observer.next(x * 10),
                error: (err) => observer.error(err),
                complete: () => observer.complete()
            });
        });
        return output;
    }

    var input = Rx.Observable.from([1, 2, 3, 4]);
    var output = multiplyByTen(input);
    output.subscribe(x => console.log(x));

控制台打印

    10
    20
    30
    40

上面可以发现，订阅output会导致input也被订阅，称之为操作符订阅链。

### 实例操作符 ###

实例操作符是Observable实例上的方法，试着把上面的例子改为实例操作符：

    Rx.Observable.prototype.multiplyByTen = function(){
        var input = this;
        return Rx.Observable.create(function subscribe(observer){
            input.subscribe({
                next: (x) => observer.next(x * 10),
                error: (err) => observer.error(err),
                complete: () => observer.complete()
            });
        })
    }

实例操作符的特点是使用this关键字来指代输入参数为Observable的函数。

### 静态操作符 ###

静态操作符是定义在Observable类上的。它内部不使用this关键字，而是依赖与它的参数。它只接收非Observable参数，比如数字，然后创建一个新的Observable。

典型的静态操作符例子是interval函数：

    var observable = Rx.Observable.interval(100);

## Schedulers(调度器) ##

调度器控制何时启动Subscription和何时发送通知。它由三部分组成：

* 调度器是一种数据结构。它知道如何根据优先级或其他标准来存储任务和将任务进行排序。
* 调度器是执行上下文。它表示在何时何地执行任务，比如立即执行或者回调函数机制。
* 调度器有一个虚拟时钟。通过调度器的now()方法提供了时间的概念。调度器上的任务将严格遵循该时钟所表示的时间。

调度器可以规定Observable在什么样的执行上下文中发送通知给观察者。

### 调度器类型 ###

Rx.Scheduler.queue 适合在有递归情况下使用。

Rx.Scheduler.asap 非同步执行，在setTimeout中执行。

Rx.Scheduler.async 通常跟时间有关的操作符会用到，使用setInterval执行。

### 使用调度器 ###

使用操作符observeOn来指定调度器：

    var observable = Rx.Observable.create(function (observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.complete();
    })
    .observeOn(Rx.Scheduler.async);

    console.log('just before subscribe');
    observable.subscribe({
        next: x => console.log('got value ' + x),
        error: err => console.error('something wrong occurred: ' + err),
        complete: () => console.log('done'),
    });
    console.log('just after subscribe');

控制台打印

    just before subscribe
    just after subscribe
    got value 1
    got value 2
    got value 3
    done

上面的例子原本是同步执行的，但是用了`observeOn(Rx.Scheduler.async)`后变成了非同步执行。

静态创建操作符可以接收调度器作为第二个参数。

    Rx.Observable.from(array,scheduler)

以下静态创建操作符接收调度器参数：

* bindCallback
* bindNodeCallback
* combineLatest
* concat
* empty
* from
* fromPromise
* interval
* merge
* of
* range
* throw
* timer