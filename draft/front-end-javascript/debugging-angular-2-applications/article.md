## Console
The Chrome Devtools enable you to get the refernce of an element you have selected by using <code> $0 </code>

![chrome-dev-tools-selecting-element](https://raw.githubusercontent.com/pluralsight/guides/master/images/3d8e63dc-d62e-433d-88ae-347d436b452a.gif)

### Inspecting a component
Having the reference of an DOM element, you have the ability to inspect the scope of the component it is located in. You can do this by using <code>ng.probe()</code>
```
> ng.probe($0)
```

**<code>ng.probe($0)</code> enables you to access the scope of the component itself and inspect and manipulate its various attributes directly from the console.**. You can go ahead and explore the component's model and change things around in its instance:

```
> ng.probe($0).componentInstance
```



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b0b6563f-3a82-4cd5-aeb5-55f9cb993376.gif)




Because you are editing the component programatically isntead of doing it directly through the DOM, Angular does not detect the change in the model.If you are coming from Angular 1, you know you can apply the changes by creating [$digest](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest) event that will check for changes in all the watchers in the applications.

**However, in Angular 2, each component has its own scope, so you must apply the changes to the component's scope itself.**

```
ng.probe($0)._debugInfo._view.changeDetectorRef.detectChanges()

```



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/384ceda7-7415-4229-b83a-574af117e820.gif)

## Source maps and debugger
Enabling source maps will let you see the TypeScript code of your application instead of the transpiled JavaScript code, making it easier for you to debug your code line by line.

To enable sourcemaps, edit <code> tsconfig.json </code>

**tscofig.json**
```json

{
  "compilerOptions": {
    "sourceMap": true,

  }
}
```

Now, in your source code, whever you put the <code> debugger </code> statement, you'll be able to see the TypeScript code in your debugger, instead of your transpiled JavaScript:
```

search(term: string) { 
    this.searchSubject.next(term); 
    debugger; //debug here!
}
    
```


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ec5840ee-3d67-4fdd-a532-f0649aba9899.06)

## JSON pipe

 [Pipes](https://angular.io/docs/ts/latest/guide/pipes.html) are used to represent data in a particular manner in Angular 2. They bear close resemblance to Angular 1's [filters](https://docs.angularjs.org/api/ng/filter/filter). 
 
 The JSON pipe can help you debug by representing models as JSON objects. Just put  <code>| json </code> next to the name of one of your component models:
 
```html
<p> I can see the whole model here! :</p>
{{hero | json}}
 
```


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/fc69c2d4-829d-4f52-80d7-254a66969fe1.04)

