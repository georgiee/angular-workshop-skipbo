# RxJS
RxJS in its version 6 is a core part of the Angular framework so it's worth to learn at least some basics.

Great source to learn rxjs: https://reactive.how/


## Debugging
I see myslef using tap a lot.
You might find [rxjs-spy](https://github.com/cartant/rxjs-spy) useful.

[withLatestFrom vs combineLatest](https://medium.com/@martinkonicek/rx-combinelatest-vs-withlatestfrom-ccd98cc1cd41)

takeLast vs last & error `EmptyError: no elements in sequence`

toArray() to collect everything

```
 const keyPressed = (key: string) => fromEvent(window, 'keydown')
      .pipe(filter((event: KeyboardEvent) => event.key === key));
	
	keyPressed('r').pipe(first())
```

common shortcut with underscore if your are not interested in the value
```
subscribe(_ => {
  
})
```

static vs instance methods:
pipe, merge, interval,

mapTo(true)

## Dollar Signs
Have you encountered a dollar sign in source files using rxjs ? Angular has a [small description](https://angular.io/guide/rx-library) about it. Dollar signs help to separata Observables from values more easily.

Actually the dollar sign usually stands for pluralization as an observables streams multiple values. It's called the [Finnish notation](https://medium.com/@benlesh/observables-and-finnish-notation-df8356ed1c9b).

```
const click$ = Observable.fromEvent(button, 'click');
```

But it's no rule or a styleguide thing. I see myself not using it that strict.

## Startign a stream
fromEvent, of(array), of(...array)

## Cold vs Hot
I will cite Ben Lesh here from his excellent [article](https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339) on the topic.

A cold observable creates its producer on each susbcription, a hot observables closes of a and alreayd existing instance.


```
// COLD
var cold = new Observable((observer) => {
	var producer = new WebSocket();
	producer.listen(() => {
		observer.next()
	});
});

// HOT
var producer = new WebSocket();
var hot = new Observable((observer) => {
	producer.listen(() => {
		observer.next()
	});
});
```

A cold observable is therefore unicast, each subscription gets its own individual stream. A hot observable shares the stream between all subscriptions (multicast). It also creates values independent of the amount of subscriptions.

It's important to understand the difference as you might create subscriptions with operators like switchMap/concaptMap and you really don't want to create individual instances of let's say web sockets where you only wanted one.


## RxJs in practice
Here some things I use frequently so you get confident in using them.

### asObservable vs Subject
If you have a subject just for signaling something you usually use a Subject.

```
class YourComponent {
	changed: Subject<any> = new Subject();
	
	notify() {
		this.changed.next()
	}
}
```
The problem here: You expose the subject directly and you would allow any listener to forward a value with `yourComponentInstance.changed.next()`. That's a private action 
and should be implemented as such. That's where `subject.asObservable()` comes to help you.

```
private _changed: Subject<any> = new Subject();
get changed(): Observable<any> {
	return this._changed.asObservable();
}
```

This way you can protect it from forwarding values. 

> **Learning:** Have a private subject and only expose it through the asObservable() method.


### BehaviourSubject
That's a pretty important part of RxJs if you use subjects for some kind of state where you don't know when listeners are subscribing.

Imagine a service loading some data at the application startup and you have a router navigating through your page components. Your interested page might come to life quite some time the data arrived. 

If you subscribe to such a hot observable where the data might have been produced already you won't get any data when you subscribe too late.

```
it('BehaviorSubject', () => {
    const normalSbject: Subject<number> = new Subject();
    const behaviourSubject: BehaviorSubject<number> = new BehaviorSubject(0);

    const produceValue = value => {
      normalSbject.next(value);
      behaviourSubject.next(value);
    };

    produceValue(1);
    produceValue(5);
    produceValue(7);
    normalSbject.subscribe(value => {
      console.log('normalSbject: ', value);
    });

    behaviourSubject.subscribe(value => {
      console.log('normalSbject: ', value);
    });
  });
```

Your output is this:
> LOG: 'normalSbject: ', 7

and if another value is produced after subscription you get that output in addition.

```
produceValue(13);
```

> LOG: 'normalSbject: ', 13
> LOG: 'behaviourSubject: ', 13

> **Learning:** A BehaviorSubject ensures that you always retrieve the latest data even when you subscribe after the last value has been produced.

### destroy & takeUntil
You must ensure that you don't get dangling subscriptions like you don't want event listeners to be alive far after a component has been detroyed. That way you create unintended side effects and you create memory leaks as the garbage collector can't get rid of all involved resources.

It's easy to create a dangling subscription:

```
class YourComponent {
	initService() {
		this.yourService.subscribe(data => {
			// do something
		})
	}
}
```

When your `YourComponent` is destroyed you have a subscription still listening to your services. The idiomatic way of unsubscribing is using a reference to the subscription and removing it when you destroy the component.

```typescript
private _subscription: Subscription = Subscription.EMPTY;
initService() {
	this._subscription = this.yourService.subscribe();
}

ngOnDestroy() {
	this._subscription.unsubscribe();
}
  
```
But if you habe more than one subscription this gets tedious. You can use the power of rxjs to unsubscribe automatically. The only thing you need in addition is a signal when the component is destroyed.

```
private _destroyed: Subject<any> = new Subject();
ngOnDestroy() {
    this._destroyed.next();
}
```

With that at your hand you can automatically complete and unsubscribe any stream you create.
```
this.yourService
	.pipe(takeUntil(this._destroyed))
	.subscribe(data => {
		// do something
	})
```

`takeUntil` will complete the event.

Neat isn't it? There is another good [article](https://medium.com/@benlesh/rxjs-dont-unsubscribe-6753ed4fda87) from Ben Lesh about it.

### toArray 
collect all values and return them once the stream completes
```
range(1, 10)
    .pipe(
      toArray()
    ).subscribe(values => console.log(values));
```    
> (10) [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

### tap
to-go tool for debugging without interfering with the stream.

```typescript
fromEvent(document, 'click')
    .pipe(
      tap(_ => console.log('click occured'))
    ).subscribe();
```

> click occured
> click occured
> 

### merge
Merge can be used as an operator or standalone to begin a stream. You use it to merge multiple stream and dispatch them as one source. You can use it for example as an abort signal. Here whenever a click or a keydown event appears the signal would fire.

```typescript
const abortSignal = merge(
  fromEvent(document, 'click'),
  fromEvent(document, 'keydown')
);
```

If you now start some other stream (to load something or to run something automatically for the user) you can abort it:

```typescript
interval(1000)
    .pipe(
      takeUntil(abortSignal)
    ).subscribe(value => {
      console.log('new interval step');
    });
```

`interval` will dispatch a value every second and together with the takeUntil operator (that you already know from the onDestroy example) we can complete the stream whenever we receive an abort signal. That's really pretty code coimpared to the imperative version where you would have to maintain boolean flags, timeout referernces etc.

### flatMap (alias mergeMap) vs switchMap

Those operators are seen often but understanding them is not so easy when you are confronted with the whole RxJs universe.

The map suffix tells you that those operators (like any other `*Map` operator) will convert any observable produced to its raw value. That's important. We want to have inner observable to do stuff like loading.

Here to show the flattening. It's a made up example. We have a value of 1 `of(1)` and map it to an observable containing a single value. The only goal is here to get an output that let us know we received an Observable.
 
```
of(0)
.pipe(
  mapTo(_ => of(1))
)
.subscribe(value => {
  console.log(value);
});
```

> Observable {_isScalar: true, _subscribe: ƒ, value: 1}

You don't want this. Imagine you are running a http request instead of the `of(1)`. So you need something that is returning the value of the observable — that's called flattening and rxjs has us covered with operators like `switchMap`, `flatMap/mergeMap`, `concatMap` & `exhaustMap`.

```
of(0)
.pipe(
  flatMap(_ => of(1))
)
.subscribe(value => {
  console.log(value);
});
```

This will correctly output the value of the observable.
> 1

With this understood let me show you whats the difference between flatMap and switchMap. Both can flatten a value but the difference is what's happening with the inner observable (here `of(1)`) when a new value arrives in the outer observable. The current conceived examples don't helps here because there is only one value and not a stream of values.

```
interval(1000)
    .pipe(
      take(3),
      mergeMap(_ => {
        // simulate fetching http data
        return of(1)
          .pipe(
            delay(2000), // and that it takes 500ms
            tap(value => console.log('received new data'))
          );
      })
    ).subscribe(value => {
      console.log('new polled data arrived');
    });
```
What happens here is that the interval will stream every second three times (take(3)). mergeMap creates another observable (of, delay, tap) and resolves it after only two seconds.

Your output with mergeMap will look like this:
> received new data
> new polled data arrived
> received new data
> new polled data arrived
> received new data
> new polled data arrived

Whereas if you use `switchMap` you output looks so:
> received new data
> new polled data arrived

That's because switchMap will always unsubscribe from the inner observable when another value arrives. The inner observable 

```
of(1).pipe(
	delay(2000), // and that it takes 500ms
	tap(value => console.log('received new data'))
);
```

is canceled by `switchMap` where `mergeMap` simply enqueues every observable, wait for a it's completion and still streams the value.

Depending of your use case you want the either or the other and you must understand what you are using. Otherwise you would receive oudated data (if using mergeMap with http for exeample).

## forkJoin

## Testing
RxJS is can easily be tested if no timers or asynchronious sources (like http, user input) are mocked. Even if just stick to the rules for any test, if there is asynchronity use fakeAsync & test or async if necessary.

You can therefore test rxjs usually with the two following methods.

### Subscribe directly to your observable

```
it('tests rxjs', () => {
    let result = false;

    component.changes.subscribe(value => {
      result = value;
    });

    // this will cause a true value to be streamed
    component.triggerSomeChanges();

    expect(result).toBeTruthy();
  });
```

### Or just monitor for expected changes (css/markup)
That's even better in a component because you don't want to test implementation details but the things being exposed.

For example, if you have a component that is incrementing a counter whenever a click occurs. You know that you are using rxjs to do it but in the end it only matters that the changes are reflected in the template. So just test the counter not the rxjs.

```
it('should increment', () => {
    fixture = TestBed.createComponent(BaseTest);

    const nativeElement: HTMLElement = fixture.nativeElement.querySelector('.counter');

    expect(nativeElement.textContent.trim()).toBe('0');

    nativeElement.click();
    fixture.detectChanges();

    expect(nativeElement.textContent.trim()).toBe('1');
  });
```


### Marble Testing
You might have seen [marble test](https://github.com/ReactiveX/rxjs/blob/de9d2e4cef759b358c6c5d53b85e337dbf1c9c45/doc/marble-testing.md) like those:

```
it('generate the stream correctly', () => {
  scheduler.run(helpers => {
    const { cold, expectObservable, expectSubscriptions } = helpers;
    const e1 =  cold('-a--b--c---|');
    const subs =     '^----------!';
    const expected = '-a-----c---|';

    expectObservable(e1.pipe(throttleTime(3, scheduler))).toBe(expected);
    expectSubscriptions(e1.subscriptions).toBe(subs);
  });
```
That testing is useful when creating your own observables and is probably too much of testing for your own rxjs code.