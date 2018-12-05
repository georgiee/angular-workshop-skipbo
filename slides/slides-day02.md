build-lists: true
![](../images/slides/intro.png)

^ Angular Workshop

^ Skip-Bo Edition

---
![original](../images/slides/bg.png)
# [fit] RECAP

---
# _Day 1_

+ Chapter I — Modules
+ Chapter II — Components
+ Chapter III — Routing

---
![original](../images/slides/bg.png)
# [fit] DAY 2

---
# _Schedule_
+ Chapter IV — RxJS
+ Chapter V — Testing
+ Chapter VI — Animation

---
![original](../images/slides/bg.png)
#### Chapter IV
# [fit] RXJS

---
# [fit] THEORY

---
+ Introduction
+ Debugging
+ About Dollar Signs
+ Cold vs Hot Observables
+ Make Cold Observables Hot
+ RxJS in the wild
+ Testing

---
# Introduction

+ Extended Observer Pattern (Gang of Four)
+ Subject and Observers
+ Event System is an Observer Pattern

^ Extended Observer Pattern: Completion & Error Signaling, nothing more

^ http://reactivex.io/documentation/operators.html

[.footer: Chapter 04 — RxJS: Introduction]

---
![](../images/slides/rxjs-quadrants.jpg)

---
[.code-highlight: none]
[.code-highlight: 1]
[.code-highlight: 2]
[.code-highlight: 3-4]
[.code-highlight: 5]
[.code-highlight: all]

```typescript
of(1, 2, 3, 4, 5, 6)
  .pipe(
    take(3),
    filter(value => value%2 === 0),
  ).subscribe()
);
```

^ stream of data (could also be a http request, a stream of mouse data, interval )

^ pipe starts the plumbing of operators

^ operators for many things: Transform (Map, Scan), Filter (Distinct, Debounce, First, Last, Take), Combining (merge, zip), condition (takeWhile, takeUntil, SkipWhile), Math etc.

^ http://reactivex.io/documentation/operators.html

[.footer: Chapter 04 — RxJS: Introduction]

---
# Debugging

## _Tap_

[.code-highlight: 3]
```typescript
fromEvent(window, 'keydown')
    .pipe(
      tap(event => console.log('key pressed'))
  ).subscribe();
```

[.footer: Chapter 04 — RxJS: Debugging]

---
## _RxJS Spy (Tool)_

[.code-highlight: 5]

```typescript
import { tag } from 'rxjs-spy/operators/tag';

fromEvent(window, 'keydown')
  .pipe(
    tag('🎹 Key'),
).subscribe();
```

^ RxJS Spy can hook into any RxJS stream by placing a pipeable `tag`.

[.footer: Chapter 04 — RxJS: Debugging]

---
```bash
Tag = 🎹 Key; notification = subscribe<br>

Tag = 🎹 Key; notification = next; value = {key: "a"…}
Tag = 🎹 Key; notification = next; value = {key: "b"…}

🎹 Key  notification = unsubscribe<br>
```

^ you are perfectly informed

^ subscriptions, unsubscribes and of course next, errors and complete.

[.footer: Chapter 04 — RxJS: Debugging]

---
# Dollar Sign

```typescript
const click$ = Observable.fromEvent(button, 'click');
```
+ pluralization, called Finnish notation
+ peopl€, mic€, oxe₦

^ via Ben Lesh & André Staltz
^ vs Hungarian `const sTest = "test";`

---
# Cold vs Hot Observables

> A cold observable creates its producer on each subscription, a hot observables closes over an already existing instance.
-- Ben Lesh

[.footer: Chapter 04 — RxJS: Cold vs Hot]

^ Netffix, Google, RxJS Lead Developer

---


```typescript
// COLD (unicast)
var cold = new Observable((observer) => {
  var producer = new Producer();
  producer.listen(() => {
    observer.next()
  });
});

// HOT (multicast)
var producer = new Producer();
var hot = new Observable((observer) => {
  producer.listen(() => {
    observer.next()
  });
});
```

^ unicast vs multicast

^ Important to understand, create to many resources (http, websocket)

^ Cold Observables only start producing data with a subscription, each subscription gets own stream of values (unicast)

^ unicast/cold: `timer()` and `interval()` observables are cold.

^ Hot Observables are streaming values regardless of the amount of subscribers and each one gets the same data (multicast)

^ `fromEvent()` produces data without a subscription.

[.footer: Chapter 04 — RxJS: Cold vs Hot]

---
## _Make Cold Observable Hot_

+ Cold: Create a producer (like a websocket) for each subscriber

+ Make Hot: Create only one producer, then send same data to all susbcribers

[.footer: Chapter 04 — RxJS: Make Cold Observable Hot]
---

```typescript
const myInterval = interval(500).pipe(
  tap(value => console.log('interval produced a value'))
);

myInterval.subscribe(value => {
  console.log('received a value', value)
});
myInterval.subscribe(value => {
  console.log('received a value', value)
});

/**
  interval produced a value
  received a value 0
  interval produced a value
  received a value 0
*/
```
[.footer: Chapter 04 — RxJS: Make Cold Observable Hot]

---
```typescript
const myInterval = interval(500).pipe(
  tap(value => console.log('interval produced a value'))
);

const subject = new Subject();
// 1. let this subject subscribe to the cold observable
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

[.footer: Chapter 04 — RxJS: Make Cold Observable Hot]

---

## _RxJS in the wild_
+ asObservable vs. Subject
+ BehaviourSubject
+ destroy & takeUntil
+ toArray

---


```typescript
private _changed: Subject<any> = new Subject();

get changed(): Observable<any> {
  return this._changed.asObservable();
}
```
+ A subject is both an observer and observable
+ Prevent the observer part (next)
+ ~~changed.next('new value')~~

^ Prevents

[.footer: Chapter 04 — RxJS: RxJS in the wild]

---
+ Hot Observables can produce values without someone listening.
+ Page mounted vs Data already delivered😢
+ `BehaviorSubject` is the solution

^ It's pretty sad to imagine someone telling important stuff and nobody is listening.

[.footer: Chapter 04 — RxJS: RxJS in the wild]

---

[.code-highlight: 9]
[.code-highlight: 7, 8]
[.code-highlight: 1, 2]
[.code-highlight: 4, 5]

```typescript
const subjectA = new Subject();
const subjectB = new BehaviorSubject(null);

subjectA.next('your loaded data');
subjectB.next('your loaded data');

subjectA.subscribe(value => console.log('value from subjectA:', value));
subjectB.subscribe(value => console.log('value from subjectB:', value));
// value from subjectB: your loaded data
```
^ BehaviorSubject as the solution

^ Create the Observer

^ Forward some data (imagine: before the page components is alive)

^ Subscribe for data (imagine: page components is alive, subscribe)

^ One receiver is now really sad as no data will arrive

[.footer: Chapter 04 — RxJS: RxJS in the wild]

---

+ addEvenListener -> removeEventListener
+ subscribe -> unsubscribe
+ This is bad

[.code-highlight: 3]

```typescript
class YourComponent {
  initService() {
    this.yourService.subscribe(data => {
      // do something nice
    })
  }
}
```
^ like events streams net to be unsubscribed

^ if you don't unsubscribe, the stream could receive data when the component is gone already.

[.footer: Chapter 04 — RxJS: RxJS in the wild]

---

[.code-highlight: 1, 3, 7]

```typescript
private _subscription: Subscription = Subscription.EMPTY;
initService() {
  this._subscription = this.yourService.subscribe();
}

ngOnDestroy() {
  this._subscription.unsubscribe();
}

```

^ That's better we are unsubscribing onDestroy

[.footer: Chapter 04 — RxJS: RxJS in the wild]

---

[.code-highlight: 1, 6, 13]

```typescript
private _destroyed: Subject<any> = new Subject();

initService() {
  this.yourService
    .pipe(
      takeUntil(this._destroyed)
    ).subscribe(data => {
      // do something nice
  })
}

ngOnDestroy() {
  this._destroyed.next();
}

```

^ Think reactive

^ Yoy can automatically complete and unsubscribe any stream you create with a signal

[.footer: Chapter 04 — RxJS: RxJS in the wild]

---

## _RxJS Testing_
+ RxJS is basically synchronous 🙌
+ Test for effects, don't test the stream itself.
+ Forward time with tick & fakeAsync
+ Never use Marble Testing to test streams
+ (Example `rxjs/testing`)

^ Synchronous makes Testing easy

^ Async when using async observables (interval, timer, http)

^ Effects: Test if a value arrives or template changes

^ Marble Testing for custom Observables only

---
# [fit] CHALLENGE

---
## _Your tasks_

+ Redirect to the Gameover Page
+ AI 🐙 Autoplay V1
+ AI 🐙 Autoplay V2
+ AI 🐙 Autoplay V3
+ Stop the AI after game is over

---
# [fit] RESULT

---

![fit](../images/abc.png)

^ You can now play against the CPU players!

---
![original](../images/slides/bg.png)

#### Chapter V
# [fit] TESTING

---
# [fit] THEORY

---

+ Setup
+ Component Testing
+ Micro & Macro Tasks (Theory)
+ Testing Async Code
+ Change Detection
+ Testing Routing

---
# _Setup_
+ Different Reporter (mocha vs progress (default))
+ Headless (no browser window)
+ Firefox als works Headless 👌
+ No, not IE.

![left 150%](../images/slides/mocha.png)

^ I prepared your Workshop project already with a mocha reporter and a headless configuration

^ mocha vs progress, what's better 😎

---
# _Component Testing_

+ Angular CLI default is not good


```typescript
beforeEach(() => {
  fixture = TestBed.createComponent(SomeComponent);
  component = fixture.componentInstance;
  fixture.detectChanges();
});

it('should create', () => {
  expect(component).toBeTruthy();
});
```

^ No bindings, no different configurations/scenarios

---
[.code-highlight: none]
[.code-highlight: 12-16]
[.code-highlight: 1-3]
[.code-highlight: 5-10]

```typescript
TestBed.configureTestingModule({
  declarations: [ FooComponent, TestSomeComponent ]
})

it('should create v2', () => {
  const myFixture=TestBed.createComponent(TestSomeComponent);
  myFixture.componentInstance.helperVariable = 345;
  fixture.detectChanges();
  expect(myFixture.componentInstance.myComponent).toBeTruthy();
});

@Component({ template: `<app-some [myInput]="helperVariable"></app-some>` })
class TestSomeComponent {
  public helperVariable = 123;
  @ViewChild(SomeComponent) myComponent: SomeComponent;
}

```

^ 1. Create a small Host Components inside your test, use your component tag and use ViewChild to query for it so your host component can give access to your test.

^ 2. Add the Host Component also to the declaration array

^ 3. Use  it. Makes it so easy to test different input variations (myInput helperVariable)

[.footer: Chapter 04 — RxJS: Component Testing]

---
# _Micro & Macro Tasks_

> What's the output ? 🧐

```typescript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```

^ Question is from Jake Archibald (developer advocate for Google Chrome)

[.footer: Chapter 04 — RxJS: Micro & Macro Tasks]


---
[.code-highlight: 1-13]
[.code-highlight: 1, 16]
[.code-highlight: 3-5]
[.code-highlight: 7-12]
[.code-highlight: 16,17]
[.code-highlight: 7,8,16-18]
[.code-highlight: 9,10, 16-19]
[.code-highlight: 4, 20]

```typescript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');

/*
script start
script end
promise1
promise2
setTimeout
*/

```
^ 1. current script is a task being processed

^ 2. process first console.log

^ 3. queue `setTimeout` as a (macro) task

^ 4. queue first  `Promise callback` as a Microtask

^ 5. script task is done, next up is the microtask queue

^ 6. process promise callback, queue next callback as microtask

^ 7. microtask queue is processed immediately

^ 8. when no microtasks are left, process next macro task

^ Awesome Source: https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/

[.footer: Chapter 04 — RxJS: Micro & Macro Tasks]

---
What are Macro Tasks?
> Queued up but allow the browser engine to render between each task.

+ scripts
+ setTimeout / setInterval
+ event listener callbacks

```typescript
document.appendChild(el);
el.style.display = 'none';
```

^ You will never see this flash as the full script is executed as a single task

[.footer: Chapter 04 — RxJS: Micro & Macro Tasks]

---
What are Microtasks?
They are queued up and executed at the end of a task. No browser action in between.

+ MutationObserver callback (DOM changes)
+ Promises (even settled ones)

^ Microtasks don't allow to breath, they can queue up endlessly if not used correctly.

[.footer: Chapter 04 — RxJS: Micro & Macro Tasks]

---

```javascript
// endless (macro) tasks queue - is useless but okay 👌
function cb() {
	console.log('cb');
  setTimeout(cb, 0)
}

cb();
```

```javascript

// ⚠️ ⚠️ ⚠️  This will hang your browser
// — save everything then try 🙊
function cb() {
  console.log('cb');
  Promise.resolve().then(cb);
}

cb();

```

[.footer: Chapter 04 — RxJS: Micro & Macro Tasks]

---
# _Testing Async Code_

+ What is async ?
+ ngZone Primer
+ fakeAsync & tick + flush
+ async & fixture.whenStable
+ done (jasmine)

[.footer: Chapter 04 — RxJS: Testing Async Code]

---
What is async?

+ User does something
+ Time passes
+ Nothing else can change your application

^ Small list: You can get full control over it.

---
Ever wondered what ngZone is ? Here a ngZone Primer

+ Zone.setInterval()
+ Zone.alert()
+ Zone.prompt()
+ Zone.requestAnimationFrame()
+ Zone.addEventListener()
+ Zone.removeEventListener()

^ monkey patches all those functions

^ Angular has now **complete** control over all macro und micro tasks.

^ Change Detection whenever something here happens

^ With full control over micro and macro task means: we can easily test

[.footer: Chapter 04 — RxJS: Testing Async Code]

---

```typescript
it('setTimeout & tick & flushMicrotasks ', fakeAsync(() => {
  let state = [];

  Promise.resolve().then(function() {
    state.push('promise result');
  });

  setTimeout(() => { state.push('timeout called'); });
  setTimeout(() => { state.push('timeout called after 2s'); }, 2000);

  expect(state).toEqual([]);

  flushMicrotasks();
  expect(state).toEqual(['promise result']);

  tick();
  expect(state).toEqual(['promise result', 'timeout called']);

  tick(2000);
  expect(state).toEqual(['promise result', 'timeout called', 'timeout called after 2s']);
}));
```
^ see that? we are the master of all micro and macro tasks 💪

[.footer: Chapter 04 — RxJS: Testing Async Code]

---

```typescript
it('setTimeout(0) & tick ', fakeAsync(() => {
  let state = [];

  setTimeout(() => { state.push('timeout called'); });
  setTimeout(() => { state.push('timeout called after 2s'); }, 2000);

  expect(state).toEqual([]);

  // tick wont' work -> Error: 1 timer(s) still in the queue.
  // tick();
  flush();
  expect(state).toEqual(['timeout called', 'timeout called after 2s']);
}));
```

^ `flush()` to ignore time and drain all queues
[.footer: Chapter 04 — RxJS: Testing Async Code]

---
+ async & fixture.whenStable
+ done

```typescript
it('manually finish your spec', (done) => {
  console.log('run');
  expect(true).toBe(true);
  done();
});
```

^ real time to pass

^ done is from jasmine

[.footer: Chapter 04 — RxJS: Testing Async Code]

---

> If you expect changes in your template call<br>
> 👉 `fixture.detectChanges()`


[.footer: Chapter 04 — RxJS: Testing Async Code]

^ In tests we have no zone running, no zone automagic

^ So there is only one rule

^ ComponentFixtureAutoDetect Provider but use the moment to reflect about your component, don't use magic.

---
Testing Routing


[.code-highlight: 4]
[.code-highlight: 16]
[.code-highlight: 17]
[.code-highlight: 18]

```typescript
beforeEach(async(() => {
  TestBed.configureTestingModule({
    imports: [
      RouterTestingModule.withRoutes(routes),/*...*/
    ],
    declarations: [/*...*/],
  }).compileComponents();

  router = TestBed.get(Router);
  location = TestBed.get(Location);
  fixture = TestBed.createComponent(AppComponent);
}));

describe('application routing', () => {
  it('navigate to "" redirects you to /welcome', fakeAsync(() => {
    fixture.ngZone.run(() => router.navigate(['']));
    tick();
    expect(location.path()).toBe('/welcome');
  }));
});
```

^ 1. Use the RouterTestingModule

^ 2. Navigate as usual

^ 3. Tick to drain micro and macro task queue (router is async)

^ 4. check the current location for success

---

# [fit] CHALLENGE

---
## _Your tasks_
+ Stock Bug (Investigate) 🐛
+ Stock Bug — Part 1, 2, 3
+ Test RxJS w/ Oscar 🐙 — CPUs
+ Test RxJS w/ Oscar 🐙 — Humans
+ Can Oscar play multiple cards ?

---
# [fit] RESULT

---
![fit](../images/challenge-05-end.jpg)

^ Well tested and we can see the opponent players finally!

---
![original](../images/slides/bg.png)

#### Chapter VI
# [fit] ANIMATION

---
# [fit] THEORY

---
+ Animation Basics
+ Appear & Disappear
+ Numeric Triggers
+ Disable
+ Router Animations
+ Animate Children


---
# [fit] CHALLENGE

---
## _Your tasks_
+ First Flip - Part 1 & 2
+ Flip Party
+ Flip with Style
+ Make the Hand Cards flip
+ Animate Stock Flip

---
# [fit] RESULT

---
![fit](../images/animation-end.gif)

---
![original](../images/slides/bg.png)
### [fit] END 2
---
![original](../images/slides/bg.png)
### [fit] THANKS
