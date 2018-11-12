# Testing

Testing in Angular is done with karma as the test runner (serving your tests in the browser) and jasmine as the testing framework (giving you the `describes` and `its`). There is a nice [thesis](https://github.com/karma-runner/karma/blob/master/thesis.pdf) from the maker of karma if you want to dig deep and understand the differences. 

Angular provides a working set of defaults but there are things you want to change.

## Preparations
### Different Ouput
I use [mocha reporter](https://www.npmjs.com/package/karma-mocha-reporter) in every project. Looks not only better but you get reminded about your current tests as they are printed in your terminal.

progress (Only numbers)
dots (Show dots)
![](images/test-output-progress.png)
![](images/test-output-mocha.png)
![](images/test-ouput-dots.png)

```bash
npm install karma-mocha-reporter -D
```
in your karma config

```json
plugins: [
  require('karma-mocha-reporter'),
],
reporters: ['mocha', 'kjhtml'],
```

### Go Headless
While you have the `karma.config` open let's switch to Chrome Headless so no browser window opens when we are testing. You can throw in the default `[Chrome]` in case you want to run them in the browser.

```json
browsers: ['ChromeHeadless'],

customLaunchers: {
  ChromeHeadless: {
    base: 'Chrome',
    flags: [
      '--headless',
      '--disable-gpu',
      '--remote-debugging-port=9222'
    ]
  }
}
```

This is also working with Firefox Headless (in any cases you will need karma-firefox-launcher)

```json
FirefoxHeadless: {
	base: 'Firefox',
	flags: [ '-headless' ]
}
```

## Component Testing

Angular CLI gives you a spec file for each component you create. This is a good beginning but testing looks different in the real world usually.

The first problem: You don't use `TestBed.createComponent` to bootstrap your component directly but you create one or multiple individual test components.

Angulars gives you idea by bootstrapping this app spec.

```typescript
it('should create the app', () => {
	const fixture = TestBed.createComponent(AppComponent);
	const app = fixture.debugElement.componentInstance;
	expect(app).toBeTruthy();
});
```
You directly create an instance of your component with `TestBed.createComponent(AppComponent);`. You don't want this. In your project you rather want to test it like this. Create a new component inside the test (your test wrapper)

```typescript
// 1. Create a test wrapper holding your component.
// There are usually multiple test components in a single spec file
// And all of them are collected at the bottom of your spec file.
// It's here to easily talk about.
// Notice the @ViewChild to query for your actual component.

@Component({
  template: `<app-foo></app-foo>`
})
class TestFooComponent {
  @ViewChild(FooComponent) fooInstance: FooComponent;
}

// 2. Add the created test wrapper (TestFooComponent)
// to your testbed so you can instantiate it.
TestBed.configureTestingModule({
  declarations: [ FooComponent, TestFooComponent ]
})

// 3. Here your specs using the wrapper.
it('should create', () => {
	const fixture: ComponentFixture<TestFooComponent> = TestBed.createComponent(TestFooComponent);
	const testInstance: TestFooComponent = fixture.componentInstance;
	
	expect(testInstance.fooInstance).toBeTruthy();
});
```

If you forget to add the test component to the TestBed config you get cryptic error like this.

> Error: Illegal state: Could not load the summary for directive TestFooComponent

```typescript
TestBed.configureTestingModule({
  declarations: [ FooComponent, TestFooComponent ]
})
```

With such a setup you can test your inputs easily and you can prepare edge cases of how your component being used. 

### Abstract Test Wrapper
To create other components easily you use an abstract class, to reflect the fact that every component you test will query for your actual component. Let's refactor this.

Rename `TestFooComponent` to `FooTest`, remove the component decorator and prepend a `abstract`. 

```typescript
abstract class FooTest {
  @ViewChild(FooComponent) fooInstance: FooComponent;
}
```

That's now our basic contract between all test components we will create. Let's recreate our first Test Component.

```typescript
@Component({
  template: `<app-foo></app-foo>`
})
class BasicComponent extends FooTest {}
```

Both components extend from our "contract" FooTest (`extends FooTest {}`) that way we can create a flexible helper function to create our components.

```typescript
describe('FooComponent', () => {
	let fixture: ComponentFixture<FooTest>;
	let testInstance: FooTest;
	let fooInstance: FooComponent;

	function createFooComponent(component: Type<FooTest>) {
		fixture = TestBed.createComponent(component);
		fixture.detectChanges();
		testInstance = fixture.componentInstance;
		fooInstance = testInstance.fooInstance;
	}
```

and use it like that:

```typescript
it('should create', () => {
	createFooComponent(BasicComponent);
	expect(testInstance.fooInstance).toBeTruthy();
});

```

This is nice but nothing more than a little refactoring so far. Where's the benefit you're asking ? Let's create a new feature. Our component should greet us now. 

### Test an Input
We start with how we expect to use that feature. That's easily possible now, as we have the test wrappers to create different scenarios.

```typescript
@Component({
  template: `<app-foo greeting="hello test"></app-foo>`
})
class GreetingComponent extends FooTest {}
```
Don't forget to add it to the test bed declaration fields in your testbed config. Let's create the test.

```typescript
it('should greet', () => {
	createFooComponent(GreetingComponent);
	expect(fixture.nativeElement.textContent.trim()).toContain('hello test');
});
```
> Expected 'foo works!' to contain 'hello test'.

### Implement
Good, test is failing, let's implement it 🙌
Add an `Input() greeting: string` and use it in the template `{{greeting}}.

Your test should switch to green.

The real beauty comes now. Replace your test component with the following version. Your test should stay green.

```typescript
@Component({
  template: `<app-foo [greeting]="currentGreeting"></app-foo>`,
})
class GreetingComponent extends FooTest {
  public currentGreeting = 'hello test';
}

```

Write a new test:

```typescript
it('should accept different greetings', () => {
    createFooComponent(GreetingComponent);
    const myInstance: GreetingComponent = testInstance as GreetingComponent;
    myInstance.currentGreeting = 'konichiwa & servus!';

    fixture.detectChanges();
    expect(fixture.nativeElement.textContent.trim()).toContain('konichiwa & servus!');
  });
```

We just created a test scenario where we can test differnt values pouring in from the input `greeting` by setting the value of the wrapper variable  `currentGreeting` and update the bindings with `fixture.detectChanges()`. With that technique you can exhaustive test any dumb component — which are components that only rely on inputs.

Great, let's look at some other aspects of testing. Async Testing is one of the difficult ones.

## Async Testing


## Micro & Macro Tasks
Let me begin with that example from [Jake Archibald](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
and his talk [In the loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0&feature=youtu.be) from 2018.

```
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

So what's the output ?

```
script start
script end
promise1
promise2
setTimeout
```

The output is a direct result of how the JavaScript event loop works. 
One for each web worker and each websites — the ones from the same origin share a event loop.

**(Macro) Tasks:** Work on JavaScript/DOM in a sequential way. So queue them up. Between tasks, the browser may render updates. 
Members: scripts, setTimeout

**Microtasks:** queued up and executed at the end of each task. 
Members:  mutation observer callbacks, Promises (event settled ones)

There is also a whatwg proposal [queueMicrotask](https://github.com/whatwg/html/issues/512) but it's still only in the specs.

Importan things here: Tasks give the browser the oopportunity to render things in between. Microtasks not as they can queue up endlessly if not used coorrectly.


This is one task:
```
document.appendChild(el);
el.style.display = 'none';
```

You will never see this flash 

setTimeout(cb, 0) is in reality setTimeout(cb, 4.7)


Promise.resolve().then(() => console.log('micro task'));
console.log('Task ended');

> Task ended
> micro task

Beware! This will hang your browser as it will endless fill the micro task queue and no task will ever run again after.

```javascript
function cb() {
	Promise.resolve().then(cb);
}

cb();

```

This one has test implications.

```javascript
document.body.addEventListener('click', function() {
	console.log('body clicked');
	Promise.resolve().then(() => console.log('micro task'));
});

document.body.addEventListener('click', function() {
	console.log('body clicked');
	Promise.resolve().then(() => console.log('micro task'));
})
```
Output:

```
> body clicked
> micro task
> body clicked
> micro task
```

but



```
document.body.click();

> body clicked
> body clicked
> micro task
> micro task
```

each async listener is creating a task.
but body.click() is already on the stack, soo the event listeners will execute synchronously and you get the micro tasks pushed to the end of the stack.

## Async Test Utilities
Let's talk about what asynchronicity is, how Angular get controls over it.

+ done 
+ fakeAsync & tick + flush (zone)
+ async & fixture.whenstable (zone)

### What is async ?
Every task that happens with a delay of time and is called through a callback. That can be a server call, a timeout or a any event like user interaction. So there is plenty of asynchronity in our applications. Actually the only thing an application can change are asychronious tests — the user is doing something, time passes and something is triggered and so on.

Angular uses a tool called zone.js to get full control of asynchronious tasks, so let's quickly talk about this as we need to understand it to properly test.

### ngZone Primer
What is zone.js doing?

Monkey patching all async triggers, there aren't that many (via [Understanding Zones](https://blog.thoughtram.io/angular/2016/01/22/understanding-zones.html))

```
Zone.setInterval()
Zone.alert()
Zone.prompt()
Zone.requestAnimationFrame()
Zone.addEventListener()
Zone.removeEventListener()
```

With those zones at hand we have complete control over macro und micro tasks. Angular take use of it during runtime to automatically detect changes in your application and render those changes.

But full control over micro and macro tasks is also exactly what we want when testing. We can run async code sequential. You will see how in the following parts.

### fakeAsync & tick + flush (zone)
With the poweer of zones we can test our asynchronious code synchroniously. Because Tasks and Microstasks are queued up and the developer tells when everything is ready to be process just before the testing is happening.

Those are your best friends when handling async function calls in your test — that includes timers & events. 

You open a zone with `fakeAsync` to gain control over all micro and macro tasks. If you do so the docs clearly tell you what helpers you want to use: `tick()` and `flushMicrotasks();`.

You can use tick() without any argument to flush micro tasks and execute pending tasks (setTimeout(fn, 0)) but you can also use it to simulate progress of time to execute future calls. See this:

```
it('setTimeout(0) & tick ', fakeAsync(() => {
	let state = [];
	
	setTimeout(() => { state.push('timeout called'); });
	setTimeout(() => { state.push('timeout called after 2s'); }, 2000);
	
	expect(state).toEqual([]);
	
	tick();
	expect(state).toEqual(['timeout called']);
	
	tick(2000);
	expect(state).toEqual(['timeout called', 'timeout called after 2s']);
}
```

There is a third one called `flush()`. It's the newest member (well since Angular 4.2). It will progress time as long as there are macro tasks. You don't have to cumbersome manage the time yourself if this is not an important detail of your app.


```
it('setTimeout(0) & tick ', fakeAsync(() => {
	let state = [];
	
	setTimeout(() => { state.push('timeout called'); });
	setTimeout(() => { state.push('timeout called after 2s'); }, 2000);
	
	expect(state).toEqual([]);
	
	flush();
	expect(state).toEqual(['timeout called', 'timeout called after 2s']);
}
```
### async & fixture.whenstable (zone)
What abotu async/whenStable?

They are also happening in the fake zone like fakeAsync but you will have to wait for the real time to pass and use `fixture.whenStable()` to execute after your component has settled. You usually stick to fakeAsync these days. Only if you do some testing with real XHR (what you shouldn't) you can use async as fakeAsync would not work.

### done
That's a jasmine helper and has nothing to do with the zone. You can simply take over control when your test is done. Normally jasmine would complete after the code block ends (either with or without specs being processed). This might help in edge cases but again your daily life of tests consists of normal tests without done or fakeAsync tests.

```
it('manually finish your spec', (done) => {
	console.log('run');
	expect(true).toBe(true);
	done();
});

it('this will hang', (done) => {
	console.log('run');
	expect(true).toBe(true);
});

it('this will not hang', () => {
	console.log('run');
	expect(true).toBe(true);
});
```

## Change Detection in your specs
We already touched the zone system that allows Angular to handle changes being triggered in your application. There is an detailed [blog post](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f) about the whole topic of ChangeDetection that I recommend every Angular developer to read.

I won't go into details about the change detection itself but I want to separate them from your async test utilities.

Change Detection is about updating components and in the end their representaiton in the DOM. You could also say it's the template update mechanism.

### How it works
Angular has zone in place. Any async call is therefore detected.
So Angular will run the change detector frequently and after almost anything the application is doing.

But running the change detector is not updating a template inevitable. It's only the opportunity to do so. The change detector will now ask each component in your application if there are changes.

A component will check (mostly through `===`) every single interpolation expression in a template and update that part if the value has changed from the last time. This also allows too detect changes in mutable objects like arrays — but only if you access the elements and not only the array itself.

### Detect Changes in Tests

Just remember one rule:
> Whenever you expect changes in your template, call `fixture.detectChanges()`.

That's it.

There is also a possibility to perform automatic change detection with the service `ComponentFixtureAutoDetect`. Provide through your testbed and good to go. Right? No. It's only for async changes (macroo & micro). Things like setting an input programmatically won't be detected. Before running into failures with this automagic use the opportunity, think about your what your component will do and use the right tools. You don't need automatic change detection in tests.


### OnPush
You might have heard of `OnPush`. That's a mode you can put your component into.
When zone detects changes in the app it would normally start at the root of your app and traverse down to all children telling them to check for changes.,

With OnPush you stop Angular from process your component tree. You tell Angular, that you ensure that the only thing to check are the inputs. If they are not changed nothing inside the component has changed. Comparable to `shouldComponentUpdate()` in react.

This is different to the previous, more general approach without any additional complexity where thinks just work. Before: Every component template/expression got checked. Now you stop Angular from doing so. This saves a lot of performance in large applications.

OnPush component updates if:

+ Input() references changes
+ event handler gets triggered (includes any UI event, so a nested button will trigger an update)
+ observable linked to the template via the async pipe emits a new value


Visualization:
https://hackernoon.com/angular-2-4-visualizing-change-detection-default-vs-onpush-3d7ed1f69f8e

### Summary
Default CD is robust because. It's triggered often and re-evaluate all the expressions in the templates of ALL components.

OnPush stops Angular from performing template evaluation in your component and all children. OnPush only allows component template to be evaluated when the input bindings (no object mutation!) have changed or an event occured inside the component itself.
There is an [nice read](https://blog.mgechev.com/2017/11/11/faster-angular-applications-onpush-change-detection-immutable-part-1/) about real world consequeences.

