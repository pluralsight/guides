![ruby-on-rails-angular2](https://raw.githubusercontent.com/pluralsight/guides/master/images/034d7c62-833d-45a7-8222-04c90edb4759.png)

##### ** El proposito de este material es aportar una guia para la integracion de ruby on rails con angular 2. Antes de comensar abrimos la terminal de linux "ctrl+alt+t" para chequear si se tiene instalado:**

#### Ruby

```
~ $ ruby --version

```

##### si la salida es algo asi:

```
ruby 2.3.0p0 (2015-12-25 revision 53290) [i686-linux]

``` 

###### bien podemos continuar si no revisa este material
* [Como instalar Ruby](http://hackguides.org)

#### Node

```
~ $ node --version

```

##### si la salida es algo asi:

```
v7.2.1

``` 

###### bien podemos continuar si no revisa este material 
* [Como instalar Node](https://www.npmjs.com/)

#### **listo ya podemos comenzar a integrar Angular2 y Ruby on Rails **

##### Creamos un nuevo proyecto ruby abre la terminal y ubicate sobre un directorio en donde se encontrara tu proyecto, la carpeta de trabajo puede ser tu repositorio git.

```
~ $ cd /home/nombre-usuario/carpeta-trabajo 
/home/nombre-usuario/carpeta-trabajo $ rails new nombre-proyecto

``` 
##### Con esto se crea tu proyecto con todas las carpetas y archivos del framework.

#### Se tiene que realizar los siguientes ajuste para tu aplicacion ruby.

##### nos ubicamos sobre el directorio root del proyecto

``` 
~ $ cd nombre-proyecto
nombre-proyecto $ 
``` 

##### con lo que se tiene en este momento es una aplicacion rails por defecto. entonces instalemos unos requerimiento necesarios para esto se tiene que ejecutar lo siguiente.


``` 
nombre-proyecto $ gem install rails

``` 
salida:
``` 
Successfully installed rails-5.0.1
Parsing documentation for rails-5.0.1
Installing ri documentation for rails-5.0.1
Done installing documentation for rails after 1 seconds
1 gem installed
``` 

##### De forma similar ejecutar de ser necesario 


``` 
nombre-proyecto $ gem install sqlite3
nombre-proyecto $ gem install sprockets
``` 

##### Todas estas gemas se intalaran usando un archivo llamado Gemfile. que es de creacion automatica cuando se ejecta los comandos anteriores o muy bien puedes editar de forma manual.

##### Para lograr que se aplique e instalen los cambios ejecutar en la terminal.

``` 
nombre-proyecto $ bundle install

``` 

##### Si la instalacion fue correcta la salida es:


``` 
Bundle complete! 15 Gemfile dependencies, 62 gems now installed.
``` 
##### para mas informacion
* [sqlite3](https://rubygems.org/gems/sqlite3/versions/1.3.11)
* [sprockets](https://github.com/rails/sprockets-rails)


### ** Listo Comensemos a integrar con Angular 2 **

------------------------

##### Nos ubicamos en el directorio root del proyecto y creamos los siguientes archivos

* package.json
* typings.json
* tsconfig.json

#### ** package.json  **
##### * contiene una lista de los paquete y dependencia requerido para integrar Angular 2*

``` 
{
 "name": "rails-ng2", 
"version": "0.0.1", 
"scripts": { "tsc": "tsc",
 "tsc:w": "tsc -w", "typings": "typings",
 "postinstall": "typings install" 
},
 "license": "ISC",
 "dependencies": { 
"angular2": "2.0.0-beta.15", 
"systemjs": "0.19.26",
 "es6-shim": "^0.35.0",
 "reflect-metadata": "0.1.2",
 "rxjs": "5.0.0-beta.2",
 "zone.js": "0.6.10",
 "intl": "^1.0.1" 
},
 "engines": { 
"node": "5.3.0" 
},
 "devDependencies": { 
"typescript": "^1.8.10",
 "typings":"^0.7.12" 
} 
}

``` 
##### Copia y pega el contenido en el archivo package que creaste, puedes encontrar mas informacion visitando [typings](https://www.npmjs.com/package/typings), [typescript](https://www.npmjs.com/package/typescript),

#### ** typings.json  **

##### * este es un archivo es usado para la configuracion de la dependencia del package puedes crearlo por terminal*

``` 
nombre-proyecto $ typings install
``` 
##### * o copiando y pegando este contenido en el archivo typings.json ya creado*

``` 
 {
      "globalDependencies": {
        "core-js": "registry:dt/core-js#0.0.0+20160602141332",
        "jasmine": "registry:dt/jasmine#2.2.0+20160621224255",
        "node": "registry:dt/node#6.0.0+20160807145350"
      }
    }
``` 

#### ** tsconfig.json  **

##### * el proximo archivo es el tsconfig.json *

``` 
{
  "compilerOptions": {
   "target": "es5",
   "module": "commonjs",
   "moduleResolution": "node",
   "sourceMap": true,
   "emitDecoratorMetadata": true,
   "experimentalDecorators": true,
   "removeComments": false,
   "noImplicitAny": false
  }
}
``` 

##### 

![seccion en construccion](https://raw.githubusercontent.com/pluralsight/guides/master/images/4f4d3e08-08f5-4aea-9331-514852212238.png)


