In the [previous article](http://tutorials.pluralsight.com/front-end-javascript/getting-started-with-angularjs) we seen the basic implementaion of angularJS. In this articles I will talk about filters, in-built ngDirectives and custom ngDirective.

This article covers the following areas for angularJS with implementation of exapmles.

* Filter Component
* In-build Directives
* Create our own custome Directive
* comparison between ng-include and Custom Directive

## Filter Component

Angular provides [Filter component](https://docs.angularjs.org/api/ng/filter) to filter, orgnized and arrange the values according to requirements.

1. Filter
    
    - [Filter](https://docs.angularjs.org/api/ng/filter/filter) returns a new array as subset from an array after filtering it based on expression and condition.
    
    - filter can be used in HTML Template as wel as in JavaScript.

        * In HTML Template
        
        ```html
            {{ filter_expression | filter : expression : comparator}}
        ```
        
        * In JavaScript
        
        ```js
            $filter('filter')(array, expression, comparator)
        ```
        
        In HTML template, we use filter in pipe expression as in above. here, between this two call we have a pipe, pipes take the result from first expression and send me output into second expression.
        
        **Explanation with example** (filter in HTML Template) :
        
        ```html
            <!-- ng-app - attech an application module to the page -->
            <html ng-app="myApp">
            
            <!-- ng-controller - attech a controller functions to the page -->
            <body ng-controller="myCtrl">
            <!-- ng-init to initialize products as an array -->
            <div ng-init="products = [{ name : 'sony', price : 23, quantity : 4},
                                      { name : 'nokia', price : 45.3, quantity : 3},
                                      { name : 'sumsang', price : 65, quantity : 6},
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
        In above example `products` is an actual array and shows a filtered values based on the key entering in searchText.
        
        #### Source Code :
        
        [Plunker](https://plnkr.co/edit/buFI2NSz3Ago2KfillkB?p=preview) for filter in HTML Template.
        
        **Explanation with example** (filter in JavaScript) :
        
        ### controller.js
        ```js
        // register myCtrl to angular module app.
        app.controller('myCtrl',function($scope,$filter){
            // create an array as $scope.products
            $scope.products = [{ name : 'sony', price : 23, quantity : 4},
                      { name : 'nokia', price : 45.3, quantity : 3},
                      { name : 'sumsang', price : 65, quantity : 6},
                      { name : 'motorola', price : 12.7, quantity : 8},
                      { name : 'micromax', price : 39.75, quantity : 3},
                      { name : 'lenovo', price : 10, quantity : 2}
                    ]
            
            // created mirror copy of actual array
            $scope.mirrorProducts = angular.copy($scope.products);                 
            
            // bind function to ng-keyup event.
            $scope.filterFunc = function(){
                // override the value of mirrorProduct with filtered value
                $scope.mirrorProducts = $filter('filter')($scope.products,{$ : $scope.searchText}); 
            }
        });
        ```
        
        ### index.html
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
                  <!-- use mirrorProducts array to display changes, since, we are override mirrorProducts array each time with original products array. -->
                  <tr ng-repeat="p in mirrorProducts">
                    <td>{{p.name}}</td>
                    <td>{{p.price}}</td>
                    <td>{{p.quantity}}</td>
                  </tr>
                </tbody>
            </table>
        </body>
        ```
        
        - When we use filter component in angular's controller, we have to load the dependency in controller's function as `$filter`. Since, we are using `$filter('filter')(array, expression, comparator)`.
        
        - Type of filter component is `filter`, hence the code `$filter('filter_component')` i.e `$filter('filter')`.
        
        - In the above example, I use `$scope.mirrorProducts` variable and override each time the user press any key in input field, since we are showing only filtered information to the user we have to create subset of actual array and override each time to display a correct result on page.
        
        - What if I do not create a mirror copy of actual array ?
            >Its very simple, If I does not create a mirror copy of actual array then at time of filter an actual array goes to override and we have only filtered value, that means we loss our actual array values.
        
        - We uses `$` as expression while filtering an array.
        
        - `$` is a special property that can be used to filter on any property of the object.
        
        #### Source Code :
        
        [Plunker](https://plnkr.co/edit/kLVpQiianqKOXrTGExbi?p=preview) for filter in JavaScript
        
2. Currency filter :
    - Angular provides a better way to formating the price and display on page.
    - [Currency](https://docs.angularjs.org/api/ng/filter/currency) filter formats a number as a currency.
    - Currency filter formating the number to proper decimal points(like $25.70).
    - Currency filter can be used in HTML Template as wel as in JavaScript.
    
    ** In HTML Template**
        
        ```html
        {{ currency_expression | currency : symbol : fractionSize}}
        ```
        
        - Here, currency expression is the numerical value that will be formated by curency filter to display numerical value as price with currency symbol.
        
        - Second parameter after the pipe expression is a name of filter component i.e `currency`.
        
        - `symbol` and `fractionSize` are the options value.
        
        - `symbol` is to put your locale currency symbol to format the number to price and `fractionSize` is a number of decimal placed to round the price. 
            
            for example :
        
        ```html
            <p>{{25 | currency }}</p>
            <!-- this will print result as $25.00-->
                
            <p>{{25 | currency : "₹" }}</p>
            <!--this will print result as ₹25.00-->
                
            <p>{{25.46 | currency : "₹" : 4}}</p>
            <!--This will print result as ₹25.4600 as factionSize is 4 decimal-->
        ```
        
    **In JavaScript Template**
        
        ```js
        $filter('currency')(amount, symbol, fractionSize)
        ```
        
        - `Symbol` and `fractionSize` aur the same as in HTMP Template.
        - `Amount` is the numerical value which has to convert into currency format.
        - `$filter` is an instance of filter component service which is inject to controller's function as an dependency.
            
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
        
        Currency format with cutom symbol :
        ```html
            <!--JS code-->
            $scope.price = $filter('currency')(25,'₹');
            
            <!--html code-->
            <p>{{price}}</p>
            
            <!--Output-->
            ₹25.00
        ```
        
        Currency format with cusmot decimal points :
        ```html
            <!--JS code-->
            $scope.price = $filter('currency')(25,'₹',4);
            
            <!---Html Code-->
            <p>{{price}}</p>
            
            <!--Output-->
            ₹25.0000
        ```
            
        see  [plunker](https://plnkr.co/edit/cjYyN9RjvtsxIdmn3PCz?p=preview) for live example.