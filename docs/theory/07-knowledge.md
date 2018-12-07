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

## Chrome Stack Traces.
Did you know, you can hide parts of the framework you are sure will never take part in any error. That makes a stack trace more readable.

Look here 💪
![](../images/blackbox-before.png)
![](../images/blackbox-after.png)

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

## Bash
Did you know that you can jump back to the last branch?
`git checkout -`

if you don't know this maybe you don't know that this origin in the cd command,
where you can jump back to the last working directory with `cd -`

## Opensource
Create a folder `~/opensource` and whenever you see an interesting project, don't stop with a start on github. Clone it and quickly browse through it. When you are working on a daily project, remember that project and if it fits in the overall complex browse through it again for inspiration about structure & code techniques.

Angular Material is for examples my goto project to look for ideas around RxJS & Testing.

## Lost & Found
Here some ideas from the workshop when we exchanged tips & tricks.

+ github: While being on a repository, try to press the key `t` to find any file

+ github: While being on a file in a repository presse key `y` to get the full url including the sha of the commit — that way you can link a file that will never change even if the code in the repository changes.

+ Interesting Twitter User for daily coding tips
https://twitter.com/umaar?lang=en

+ oh-my-zsh (with few plugins only 🤓) is a great supplement to zsh.

+ Others are not fan of oh-my-zsh as it can slow down the boot time of your shell.

+ List of Awesome Lists: https://github.com/sindresorhus/awesome

+ Tools to help you manage bookmarks.
  + https://pinboard.in/
  + https://raindrop.io/

+ Interactive alternative to `npm outdated`: https://www.npmjs.com/package/npm-check

+ We have seen the Lighthouse Audit applied to your agency page

+ web.dev can run lighthouse tests without an extension. web.dev is a page curated by google to help developers with tools and lear material.

+ I briefly mentioned how Lighthouse calcualtes the score with a formula based on the httparchive and the results in the Google BigQUery DB. Here some links:
  + https://github.com/ebidel/lighthouse-httparchive
  + https://www.igvita.com/2013/06/20/http-archive-bigquery-web-performance-answers/

+ We briefly looked at the webpack bundle analzyer to analzye the bundel content. Steps are: ng build with stats-json and use the jsinjson to generate a visualization. Either with the webpack bundle analyzer or some online tool.

+ Micro & Macro was heard, here he amazing and very recent talk from Jake: [Jake Archibald Micro Macro Tasks 2018](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
