# Preparations

## Editor
Use whatever coding editor you are comfortable with. if you are unsure I recommend Visual Code Editor (VSCode) from Microsoft.
Make sure you have the following extension (or similar ones in other editors) installed:

+ *Angular Language Service*: to get language support in your html templates
+ *tslint*:  to be a good citizen in the programming world
+ Switch Typescript to the Workspace version (bottom right in VSCode when you have a TS file open). Otherwise you will have problems with Intellisense. That will hunt you in any Angular project.

IntelliJ/Webstorm Users:
The provided `karma.conf` uses a Headless Chrome configuration

```
browsers: ['Chrome_Headless'],
```

Unfortunately this seems not to work in Webstorm,
as the underscore is mistreated. This will work.
This workshop is not yet updated for this, so here the fix:


```
browsers: ['ChromeHeadless'],
```

Attention when switch branches (you will do this often) — this will revert this and you have to apply the changes again.

## Code
Clone this workshop repository. Use `--recurse-submodules` as it will include the Angular project (/skipbo-angular) itself which is included here only as a submodule.

```bash
git clone --recurse-submodules https://github.com/georgiee/angular-workshop-skipbo
```

Switch in branch `workshop/start` and install all dependencies with `npm install`.
Open the folder `skipbo-angular` in your editor.

## Workshop Process
+ There are six chapters.
+ Each chapter has a theory part and a challenge part.
+ Each challenge will start with a dedicated branch to provide additional files
and to make progress with the game itself (beyond the parts you will implement)
+ You don't have to commit, we will change branches and you will loose your changes — but you will recognize
the code you have written. You are part of the progress.

## Start
→ The workshop will start with [Chapter 01 - Modules & Injection (Theory)](theory/01-modules.md)
