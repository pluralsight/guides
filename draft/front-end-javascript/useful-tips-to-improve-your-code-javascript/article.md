1.  Detecting the properties of an object
    ```javascript
    var myObject = {
        name: 'Javier',
        lastName: 'Ruiz Vázquez'
    };
    if ('name' in myObject) {
        console.log('My name is: ', myObject.name);
    } else {
        console.log('The property doesn`t exist');
    }
    ```
    
    This is very useful to know if an attribute of the object exists and avoid getting undefined attributes or functions, ie you return undefined.