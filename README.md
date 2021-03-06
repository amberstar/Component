# The Component Pattern

This project is a collection of ideas around structuring software.
The component pattern is generally a pattern of understanding. It can be implemented or reasoned about in varying degrees mostly by convention using existing types you already know. The ideas here are not unique, in-fact you will see this pattern everywhere including  in Unix, Lisp, and many other surprising places.
 
![Component](img/ComponentPattern.jpg)
Components define the structure of software and are are composed together as shown in the diagram above. Arrows pointing in represent input. Arrows pointing out represent output. Outer components are parent components of inner components. This keeps dependencies pointing inward and output outward. No inner component is dependent on an outer component until it is in the context of its parent. Because the dependencies flow in the direction of composition of the system , the dependencies are inherent in the system itself.

What is a component?
--------------------
Components are based on the idea that software has three axis, **structure**, **behavior**, and **state** and each are interdependent to make a coherent well organized system. Note that behavior is synonymous with process, and implies time. A component provides structure, and is defined as any composed structural element in this pattern that has input, a process(behavior), and output. Another way to think about a component is a pure function with a set of possible inputs and a set of possible outputs.

![Component](img/Component1.png)

Composition:
------------
 ![](img/Composition2.png)
 
Components are composed of sub-components as shown above. The Y axis represents composition or structure. A component receives input, some process happens, and it produces output at some point in time. This is represented by time int the X axis . Each child components input is a mapping of its parent's input. Each child components output is a reduction into the parents state and may or may not cause output from the parent. 
 
  -  Input  = ParentInput -> ChildInput
 
  -  Output = ParentInput, ChildOutput -> ParentOutput? 
  
  ![Composition](img/Component2.png)

A triangle is formed from the focal point of reasoning (number 1) within the parent scope to the input into children (number 3) and output from children (number 4). This pattern recurses forming a hierarchy.

1. parent component (The focal point of reasoning)
2. parent inputs
3. input sent from parent to child component
4. output from child component received by parent 
5. parent outputs


The Constraints:
------------
1. Only a parent component can send input to owned child components
2. Only a parent connects child outputs to other child inputs
3. Only a parent (or sibling connected by a parent) can respond to output of a component
4. Output from a child component may or may not cause output from a parent component

Note: If objects are used, public methods (and public properties) can only be called by owning parents.

Why public methods(that return values) and properties are harmful to software structure, and the significance of an output.
------------------------------
When asked about the meaning of Object Oriented Programming Alan Kay once said:

>> *"OOP to me means only messaging, local retention and protection and
 hiding of state-process, and extreme late-binding of all things"*

What is wrong with messaging in common OOP is that objects communicate across their structural and temporal boundaries and at the request of the receiver. A receiver has to ask another object for its state when it needs it. The source of facts sends the state back to caller at the time of request, not at the time of truth. Software is a process moving forward in time and any recorded data in the system is potentially out of date. 

  Messaging in the component pattern occurs on a "plane of truth" in realtime. Asynchronicity happens on a orthogonal plane of truth. All messaging is coordinated from a focal point of reasoning (the parent scope)

What is important::

1. The existence of an output in context of some structure. (components) delivered as facts occur.
2. The intention of the receiver to receive that output as facts occur.

Public methods(that return values) and properties do not provide these, and is why components have outputs.


**_Isn't an output an observable?_**

Yes, sort of. The observable pattern focuses on _values over time_ but I believe that the higher order idea is the idea of _a value at some point in time._ and in context. Facts are useless without context. If I said "42" you have no idea what that means. But if I say the "thermometer output 42" it has meaning. Here we have structure, behavior, and state.

This is the significance of a component output, it coerces proper system semantics with constraints.

**Side note:**
 I believe a pattern smell is when a software system is made of parts that have obscure abstract names that are nothing about the solution. Names like presenters, interactors, or routers for example.

# Simple Components: Operators
Operators are simple components that compose together to make new ones. They take a single input, and may produce output, or, not. The not part is important. It's not that operations do or don't output, they may or may not, depending on context and purpose. For example a filter operator.

**The idea behind operators are simple:**

- a useful abstraction is "as much process with as little content as possible".
- processing a value should not be coupled to its delivery method. (such as observable operators)

```swift
/// A type that operates on values possibly producing a different type,
/// or no value at all.
public protocol OperatorProtocol {
    associatedtype Input
    associatedtype Output
    
    /// Operates and produces the next value with the specified input.
    mutating func input(_ input: Input) -> Output?
    
    /// Composes a target operator with `self` and returns a new operator.
    func compose<Target:OperatorProtocol>(_ : Target) -> Operator<Input, Target.Output> where Target.Input == Output
}
```

```swift
var myOperator = Take<MyStruct>().map{ format($0.myProperty) }.action { label.text = $0 }
```
We start with `Take` to declare we want to input values of type`MyStruct`, we then perform a `map` and an `action`.  This defines `myOperator`, a new `operator`. Now we have a distinct purposeful operation component we can input values into and produce a result.

```swift

myOperator.input(myStruct) 

```

**Example Operators**

| `Operator`         | Description                                                                            |
|--------------------|----------------------------------------------------------------------------------------|
| `Take`          | Produces its input                                                                    |
| `Limit`         | Produces its input a limited number of times                                          |
| `Distinct`         | Produces distinct values relative to its last output                                  |
| `Discard`          | Produces Void regardless of input                                                      |
| `Action`           | Performs an action with  and produces its input                            |
| `Map`              | Produces the result of mapping a function over its input                              |
| `Filter`           | Produces output that satisfies a predicate                                             |
| `Reduce`           | Produces the result of calling a `combine` function on each input and the last combine |
| `Produce`           | Produces a result from a producer function |
| `Count`            | Produces a count of its input                                                         |
| `Branch`           | Branch with the current output, then continue on the main branch                       |
| `Combine`          | Combines its input with the output of an operator producing a tuple      |
| `.print`            | Prints and produces its input                                                             |
| `.description`      | Produces inputs description                                                            |
| `.debugDescription` | Produces the inputs debug description                                                  |
| `.prefix`           | Prefix the string input with a string                                                  |
| `.date`             | Produce a tuple with the input and a date                                              |
| `.defaultTo`             | Produce a default value if no output is produced                              |
