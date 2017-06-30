



### Development Guidelines

#### Naming conventions
1. `Directory names` should be `dash`(-) separated  Ex: `app-config` , `customer-order`.
2. `File names` should be `dash`(-) separated Ex : `create-order.js` , `cancel-order.js`.
3. `camelCase` should be used for variable and functions declaration.
    ```
    let billingAddress={}; let deliveryAddress = {};
    
    function getOrder(orderid){
        //code goes here
    }
    ```
4. `PascalCase` should be used for `class` or `ES6 module` 
    ```
    class CreateOrder{
        constructor(){
            this.order = order
        }
    }
    
    //while importing a ES6 module which exports more than one function
    
    ** file: order-operation.js **
    ------------------------------
    
    function getOrder(){
        //
    }
    function createOrder(){
        //
    }
    
    module.exports = {
        getOrder:getOrder,
        createOrder : createOrder
    }
    
    ** file: index.js **
    --------------------
    
    import OrderOperation from './order-opertion'; //PascalCase
    
    ```
5. All `constants` should be declared with underscore separated `ALL_CAPS` case 
    EX : `const VAN_DELIVER=1`
