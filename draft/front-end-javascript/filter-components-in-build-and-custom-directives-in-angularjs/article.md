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
        
        We use use filter in pipe expression as in above. here, between this two call we have a pipe, pipes take the result from fisrt expression and send me output into secon expression.
        
        for example
        
        ```html
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
        ```
        
        




