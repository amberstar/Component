# The Component Pattern

## What is the component pattern?
The component pattern is a simple experiment to reason about the structure of software. It can be tried in varying degrees mostly by convention with exsiting types you already know. The ideas here are not unique, in fact you will recognize the ideas and see them everywhere, and as far back as the Unix OS, and the Lisp  programming language, and probably way before that.

The purpose is to raise the question, is this a logical truth, and if so why not use tools that set the constraint?

![Composition](img/Component2.png)

## What is a component?
A component is any composable type that has input, a process, and output. A better way to think about a component is as a set of functions because a component can have a set of intpus and a set of output, but we can think of those sets as a single truth.. In the component pattern a pure function is a component as well, because it meets those requirements. I explain what I mean in the next section about operators.

![Component](img/Component1.png)

# Operators
Operators are small simple components that compose together to make new ones, just like any component composition. They take a single input, and may produce output, or, not. The not part is signifigant. It's not that operations do or don't output, it's that it may or it may not, depending on it's context and purpose. For example a filter operator.

**The idea behind operators are simple:**
- a useful abstraction is "as much process with as little content as possible". 

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
We start with `Take` to declare we want to input `MyStruct`, we then perform a `map` and an `action`.  This defines a new `operator`. Now we a distinct operation, a small component we can input values into and produce a result. 

```swift

myOperator.input(myStruct) 

```

## Sample Operators 
(see `operator.swift` for more)

| `Operator`         | Description                                                                            |
|--------------------|----------------------------------------------------------------------------------------|
| `Take`          | Produces it's input                                                                    |
| `Limit`         | Produces it's input a limited number of times                                          |
| `Distinct`         | Produces distinct values relative to it's last output                                  |
| `Discard`          | Produces Void regardless of input                                                      |
| `Action`           | Performs an action with  and produces it's input                            |
| `Map`              | Produces the result of mapping a function over it's input                              |
| `Filter`           | Produces output that satisfies a predicate                                             |
| `Reduce`           | Produces the result of calling a `combine` function on each input and the last combine |
| `Produce`           | Produces a result from a producer function |
| `Count`            | Produces a count of it's input                                                         |
| `Branch`           | Branch with the current output, then continue on the main branch                       |
| `Combine`          | Combines it's input with the output of an operator producing a tuple      |
| `.print`            | Prints and produces it's input                                                             |
| `.description`      | Produces inputs description                                                            |
| `.debugDescription` | Produces the inputs debug description                                                  |
| `.prefix`           | Prefix the string input with a string                                                  |
| `.date`             | Produce a tuple with the input and a date                                              |
| `.defaultTo`             | Produce a default value if the no output is produced                              |
