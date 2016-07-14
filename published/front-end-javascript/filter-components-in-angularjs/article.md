In my [previous article on AngularJS](http://tutorials.pluralsight.com/front-end-javascript/getting-started-with-angularjs), we got a look at the basic implementaion of AngularJS. In this article we will look at **Filter Components**, which are integral parts to angular.

This article covers the following areas for angularJS with implementation and examples.

* Filter
* Currency
* Number
* Date
* Lower Case/ Upper Case
* limitTo
* orderBy

### **Filter Component**

Angular provides [filter components](https://docs.angularjs.org/api/ng/filter) for filtering, organizing and arranging the values based on input requirements.

## Filter

A [filter](https://docs.angularjs.org/api/ng/filter/filter) returns a subset of a new array based on the conditions and expressions from the array. Filters can be used in HTML Template as well as in JavaScript.

- **Syntax**
    
    * HTML Template:
        
    ```html
        {{ filter_expression | filter : expression : comparator}}
    ```
    
    * JavaScript:
    
    ```js
        $filter('filter')(array, expression, comparator)
    ```
    
    In HTML Template, we use `filter` in the pipe expression `|` as shown above. here. This pipe function takes the result from first expression and sends the output to second expression.
    
- **Explanation with example** (filter in HTML Template) :
     
    ```html
    <!-- ng-app - attech an application module to the page -->
    <html ng-app="myApp">
    
    <!-- ng-controller - attech a controller functions to the page -->
    <body ng-controller="myCtrl">
    <!-- ng-init to initialize products as an array -->
    <div ng-init="products = [{ name : 'sony', price : 23, quantity : 4},
                              { name : 'nokia', price : 45.3, quantity : 3},
                              { name : 'samsung', price : 65, quantity : 6},
                              { name : 'motorola', price : 12.7, quantity : 8},
                              { name : 'micromax', price : 39.75, quantity : 3},
                              { name : 'lenovo', price : 10, quantity : 2}]">
    </div>
    
    <!-- input filed to type expression to be filter -->
    <div>
        <label>Search</label>
        <input ng-model="searchText" class="form-control" />
    </div>
    
    <table class="table">
        <tbody>
            <tr>
                <th>Name</th>
                <th>Price</th>
                <th>Quantity</th>
            </tr>
            <!-- filter based on value of searchText -->
            <tr ng-repeat="p in products | filter:searchText">
                <td>{{p.name}}</td>
                <td>{{p.price}}</td>
                <td>{{p.quantity}}</td>
            </tr>
        </tbody>
    </table>
    
    </body>
    <html>
    ```
    * In the example, `products` is an actual array that shows a set of filtered values based on the input, `"searchText"`.
        
    ##### Source Code :
        
    [Plunker](https://plnkr.co/edit/buFI2NSz3Ago2KfillkB?p=preview) for filter in HTML Template.
        
- **Explanation with example** (filter in JavaScript) :
    
    ##### controller.js file
    ```js
    // register myCtrl to the angular module app.
    app.controller('myCtrl',function($scope,$filter){
        // create an array as $scope.products
        $scope.products = [{ name : 'sony', price : 23, quantity : 4},
                  { name : 'nokia', price : 45.3, quantity : 3},
                  { name : 'samsung', price : 65, quantity : 6},
                  { name : 'motorola', price : 12.7, quantity : 8},
                  { name : 'micromax', price : 39.75, quantity : 3},
                  { name : 'lenovo', price : 10, quantity : 2}
                ]
        
        // create a mirror copy of actual array
        $scope.mirrorProducts = angular.copy($scope.products);                 
        
        // bind function to ng-keyup event.
        $scope.filterFunc = function(){
            // override the value of mirrorProduct with filtered value
            $scope.mirrorProducts = $filter('filter')($scope.products,{$ : $scope.searchText}); 
        }
    });
    ```
        
    ##### index.html
    ```html
    <body ng-controller="myCtrl">
        <div class="form-group col-lg-10">
            <label class="label label-default" style="margin-left:10px">Search</label>
            <input ng-model="searchText" class="form-control" ng-keyup="filterFunc()" />
        </div>

        <table class="table">
            <tbody>
              <tr>
                <th>Name</th>
                <th>Price</th>
                <th>Quantity</th>
              </tr>
              <!-- use mirrorProducts array to display changes because we are overriding the mirrorProducts array with the original products array every time -->
              <tr ng-repeat="p in mirrorProducts">
                <td>{{p.name}}</td>
                <td>{{p.price}}</td>
                <td>{{p.quantity}}</td>
              </tr>
            </tbody>
        </table>
    </body>
    ```
        
    - When we use filtering in angular's controller, we have to load the dependency in the controller's function as `$filter`. So, we are using `$filter('filter')(array, expression, comparator)`.
    
    - Type of filter component is `filter`, hence the code `$filter('filter_component')` i.e `$filter('filter')`.
    
    - In the above example, I use the `$scope.mirrorProducts` variable and override each time the user presses any key in input field; since we are showing only filtered information to the user, we have to create subset of the actual array and override it each time to display correct results on page.
        
    - What if I do not create a mirror copy of actual array ?
        >If I do not create a mirror copy of the actual array, then, at the time of filtering, the actual array gets overridden and we are left with just the filtered version. Thus, we lose our original array values.
    
    - We use `$` as the expression while filtering an array.
    
    - `$` is a special property that can be used to apply filter on any property of the object.
    
    ##### Source Code :
    
    [Plunker](https://plnkr.co/edit/kLVpQiianqKOXrTGExbi?p=preview) for filter in JavaScript
        
## Currency

Angular provides a better way to format a price and display it on page. The [currency](https://docs.angularjs.org/api/ng/filter/currency) filter formats a number as currency, setting hte number to the proper decimal value (like $25.70). This filter can be used in HTML Template as well as in JavaScript.

**In HTML Template**
        
```html
{{ currency_expression | currency : symbol : fractionSize}}
```
    
* Here, currency expression is the numerical value that will be formatted by currency filter to display numerical value as a price with a specified currency symbol.
        
* The second parameter after the pipe expression is a name of filter component i.e `currency`.
        
* `symbol` and `fractionSize` are the option values.
        
* `symbol` is to put your locale currency symbol to format the number to price and `fractionSize` determines the number of digits after the decimal point to which the price will be recorded. 
            
for example :
        
```html
<p>{{25 | currency }}</p>
<!-- This will print result as $25.00. Note that the default fractionSize is 2. -->

<p>{{25 | currency : "₹" }}</p>
<!--This will print result as ₹25.00. -->
        
<p>{{25.46 | currency : "₹" : 4}}</p>
<!--This will print result as ₹25.4600 because fractionSize is 4 decimal places. -->
```
        
**In JavaScript Template**
        
```js
$filter('currency')(amount, symbol, fractionSize)
```
        
* `symbol` and `fractionSize` are the same in JS as in HTMP Template.
* `Amount` is the numerical value which has to be converted into currency format.
* `$filter` is an instance of the filter component service which is injected to controller's function as a dependency.
            
for example :
        
Currency Format:
```html
<!--JS code-->
$scope.price = $filter('currency')(25);
        
<!--html code-->
<p>{{price}}</p>
    
<!--output-->
$25.00
```
    
Currency format with custom symbol:
```html
<!--JS code-->
$scope.price = $filter('currency')(25,'₹');
    
<!--html code-->
<p>{{price}}</p>

<!--Output-->
₹25.00
```
    
Currency format with custom decimal points:
```html
<!--JS code-->
$scope.price = $filter('currency')(25,'₹',4);
    
<!---Html Code-->
<p>{{price}}</p>
    
<!--Output-->
₹25.0000
```
            
see  [Plunker](https://plnkr.co/edit/cjYyN9RjvtsxIdmn3PCz?p=preview) for more examples.
        
## Number
    
The [number](https://docs.angularjs.org/api/ng/filter/number) filter formats a number as text. This filter can be used in HTML Template as well as in JavaScript.

**HTML Template**

```html
{{ number_expression | number : fractionSize}}
```
    
* `number_expression` is the number which has to be formatted into text.
* `number` is the value to be formatted.
* `fractionSize` is the number of spaces after the decimal point to display after formatting.
        
for example :
            
```html
<p>{{24.76 | number}}</p>
<!--this will print result as 24.76-->
            
<p>{{24.76 | number : 0}}</p>
<!--this will print the result as 24-->
            
<p>{{24.76 | number : 1}}</p>
<!--this will print result as 24.8-->
```

* `fractionSize` provide a result after rounding-up the value. 
        
**JavaScript Code**
```js
$filter('number')(number, fractionSize)
```
        
* `$filter` instance of filter components.
* `$flter(number)` type of filter components.
* `number` to be format.
* `fractionSize` is the count of decimal point display after formatting.
        
for example : 
```js
$scope.number = $filter('number')(25.76)
// this will print a result as 25.76
        
$scope.number = $filter('number')(25.76 : 0)
//this will print a result as 25
    
$scope.number = $filter('number')(25.76 : 1)
//this will print a result as 25.8
```
        
see [Plunker](https://plnkr.co/edit/Wv5glncChtFVuZiBCEkh?p=preview) for more.
        
## Date

The [date](https://docs.angularjs.org/api/ng/filter/date) filter provides a better way to represent date in string format (`MMM d, y h:mm:ss a` will show as `Sep 3, 2010 12:05:08 PM`)
- **Syntax**
    * In HTML Template Binding
    ```html
    {{ date_expression | date : format : timezone}}
    ```
    * In JavaScript
    ```
    $filter('date')(date, format, timezone)
    ```
    
- **Explanation** : 
    
Param | Type | Details 
----- | ---- | -------
date | `Date` `Number` `string` | `Date` is the date to format either as a Date object, a number of milliseconds (string or number), or an ISO 8601 datetime string (yyyy-MM-ddTHH:mm:ss.sssZ, yyyy-MM-ddTHH:mmZ, yyyy-MM-dd, yyyyMMddTHHmmssZ, etc.)
format (optional) | `string` | Formatting rules. If not specified, `mediumDate` is used.
timezone (optional) | `string` | Timezone to be used for formatting. It understands UTC/GMT and the continental US time zone abbreviations, but for general use, use a time zone offset, for example, '+0430' (4 hours, 30 minutes east of the Greenwich meridian) If not specified, the timezone of the browser will be used.

* The `Date` filter has predefined elements to formatting the date. For convenience, I've listed them out below.

Element | Details
------- | -------
`'yyyy'` |  4 digit representation of year (e.g. AD 1 => 0001, AD 2010 => 2010)
`'yy'`|2 digit representation of year, padded (00-99). (e.g. AD 2001 => 01, AD 2010 => 10)
`'y'`| 1 digit representation of year, e.g. (AD 1 => 1, AD 199 => 199)
`'MMMM'`| Month in year (January-December)
`'MMM'`| Month in year (Jan-Dec)
`'MM'`| Month in year, padded (01-12)
`'M'`| Month in year (1-12)
`'LLLL'` | Stand-alone month in year (January-December)
`'dd'` | Day in month, padded (01-31)
`'d'` | Day in month (1-31)
`'EEEE'` | Day in Week,(Sunday-Saturday)
`'EEE'` | Day in Week, (Sun-Sat)
`'HH'` | Hour in day, padded (00-23)
`'H'` | Hour in day (0-23)
`'hh'` | Hour in AM/PM, padded (01-12)
`'h'` | Hour in AM/PM, (1-12)
`'mm'` | Minute in hour, padded (00-59)
`'m'` | Minute in hour (0-59)
`'ss'` | Second in minute, padded (00-59)
`'s'` | Second in minute (0-59)
`'sss'` | Millisecond in second, padded (000-999)
`'a'` | AM/PM marker
`'Z'` | 4 digit (+sign) representation of the timezone offset (-1200-+1200)
`'ww'` | Week of year, padded (00-53). Week 01 is the week with the first Thursday of the year
`'w'` | Week of year (0-53). Week 1 is the week with the first Thursday of the year
`'G'`, `'GG'`, `'GGG'` | The abbreviated form of the era string (e.g. 'AD')
`'GGGG'` | The long form of the era string (e.g. 'Anno Domini')

### Predefined localizable formats 

Element | Details
------- | -------
`'medium'` | equivalent to 'MMM d, y h:mm:ss a' for en_US locale (e.g. Sep 3, 2010 12:05:08 PM)
`'short'` | equivalent to 'M/d/yy h:mm a' for en_US locale (e.g. 9/3/10 12:05 PM)
`'fullDate'` | equivalent to 'EEEE, MMMM d, y' for en_US locale (e.g. Friday, September 3, 2010)
`'longDate'` | equivalent to 'MMMM d, y' for en_US locale (e.g. September 3, 2010)
`'mediumDate'` | equivalent to 'MMM d, y' for en_US locale (e.g. Sep 3, 2010)
`'shortDate'` | equivalent to 'M/d/yy' for en_US locale (e.g. 9/3/10)
`'mediumTime'` | equivalent to 'h:mm:ss a' for en_US locale (e.g. 12:05:08 PM)
`'shortTime'` | equivalent to 'h:mm a' for en_US locale (e.g. 12:05 PM)


Implementation of the `Date` filter is same as the implementation of `number` and `filter`. Let's implement a simple example:
   
```html
<span>{{1288323623006 | date:'medium'}}</span><br>
<!--output : Oct 29, 2010 9:10:23 AM -->
        
<span>{{1288323623006 | date:'yyyy-MM-dd HH:mm:ss Z'}}</span><br>
<!--output : 2010-10-29 09:10:23 +0530 (date format with time zone)-->
        
<span>{{'1288323623006' | date:'MM/dd/yyyy @ h:mma'}}</span><br>
<!--Output : 10/29/2010 @ 9:10AM (insert string/character inside date format as '@')-->
        
<span>{{'1288323623006' | date:"MM/dd/yyyy 'at' h:mma"}}</span><br>
<!--Ouput : 10/29/2010 at 9:10AM (insert string/character inside date format as 'at')-->
```

## LowerCase/UpperCase
- [LowerCase](https://docs.angularjs.org/api/ng/filter/lowercase) and [UpperCase](https://docs.angularjs.org/api/ng/filter/uppercase) converts a string into lower case and upper case.
    
- Lower Case
    * In HTML Template Binding
    ```html
    {{ lowercase_expression | lowercase}}
    ```
    * In JavaScript
    ```js
    $filter('lowercase')()
    ```
    * For example
    ```html
    <html ng-app="myApp">
        <body ng-controller="myCtrl">
            <!--this will convert text into lowercase-->
            <p>Using HTML : {{"HTML CODE TO CONVERT INTO LOWERCASE" | lowercase}}</p>
        </body>
    </html>
    <!--OutPut : html code to convert into lowercase-->
    ```
    * see [Plunker](https://plnkr.co/edit/Bb6ciKENeIVkbR74ly9I?p=preview) for live example

- Upper Case
    * In HTML Template Binding
    ```html
    {{ uppercase_expression | uppercase}}
    ```
    * In JavaScript
    ```js
    $filter('uppercase')()
    ```
    * For example
    ```html
    <html ng-app="myApp">
        <body ng-controller="myCtrl">
            <!--this will convert text into uppercase-->
            <p>Using HTML : {{"html code to convert into upper case" | uppercase}}</p>
        </body>
    </html>
    <!--OutPut : HTML CODE TO CONVERT INTO UPPER CASE-->
    ```
    * see [Plunker](https://plnkr.co/edit/qJChu75CJJggnCRsqQxZ?p=preview) for live example
        
## limitTo

The [limitTo](https://docs.angularjs.org/api/ng/filter/limitTo) filter returns a new array as a subset limited to the specified number of elements. The elements are taken from either the beginning or the end of the source array, string or number, as specified by the value and sign (positive or negative) of limit. If a number is used as input tag, that number is converted into a string value.
    
- **Syntax**
    * In HTML Template Binding
    ```html
    {{ limitTo_expression | limitTo : limit : begin}}
    ```
    * In JavaScript
    ```
    $filter('limitTo')(input, limit, begin)
    ```
- **Explanations** :
    
    * `input` is the source array, number, or string which has to be limited.
    * `limit` is the desired length of the subset array, string or number.
    * If the limit number is positive, `limitTo` will create a subset by copying from the beginning of the source (array or string).
    * If `limit` is negative, `limitTo` will create a subset by copying from the end of the source (array or string) are copied.
    * The limit will be truncated if it's more than `array.length`. If `limit` is undefined, the system will return the unchanged input array.

Let's see an example:
    
```html
<!-- ng-app - attach an application module to the page -->
<html ng-app="myApp">
        
<!-- ng-controller - attach a controller functions to the page -->
<body ng-controller="myCtrl">
    
<!-- ng-init - to initialize "friends" as an array -->
<div ng-init="friends = [
                        {name : 'John', phone : '89765', age : 34},
                        {name : 'Bob', phone : '32722', age : 28},
                        {name : 'Jake', phone : '87865', age : 30},
                        {name : 'Pop', phone : '67547', age : 26},
                        ]">
</div>
    
<table>
    <tr>
        <th>Name</th>
        <th>Phone</th>
        <th>Age</th>
    </tr>
    <!-- filter based on value of limitTo -->
    <tr ng-repeat="f in friends | limitTo : 2">
        <td>{{f.name}}</td>
        <td>{{f.phone}}</td>
        <td>{{f.age}}</td>
    </tr>
</table>
    
</body>
</html>
    
<!--output shows only 2 records because we restricted limitTo to just 2
Name	Phone	Age
John	89765	34
Bob 	32722	28
-->
```
* See [Plunker](https://plnkr.co/edit/SLBIjepAgsxEgcYxUzhv?p=preview) for more.

## orderBy

[OrderBy](https://docs.angularjs.org/api/ng/filter/orderBy) specifies an order to an array by expression. Strings are ordered alphabetically. Numbers are ordered numerically. 

- **Syntax**
    * In HTML Template Binding
    ```html
    {{ orderBy_expression | orderBy : expression : reverse}}
    ```
    * In JavaScript
    ```js
    $filter('orderBy')(array, expression, reverse)
    ```
- **Explanation**

    * `array` is the array to be sorted.
    * `expression` is the expression on which comparator predicts the order of element.
    * `reverse` is the element order.
    
    
- The minus sign (`-`) will denote descending order. See the below example:
    
    ```html
    <!-- ng-app - attech an application module to the page -->
    <html ng-app="myApp">
        
    <!-- ng-controller - attech a controller functions to the page -->
    <body ng-controller="myCtrl">
    
    <!-- ng-init to initialize friends as an array -->
    <div ng-init="friends = [
                            {name : 'John', phone : '89765', age : 34},
                            {name : 'Bob', phone : '32722', age : 28},
                            {name : 'Jake', phone : '87865', age : 30},
                            {name : 'Pop', phone : '67547', age : 26},
                            ]">
    </div>
    
    <table>
        <tr>
            <th>Name</th>
            <th>Phone</th>
            <th>Age</th>
        </tr>
        <!-- Order by ascending order based on value of age -->
        <tr ng-repeat="f in friends | orderBy : 'age'">
            <td>{{f.name}}</td>
            <td>{{f.phone}}</td>
            <td>{{f.age}}</td>
        </tr>
    </table>
    <br>
    <table>
        <tr>
            <th>Name</th>
            <th>Phone</th>
            <th>Age</th>
        </tr>
        <!-- Order by descending order based on value of age -->
        <tr ng-repeat="f in friends | orderBy : '-age'">
            <td>{{f.name}}</td>
            <td>{{f.phone}}</td>
            <td>{{f.age}}</td>
        </tr>
    </table>
    
    </body>
    </html>
    ```
    * See [Plunker](https://plnkr.co/edit/CSlQblf57hSPrVwfJVlC) for more.
    
These are the Filter Components provided by angular, and this is how we use them. I hope you found this article informative! Please leave your feedback in the comments below. See you soon with my next article on AngularJS. 