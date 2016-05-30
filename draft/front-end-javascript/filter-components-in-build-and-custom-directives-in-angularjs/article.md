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
        
        We use use filter in pipe expression as in above. here, between this two call we have a pipe, pipes take the result from fisrt expression and send me output into second expression.
        
        **for example** (filter in HTML Template):
        
        ```html
            // ng-app - attech an application module to the page
            <html ng-app="myApp">
            
            // ng-controller - attech a controller functions to the page
            <body ng-controller="myCtrl">
            // ng-init to initialize products as an array.
            <div ng-init="products = [{ name : 'sony', price : 23, quantity : 4},
                                      { name : 'nokia', price : 45.3, quantity : 3},
                                      { name : 'sumsang', price : 65, quantity : 6},
                                      { name : 'motorola', price : 12.7, quantity : 8},
                                      { name : 'micromax', price : 39.75, quantity : 3},
                                      { name : 'lenovo', price : 10, quantity : 2}]">
            </div>
            
            // input filed to type expression to be filter
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
                    // filter based on value of searchText
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
        
        **for example** (filter in JavaScript):
        
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
                  // use mirrorProducts array to display changes, since, we are override mirrorProducts array each time with original products array. 
                  <tr ng-repeat="p in mirrorProducts">
                    <td>{{p.name}}</td>
                    <td>{{p.price}}</td>
                    <td>{{p.quantity}}</td>
                  </tr>
                </tbody>
            </table>
        </body>
        ```
        
        #### Source Code :
        
        [Plunker](https://plnkr.co/edit/kLVpQiianqKOXrTGExbi?p=preview) for filter in JavaScript
        
        
        
        
        
        
        




