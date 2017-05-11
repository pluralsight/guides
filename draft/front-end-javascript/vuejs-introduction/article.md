### Qu'est-ce que VueJS ?

VueJS est une librairie permettant de développer des interfaces web interactives.  

L'interactivité vient du fait que la vue peut-être mise à jour dynamiquement lorsque le modèle de données est modifié.  

De même les données du modèle peuvent être modifier lorsque l'utilisateur interagit à travers l'interface.  

De nombreuses libraries intègrent cette fonctionalité. L'atout principale de VueJS est son approche très aisée, et son ecosytème qui ne cesse de grandir.

D'autre part VueJS peut parfaitement être utilisé côté serveur, dans une application .NET MVC par exemple. 


### VueJS et JQuery

VueJs ne remplace pas JQuery. D'ailleurs de nombreuses applications VueJS intègre JQuery.  Il est toutefois vrai que VueJS minimise son usage.  


### Application VueJS minimale

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


### Les directives de base

L'exemple suivant illustre les directives : v-bind, v-model, v-on

* v-bind  : permet de lier un attribut à une propriété
* v-model : permat de lier la valeur prise par un champ modifiable à une propriété
* v-on : permet de réagir aux évènements (click...)

```html
<!DOCTYPE html>
<html>

  <head>
    <link rel="stylesheet" href="style.css">
    <script src="https://unpkg.com/vue"></script>
  </head>

  <body>
    <div id="app">
      
      <h1 v-bind:title="titre" v-bind:style="style" >{{ message }}</h1>
      
      <input type="text"  v-model="couleur">
      
      <button v-on:click="hy">Say Hy</button>
      
      <p style="margin-top:40px;">Changer la couleur du message</p>
      
      <table style="margin-top:10px;">
        <tr><td>R</td><td><input type="range" value="0" max="255" min="0" step="1" v-model="rouge"></td></tr>
        <tr><td>V</td><td><input type="range" value="0" max="255" min="0" step="1" v-model="vert"></td></tr>
        <tr><td>B</td><td><input type="range" value="0" max="255" min="0" step="1" v-model="bleu"></td></tr>
      </table>
      
    </div>
    <script>
      var app = new Vue({
        el: '#app', 
        data: { 
          message: 'Propriétés liées et calculées',
          titre : "Titre de message",
          couleur : "",
          rouge: 0,
          vert : 0,
          bleu : 0,
        },
        methods : {
          hy : function () {
            alert("Hy Vue")
          }
        },
        computed : {
          style : function () {
            return {"color" : "rgb(" + this.rouge + "," + this.vert + ","+ this.bleu + ")"}
          }
        },
      }); 
    </script>
  </body>

</html>
```

Vous pouvez tester cet exemple ici : [Directives de base](https://embed.plnkr.co/v4DKhwGLuQoPPeMRZYfS/)

