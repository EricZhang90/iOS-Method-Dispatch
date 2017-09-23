# Method Dispatch in Swift (note)

Learned from blog: https://www.raizlabs.com/dev/2016/12/swift-method-dispatch/  

## In gerenal:
Q: What is method dispatch in computer science?  
A: A instruction that tells the CPU where in memory to find the executable code for a method call.
<br/>
<br/>
<br/>
Three primary method dispatch mechanisms in complied programming language:  
###  Direct dispatch(static)
  * Such as C++ by default
  * Advantage: Fastest
  * Disadvantage: Not good at subclassing
###  Table dispatch(dynamitic)
  * Such as Java by defualt
  * The class has an array (function table) of function pointers for each method. The subclass has own copy of function table. There are the rules for creating the subclass’s function table:
    1. If the method is extended from super class, simply copy the address of the method to the subclass’s function table.
    2. If the method is overrided, add the new method’s address to the subclass’s function table.
    3. If the method is newly created in subclass, append the method’s address to the end of in the function table.

    Example:  
    
```Swift    
  class ParentClass {  
      func method1() {
      func method2() {}
  }
  class ChildClass: ParentClass {
      override func method2() {}
      func method3() {}
  } 
```  

<img src="http://www.raizlabs.com/dev/wp-content/uploads/sites/10/2016/12/virtual-dispatch.png" align="left" hspace="10" vspace="6">  
<br/>

    What happen in runtime is:
    
      1. Read the dispatch table for the object 0xB00  
      2. Read the function pointer at the index for the method. In this case, the method index for function2 is 1, so the address 0xB00 + 1 is read.
      3. Jump to the address 0x222  
      
  * Advantage: Logic is simple; Performance is predictable
  * Disadvantage: 
    * Slow: 1, Two additional reads and a jump from a byte-code point of view. 2, Cannot optimize implementation inside the method.
    * Extensions cannot extend the function table because adding a new function pointer from class extension to the function table in the basic class would cause ‘index out of bounds’ or an undefined behavior that overrides an existing memory address. [Is it the reason why Java and C++ doesn’t support partial class or extension?]

### Message dispatch(dynamitic)
  * Such as Objective-C
  * Crawls class hierarchy in runtime to determine which method to invoke. Can changes method's behaviour and class's concept in runtime.
  * Example:
```Swift    
  class ParentClass {  
      dynamic method1() {
      dynamic method2() {}
  }
  class ChildClass: ParentClass {
      override func method2() {}
      dynamic method3() {}
  } 
``` 
<img src="https://www.raizlabs.com/dev/wp-content/uploads/sites/10/2016/12/message-dispatch.png" hspace="10" vspace="6">
<br/>


Note: lookup is guarded by a fast cache layer that makes lookups almost as fast as table dispatch once the cache is warmed up.
    
    
