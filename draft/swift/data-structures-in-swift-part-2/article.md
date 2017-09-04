# Data Structures You Should Know

This is the second part of a two-part series on data structures in Swift. In the first part, [we delved into generics and built-in Swift collection types](https://www.pluralsight.com/guides/swift/data-structures-in-swift-part-1). 
Check it out in case you missed it. 

In this tutorial, we’re going implement some of the most popular data structures from scratch in Swift. 

Swift provides some handy collection types. We can use the generic __Array__, __Set__, and __Dictionary__ right away. Yet, there may be cases when we need custom collection types. In the upcoming chapters, we’ll take a closer look at stacks, queues, linked lists, and graphs.

# The Stack
The stack behaves like an array, in that it stores the elements in a specific order. However, unlike for arrays, we can’t randomly insert, retrieve and delete items from a stack. 

The stack only permits pushing a new item, that is, appending it to the end of the collection. Also, we can only remove the last item, also known as pop.

The stack provides a __Last-In-First-Out__ (in short, LIFO) access. The item that was added last (or pushed) can be accessed first (peek or pop).
Here’s an illustration of the LIFO behavior:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9ad7373a-1c7c-445c-bf4c-db53e493ecae.png)

The stack is useful if we need to **retrieve the elements in the order in which we added them**.
We could fulfill this requirements using an Array:
```
var numbers = [Int]()
numbers.append(1)
numbers.append(2)
numbers.append(3)
var last = numbers.last // peek - returns 3
last = numbers.popLast() // pop - returns 3
last = numbers.popLast() // pop - returns 2
last = numbers.popLast() // pop - returns 1
```
But, the array does not prevent us from (accidentally) accessing the elements in a different order:
```
last = numbers[2]
numbers.remove(at: 3)
```
Thus, if we need a strict LIFO behavior, we have to create a custom stack collection.
Our Stack needs three methods:  
- push() - to append a new item
- peek() - retrieves the last element without removing it
- pop() - removes the last element and returns it

Optionally, we can add a ```removeAll()``` method to wipe out the stack.

There are different ways to implement these requirements. Let’s follow a [protocol-oriented approach and declare a protocol first](https://www.pluralsight.com/guides/swift/protocol-oriented-programming-in-swift) .
```
protocol Stackable {
    associatedtype Element
    mutating func push(_ element: Element)
    func peek() -> Element?
    mutating func pop() -> Element?
    mutating func removeAll()
}
```
The Stackable protocol defines all the requirements needed to define a stack. 

We use an ```associatedtype``` called Element that acts as a placeholder. Types that adopt the Stackable protocol can provide the exact type.

Notice the use of ```mutating``` for all the methods which change the contents of the stack. We must mark methods that modify a structure as ```mutating```. 
Class methods can always modify the class. Thus we don’t need to use the ```mutating``` keyword in classes.

>We used the mutating keyword in the Stackable protocol to also allow structs adopt it.

Next, we implement a generic stack.
```
struct Stack<T>: Stackable {
    typealias Element = T

    fileprivate var elements = [Element]()
    
    mutating func push(_ element: Element) {
        elements.append(element)
    }
    
    func peek() -> Element? {
        return elements.last
    }
    
    mutating func pop() -> Element? {
        return elements.popLast()
    }
    
    mutating func removeAll() {
        elements.removeAll()
    }
}
```
```Stack``` can work with any type:
```
var stackInt = Stack<Int>()
var stackString = Stack<String>()
var stackDate = Stack<Date>()
```
We can use the stack as follows: 
```
stackInt.push(1)
print("push() ->", stackInt)  
// Output: push() -> [1]

stackInt.push(2) 
print("push() ->", stackInt)  
// Output: push() -> [1, 2]

stackInt.push(3) 
print("push() ->", stackInt)  
// Output: push() -> [1, 2, 3]

print("peek() ->", stackInt.peek() ?? "Stack is empty")  
// Output: peek() -> 3

print("peek() ->", stackInt.peek() ?? "Stack is empty")  
// Output: peek() -> 3

print("peek() ->", stackInt.peek() ?? "Stack is empty")  
// Output: peek() -> 3

print("pop() ->", stackInt.pop() ?? "Stack is empty")  
// Output: pop() -> 3

print("pop() ->", stackInt.pop() ?? "Stack is empty")  
// Output: pop() -> 2

print("pop() ->", stackInt.pop() ?? "Stack is empty")  
// Output: pop() -> 1

stackInt.pop() 
print(stackInt)  // Output:[]
```