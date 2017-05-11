## Qu'est-ce que VueJS ?

VueJS est une librairie permettant de développer des interfaces web interactives.  

L'interactivité vient du fait que la vue peut-être mise à jour dynamiquement lorsque le modèle de données est modifié.  

De même les données du modèle peuvent être modifier lorsque l'utilisateur interagit à travers l'interface.  

De nombreuses libraries intègrent cette fonctionalité. L'atout principale de VueJS est son approche très aisée, et son ecosytème qui ne cesse de grandir.

D'autre part VueJS peut parfaitement être utilisé côté serveur, dans une application .NET MVC par exemple. 


## VueJS et JQuery

VueJs ne remplace pas JQuery. D'ailleurs de nombreuses applications VueJS intègre JQuery.  Il est toutefois vrai que VueJS minimise son usage.  


## Application VueJS minimale

Partie html :
```html
<div id=app>
 {{ message }}
</div>
```

Partie javascript :
```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

Résultat :  
<pre>Hello Vue!</pre>

Comme on peut le constater dans cet exemple, VueJS se charge de sélectionner le bon élément sans recourir à une autre librarie telle que JQuery par exemple.

Exemple en ligne: [Application minimale](http://embed.plnkr.co/3Wai4JuBQ4DmNFRSHh8c/)  



L'aspect dynamique de VueJS peut-être visualisé en modifiant la propriété 'message' 

Entrez par exemple dans le fichier 'script.js':
```javascript
app.message ="Autre message..."
```

La vue contenant l'élément lié à cette propriété  est dynamiquement modifiée.

