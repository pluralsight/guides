

## Browser console

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

