# Theory: RxJS

## Introduction
RxJS in its version 6 is a core part of the Angular framework so it's worth to learn it. This being said it's hard to learn — this challenge will teach you some parts of it to bring you one step nearer to understand RxJS.

We will look at the following things in RxJS:

+ RxJS 1: Debugging
+ RxJS 2: About Dollar Signs
+ RxJS 3: Cold vs Hot Observables
+ RxJS 4: Make Cold Observables Hot
+ RxJS 5: Into the Wild — asObservable vs Subject
+ RxJS 6: Into the Wild — BehaviourSubject
+ RxJS 7: Into the Wild — destroy & takeUntil
+ RxJS 8: Into the Wild — toArray
+ RxJS 9: Testing

## RxJS 1: Debugging
Branch `rxjs/debug`

### Tap
I see myself using tap a lot to debug rxjs code. It's an operator that can look at what data is passing by without touching it.

```typescript
fromEvent(window, 'keydown')
    .pipe(
      tap(event => console.log('key pressed'))
  ).subscribe();
```

You can use tap to debug or to create side effects like calling another method.

### RxJS Spy
You might find [rxjs-spy](https://github.com/cartant/rxjs-spy) useful.

RxJS Spy can hook into any RxJS stream by placing a pipeable `tag`. It won't interact with the stream— just like the rxjs operator `tap`. After the tag is registered you will get infos about everything that's happening with the Observable like subscriptions, unsubscribes and of course next, errors and complete.


```typescript
import { tag } from 'rxjs-spy/operators/tag';
import { create } from 'rxjs-spy';

const spy = create();
spy.log();

fromEvent(window, 'keydown')
    .pipe(
      tag('🎹 Key Pressed'),
      take(2)
  ).subscribe();
```

Your console log will look like this after pressing key A and B

> Tag = 🎹 Key Pressed; notification = subscribe; matching /.+/<br>
> Tag = 🎹 Key Pressed; notification = next; value = KeyboardEvent {key: "a", code: "KeyA" …}<br>
> Tag = 🎹 Key Pressed; notification = next; value = KeyboardEvent {key: "b", code: "KeyB" …}<br>
> 🎹 Key Pressed; notification = unsubscribe; matching /.+/<br>

This means you are perfectly informed about what happens in your stream and what it contains. I love this way of debugging a rxjs stream.

## RxJS 2: About Dollar Signs
Have you encountered a dollar sign in source files using rxjs ? Angular has a [small description](https://angular.io/guide/rx-library) about it. Dollar signs help to separate Observables from values more easily.

Actually the dollar sign usually stands for pluralization as an observables streams multiple values. It's called the [Finnish notation](https://medium.com/@benlesh/observables-and-finnish-notation-df8356ed1c9b).

```
const click$ = Observable.fromEvent(button, 'click');
```

But it's no rule or a styleguide thing. I see myself not using it that strict.

## RxJS 3: Cold vs Hot Observables
Branch `rxjs/cold-hot`

I will cite Ben Lesh here from his excellent [article](https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339) on the topic.

> A cold observable creates its producer on each subscription, a hot observables closes over an already existing instance.

This means:
> Cold Observables only start producing data with a subscription present and each subscription get its own stream of values. That's called **unicast**.
Example: `timer()` and `interval()` observables are cold.

> Hot Observables are streaming values regardless of the amount of subscribers and each one gets the same data. That's **multicast**.
`fromEvent()` produces data without a subscription.

Here another brief example.

```typescript
// COLD
var cold = new Observable((observer) => {
  var producer = new Producer();
  producer.listen(() => {
    observer.next()
  });
});

// HOT
var producer = new Producer();
var hot = new Observable((observer) => {
  producer.listen(() => {
    observer.next()
  });
});
```

It's important to understand the difference otherwise you create resource intensive producers with each new subscription. You also need the knowledge about hot & cold when dealing with operators like `switchMap` and `mergeMap` where the difference is when the inner observable is unsubscribed.

## RxJS 4: Make Cold Observables Hot
Can we make a cold observable hot? Can we make it multicast despites it's only producing values for a single observer ?
Yes! It's our good old friend Subject that is tailored for this purpose.

Let's get a little mit more concrete, we use an interval — which is cold by default.

```typescript
const myInterval = interval(500).pipe(
  take(5),
  tap(value => console.log('interval produced a value'))
);
```

You would create a new `setIterval` with each subscription and each susbcription would receive its own stream of numbers.

```typescript
myInterval.subscribe(value => console.log('received a value', value));
myInterval.subscribe(value => console.log('received a value', value));
myInterval.subscribe(value => console.log('received a value', value));

 /**
  interval produced a value
  received a value 0
  interval produced a value
  received a value 0
  interval produced a value
  received a value 0
  interval produced a value
*/
```

The 0 look similar but they are produced from different intervals. We can make it multicast with a Subject. The Subject will be some kind of mediator. Receiving the value and distribute to everyone who is interested. That's multicast.

```typescript
const subject = new Subject();
  // 1. let this subject susbcribe to the cold observable
  myInterval.subscribe(subject);

  // 2. now let future observables subscribe to the subject instead of the interval
  subject.subscribe(value => console.log('received a value', value));
  subject.subscribe(value => console.log('received a value', value));
  subject.subscribe(value => console.log('received a value', value));

  /**
  interval produced a value
  received a value 0
  received a value 0
  received a value 0
  */
```

Without changing the implementation of the given observable we were able to transform it into a multicasting stream.
That's a powerful technique 💪

There are some multicast operators in RxJS like publish, publishReplay, multicast & share, they all do the same thing with a Subject under the hood.

<details>
<summary>multicast operator</summary>
You could have made your life a little bit easier with the `multicast` operator.

```typescript
const myHotObservable = interval(500).pipe(
  take(5),
  tap(value => console.log('interval produced a value')),
  multicast(new Subject()) // make it a ConnectableObservable
) as ConnectableObservable<number>;

// all those subscribes will be delegated to the internal Subject by the multicast operator.
myInterval2.subscribe(value => console.log('received a value', value));
myInterval2.subscribe(value => console.log('received a value', value));
myInterval2.subscribe(value => console.log('received a value', value));

// let the internal subject subscribe to the given interval — no values are produced before this
myInterval2.connect();
```

</details>

## Into the wild
Following some things I use frequently in real projects so you get confident in using them too 🙌

## RxJS 5: Into the Wild — asObservable vs Subject
Branch `rxjs/into-the-wild-as-observable`

If you have a subject just for signaling something you usually use a Subject and expose it, right?

```typescript
class YourComponent {
  changed$: Subject<any> = new Subject();

  notify() {
    this.changed$.next()
  }
}
```

The problem here: A subject is both a observer and observable. It can observe by subscribing (otherObservable.subscribe(subject)) and you can subscribe to it as an Observable (subject.subscribe). By exposing a subject directly you allow other parties to use its observer side. Someone could get the idea to subscribe it to some source or just use the `next` method (as part of the observer pattern). You want to protect your subject from being used like this.

That's where `subject.asObservable()` comes to your rescue.

```typescript
private _changed: Subject<any> = new Subject();

get changed(): Observable<any> {
  return this._changed.asObservable();
}
```

This way it's only an observable — there are no observer functions like next available.

## RxJS 6: Into the Wild — BehaviourSubject
Branch `rxjs/into-the-wild-behaviour`

Hot Observables can produce values without someone listening. It's pretty sad to imagine someone telling important stuff and nobody is listening.
This is often a real problem in your application. Imagine a service loading some data at the application startup and you have a router navigating through your page components. Your interested page might come to life quite some time the data arrived. If you subscribe such an observable where the data might have been produced already you won't get any data when you subscribe too late.

RxJS has a special Subject that will help you. A [BehaviorSubject](http://reactivex.io/rxjs/manual/overview.html).

```typescript
const subjectA = new Subject();
const subjectB = new BehaviorSubject(null);

subjectA.next('your loaded data');
subjectB.next('your loaded data');

subjectA.subscribe(value => console.log('value from subjectA:', value));
subjectB.subscribe(value => console.log('value from subjectB:', value));
```

Your output is this:<br>
> value from subjectB: your loaded data

The subscription to the normal subject just missed the data — it was produced too early. The `BehaviorSubject` also
produced the value before the subscription — but it's nice enough to tell every subscription about the last subscription.

A `BehaviorSubject` only retains the last single value. If you want to keep all values you could use the `ReplaySubject`.

## RxJS 7: Into the Wild — destroy & takeUntil
Branch `rxjs/into-the-wild-take-destroy`.

That's a pretty nice pattern to learn.

When you use `addEvenListener` you must ensure that you call `removeEventListener` too at some moment. The same is true of subscriptions,
that's the return value when you subscribe to an Observable. Otherwise you would create unintended side effects and you create memory leaks as the garbage collector can't get rid of all involved resources.

It's easy to create a dangling subscription:

```typescript
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

What happens if you have more than one subscription? Yes this gets tedious. You can use the power of rxjs to unsubscribe automatically. The only thing you need in addition is a signal when the component is destroyed.

```typescript
private _destroyed: Subject<any> = new Subject();
ngOnDestroy() {
  this._destroyed.next();
}
```

With that at your hand you can automatically complete and unsubscribe any stream you create.

```typescript
this.yourService
  .pipe(takeUntil(this._destroyed))
  .subscribe(data => {
    // do something
  })
```

`takeUntil` will complete the event once the desotry signal arrives.

Neat isn't it? There is another good [article](https://medium.com/@benlesh/rxjs-dont-unsubscribe-6753ed4fda87) from Ben Lesh about it.

## RxJS 8: Into the Wild — toArray

You can collect all values from a stream with `toArray` and return them once the stream completes.
The same thing is happening when you use `last()` but you only return the last received element.

That's a pretty handy operator I discovered far too late. You will see it in action in one of the challenges.
There I use it to block a stream until it reached a given result.

Here a simple example:

```typescript
range(1, 10)
    .pipe(
      toArray()
    ).subscribe(values => console.log(values));
```
> (10) [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]


And one with `switchMap` involved to show you the blocking functionality.

```typescript
timer(0, 5000)
.pipe(
  tag('outer observable'),
  // create an inner observable
  switchMap(value => {
    return interval(500)
      .pipe(
        take(2),
        tag('inner observable'),
        toArray() // block until it completes. It will collect the array of collected values: [0, 1]
      );
  }),
  // here you will see only one value (always [0, 1]) arriving from the
  // inner observable.
  tag('after the switch'),
).subscribe();
```

## RxJS 9: Testing
Branch `rxjs/testing`.

RxJS is synchronous by default. This means testing of your streams can be easy! Just subscribe and watch for the correct data to arrive.

If async things are involved you need to make sure that things like services are mocked (and act synchronous again)
or if there is a timer or interval involved just use the power of `fakeAsync` & `tick`. We will take a look at those in the next challenge about Testing.

When you create your own observables things get more complicated and you should do marble testing (see Variant C)

But in most cases your RxJS code can be tested with the following two methods (Variant A & B). It depends on the complexity of your stream and how exposed it is.
There will be some tests to implement in the testing challenge coming soon 🙌

You have this component given:

```typescript
export class AppComponent {
  title = 'workshop-theory';
  counter = 0;
  change$ = new Subject();

  constructor() {
    this.change$.subscribe(value => {
      this.counter++;
    });
  }

  doSomething() {
    this.change$.next(true);
  }
}
```

### Variant A: Subscribe directly to your observable
If the stream is only used internally it's perfectly fine to test the effects only. Imagine a component like this.

```typescript
it('changes should be true', () => {
  const fixture = TestBed.createComponent(AppTestComponent);
  fixture.detectChanges();

  const app: AppComponent = fixture.componentInstance.instance;

  let result = false;
  app.change$.subscribe( value => {
    result = value;
  });

  app.doSomething();

  expect(result).toBe(true);
});
```

### Variant B: Monitor for expected changes only
That's even better in a component because you don't want to test implementation details but the things being exposed.

For example, if you have a component that is incrementing a counter whenever a click occurs. You know that you are using rxjs to do it but in the end it only matters that the changes are reflected in the template. So just test the counter not the rxjs.


```typescript
it('counter should increment when clicked', () => {
  const fixture = TestBed.createComponent(AppTestComponent);
  fixture.detectChanges();

  const app: AppComponent = fixture.componentInstance.instance;

  const nativeElementCounter: HTMLElement = fixture.nativeElement.querySelector('.counter');
  expect(nativeElementCounter.textContent.trim()).toBe('0');

  app.doSomething();
  fixture.detectChanges();

  // rxjs implementation is not important just test the effect
  expect(nativeElementCounter.textContent.trim()).toBe('1');
});
```


### Variant C: Marble Testing
You might have seen [marble test](https://github.com/ReactiveX/rxjs/blob/de9d2e4cef759b358c6c5d53b85e337dbf1c9c45/doc/marble-testing.md) like those:

```typescript
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

That testing is useful when creating your own observables but in most cases far too much if you are only testing your streams.


## Completed
Finish 🙌  You gained knowledged about the following awesome things in RxJS and you are ready for the challenge.

+ RxJS 1: Debugging
+ RxJS 2: About Dollar Signs
+ RxJS 3: Cold vs Hot Observables
+ RxJS 4: Make Cold Observables Hot
+ RxJS 5: Into the Wild — asObservable vs Subject
+ RxJS 6: Into the Wild — BehaviourSubject
+ RxJS 7: Into the Wild — destroy & takeUntil
+ RxJS 8: Into the Wild — toArray
+ RxJS 9: Testing


## Challenge
Let's continue with [Chapter 04 - RxJS (Challenge)](../challenges/04-rxjs.md)

## Resources
+ Great source to learn rxjs: https://reactive.how/
+ [RxJS: Understanding the publish and share Operators](https://blog.angularindepth.com/rxjs-understanding-the-publish-and-share-operators-16ea2f446635)
+ [RxJS: How to Use refCount](https://blog.angularindepth.com/rxjs-how-to-use-refcount-73a0c6619a4e)
+ [On The Subject Of Subjects (in RxJS)](https://medium.com/@benlesh/on-the-subject-of-subjects-in-rxjs-2b08b7198b93)
+ [RxJS Operators for Dummies](https://scotch.io/tutorials/rxjs-operators-for-dummies-forkjoin-zip-combinelatest-withlatestfrom)
