# Method Dispatch in Swift (note)

##### Learned from blog: https://www.raizlabs.com/dev/2016/12/swift-method-dispatch/  

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
    3. If the method is created in subclass, append the method’s address to the end of in the function table.

    Example:  
    
```Swift    
  class ParentClass {  
      func method1() {}
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
    
      1. Read the dispatch table for the object 0xB00  
      2. Read the function pointer at the index for the method. In this case, the method index for function2 is 1, so the address 0xB00 + 1 is read.
      3. Jump to the address 0x222  
      
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

  * Note: lookup is guarded by a fast cache layer that makes lookups almost as fast as table dispatch once the cache is warmed up.
    
    

## Method Dispatch in Swift
#### Four aspects that guide how dispatch is selected in Swift: Declaration Location, Reference Type, Specified Behavior and Visibility Optimizations

* Location: 

|                  | Declared in Class| Declared in Extension |
| ---------------  |:----------------:| :--------------------:|
| Property         | Direct Dispatch  | Direct dispatch       |
| Protocal         | Table Dispatch   | Direct dispatch       |
| Class            | Table Dispatc    | Direct dispatch       |
| NSObject subclass| Table Dispatc    | Table Dispatc         |


<br/><br/>
* Reference Type:  
  * Type of reference also determines how to dispatch methods. 
  * Example: 
  ```Swift
    protocol MyProtocol {}
    
    extension MyProtocol {
      func extensionMethod() {
        print("Extension In Protocol")
      }
    }
    
    struct MyStruct: MyProtocol {}
    
    extension MyStruct {
      func extensionMethod() {
        print("Extension In Struct")
      }
    }
 
    let myStruct = MyStruct()
    let myPro: MyProtocol = myStruct
    
    myStruct.extensionMethod() // -> “Extension In Struct”
    myPro.extensionMethod() // -> “Extension In Protocol”
  ```
  
  ###### Why calling `myPro.extensionMethod()` results in the output `“Extension In Protocol”`? Because, firstly, according to the location table above, this calling uses direct dispath(in other word, there is not 'override' behaviour), second, type of `myPro` is `MyProtocal`, only methods is visible to the protocal is used to direct dispatch.
  
  Let's see another example:
  ```Swift
  protocol MyProtocol {
  
    func mainMethod(){}
  }
  
  extension MyProtocol {
  
    func mainMethod() {
      print("main method in Protocal")
    }
  }
  
  struct MyStruct: MyProtocol {
    func mainMethod() {
      print("main method in Structure")
    }
  }
  
  let myStruct = MyStruct()
  let myPro: MyProtocol = myStruct
  
  myStruct.mainMethod() // -> "main method in Structure" 
  proto.mainMethod() // -> "main method in Structure" 

  ```
  ###### Why calling `proto.mainMethod()` results in output `"main method in Structure"`? Because, according to location table above, this calling uses table dispatch, therefore, 'override' occured, the method `mainMethod()` in `MyProtocal` is overrided by the one in `MyStruct`.


