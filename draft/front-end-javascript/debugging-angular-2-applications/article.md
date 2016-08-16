# Enabling and disabling debugging






## Console
If you like being old school, there is a neat way to debug your Angular 2 application from the console.
To start off, you can use the Chrome Devtools to get the refernce of an element you have selected by using <code> $0 </code>.

![chrome-dev-tools-selecting-element](https://raw.githubusercontent.com/pluralsight/guides/master/images/3d8e63dc-d62e-433d-88ae-347d436b452a.gif)


Having the reference of an DOM element, you have the ability to **inspect the scope of the component it is located in**. You can do this by using <code>ng.probe()</code> and put the DOM element as an argument:
```
> ng.probe($0)
```

As you can see<code>ng.probe($0)</code> you can acess the scope of the component itself and manipulate its various attributes directly from the console.**. You can go ahead and explore the component's model and change things around in its instance:

```
> ng.probe($0).componentInstance
```



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b0b6563f-3a82-4cd5-aeb5-55f9cb993376.gif)


However, the changes are not reflected in the UI. Why?
Because you are editing the component programatically isntead of doing it directly through the DOM, Angular does not detect the change in the model.If you are coming from Angular 1, you may already know that you can apply the changes by creating [$digest](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest) event that will check for changes in all the watchers in the applications.

**However, in Angular 2, each component has its own scope, so you have to apply the changes to the component's scope itself.**

```
ng.probe($0)._debugInfo._view.changeDetectorRef.detectChanges()

```



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/384ceda7-7415-4229-b83a-574af117e820.gif)

## Debugger
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

##  JSON pipe

 [Pipes](https://angular.io/docs/ts/latest/guide/pipes.html) are used to represent data in a particular manner in Angular 2. They bear close resemblance to Angular 1's [filters](https://docs.angularjs.org/api/ng/filter/filter). 
 
 The JSON pipe can help you debug by representing models as JSON objects. Just put  <code>| json </code> next to the name of one of your component models:
 
```html
<p> I can see the whole model here! :</p>
{{hero | json}}
 
```


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/fc69c2d4-829d-4f52-80d7-254a66969fe1.04)

Now you are  be able to see all the model's properties being changed in real time as you interact with it.


## Augury


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/bba01541-7175-4ec5-8e58-59416799ac79.05)

[Angular Augury](https://augury.angular.io/) is a great way to make the debugging process of your application more visual. You can get it as a [Chrome Extension](https://chrome.google.com/webstore/detail/augury/elgalmkoelokbchhkhacckoklkejnhcd).



With Augury
## Logging