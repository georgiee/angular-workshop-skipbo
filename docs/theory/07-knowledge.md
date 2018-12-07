# Did you know

## Debugging

To get access to the Angular debug element:

```
ng.probe(domElement)
```

In your browser console you can access the current DOM Element with `$0` (and `$1`, `$1`, `$3`, `$4`). This will access its Angular Debug Element and through it your component instance 💪

```
ng.probe($0). componentInstance;
```

Access injectors

```
ng.probe(span).injector.get('MyAppProviderToken');
```


Trigger change detection:

```
ng.probe($0).injector.get(ng.coreTokens.ApplicationRef).tick();
```

Access the merge injector of any component:

```
// merge injector
ng.probe($0).injector

// element injector
ng.probe($0).injector.elDef.element
```

How to get an injected token:
> If we ask for some token in child component it will first look at child element injector, where checks elRef.element.allProviders|publicProviders, then goes up through all parent view elements and also checks providers in element injector.

> If the next parent view element equals null(2) then it returns to startView(3), checks startView.rootData.elnjector(4) and only then,

> if token won’t be found, checks startView.rootData module.injector

See

+ https://blog.angularindepth.com/everything-you-need-to-know-about-debugging-angular-applications-d308ed8a51b4
+ Very [good read](https://blog.angularindepth.com/angular-dependency-injection-and-tree-shakeable-tokens-4588a8f70d5d) about Angular DI details


## JSON Output
You can copy ANY element in JSON format without converting it explicitly with JSON.stringify.

Try this in your console.

```
const x = [1,2,3,4,5, {x: 1, y: [1,2,3]}]
copy(x)
```

Past anywhere your clipboard content:

```
[
  1,
  2,
  3,
  4,
  5,
  {
    "x": 1,
    "y": [
      1,
      2,
      3
    ]
  }
]
```

## console.dir
So you already knew about `$0` and friends?
If yout output the values you get a representation of the DOM, but if you want to list & access the actual JavaScript properties. What do you then ?

Try this (while a DOM element is selected)

```
> console.dir($)
```

and compare it ith this

```
> $0
```

## components are directives
You can see it in the sources. Components are Directives with template and css. And CSS for Directives is in the top feature list that will get added to Angular once Ivy is released. Igor Minar told us in the Q&A Panel at Angular Connect.

## Inject Host Component
You can inject any parent component if you get your hands on the class reference. Be careful with this as this is a sign of tight coupling which is a bad sign is most cases. But it's sometimes useful for examples to create tight binding between Dropdown and its items. But as with most things there is always another, better way of doing such a coupling. Before injecting some parent component create a service to mediate between both components. Here is a nice examples how to do it:

https://blog.angularindepth.com/here-is-how-to-get-viewcontainerref-before-viewchild-query-is-evaluated-f649e51315fb

search for `shared.registerContainer(vc);`

## Chrome Stack Traces.
Did you know, you can hide parts of the framework you are sure will never take part in any error. That makes a stack trace more readable.

Look here 💪
![](../images/blackbox-before.png)
![](../images/blackbox-after.png)

## forRoot vs forChild
The FAQ is co clear about it in case you are undecided when to use which method and when to use it in your own modules.

> forRoot() and forChild() are conventional names for methods that configure services in root and feature modules respectively.

> Angular doesn't recognize these names but Angular developers do. Follow this convention when you write similar modules with configurable service providers.
>

## Injection Debugging

You can see all root providers in your console:
```
ng.probe(getAllAngularRootElements()[0]).injector.view.root.ngModule._providers
```

You can use ng.probe to access injectors of components:

```
ng.probe($0).injector.get('MyToken');
```
## Web Componist
If that's one of the things that pop up in your mind when thinking of web developing — congratulations you are a web component developer.

```
:host {
  display: block;
}
```

## Template Export
You all know template references.

```
<div #yourReference>content</div>
```

You might have seen this:

```
<div #yourReference="someKeyword">content</div>
```

That's the template pendant to something that is more common in your component code:

```
@ContentChild('yourReference', {read: ElementRef})
```


That read option is what you tell Angular with the keyword. By default it's mostly ElementRef. But if you have multiple directives and you want the instance of a single directive you can use the keyword of the directive. When you create a directive there is a field to name that keyword:

```
@Directive({
  selector: '[myDirective]',
  exportAs: 'someKeyword',
```

That's nice handy in many situations.


## Bash
Did you know that you can jump back to the last branch?
`git checkout -`

if you don't know this maybe you don't know that this origin in the cd command,
where you can jump back to the last working directory with `cd -`

## Opensource
Create a folder `~/opensource` and whenever you see an interesting project, don't stop with a start on github. Clone it and quickly browse through it. When you are working on a daily project, remember that project and if it fits in the overall complex browse through it again for inspiration about structure & code techniques.

Angular Material is for examples my goto project to look for ideas around RxJS & Testing.
\



github y t


twitter:
https://twitter.com/umaar?lang=en

oh-my-zsh, (wenige plugins 🤓)

https://github.com/sindresorhus/awesome

https://pinboard.in/
https://raindrop.io/


https://www.npmjs.com/package/npm-check

Lighthouse Audit
web.dev


https://github.com/ebidel/lighthouse-httparchive

https://www.igvita.com/2013/06/20/http-archive-bigquery-web-performance-answers/

Performance Trace: FTTI,

webpack bundel analzyer



ng build --stats-json

[Jake Archibald Micro Macro Tasks Vortrag](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
