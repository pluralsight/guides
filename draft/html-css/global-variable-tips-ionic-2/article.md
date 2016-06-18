### **Build a provider for global variables.**

From prompt shell, at app root do :
 
 > ionic generate provider app-datas
 
This will create file _app/providers/app-datas/app-datas.ts_

You can use this Global Variables Provider in  file _app.ts_ add:

> import { AppDatas } from './providers/app-datas/app-datas';

> ionicBootstrap(MyApp,[..,AppDatas],...);

You can use in all others components pages :

> import { AppDatas } from './providers/app-datas/app-datas';

> contructor ( ... , public appDatas: AppDatas,...) {
>   ...
>   this.appDatas.uid = "ASDGJGJDGJGDJGD";
> }

Sample you can set a variable **_uid_** and init data in **app.ts** and show data in a other component page:

