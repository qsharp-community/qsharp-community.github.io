---
title: "Partial application in Q#"
author: filipw
date: 2021-12-14
categories:
  - blog
tags:
  - contributed-post
  - language
  - compiler
---

🎄 This post is part of the [Q# Advent Calendar 2021](https://devblogs.microsoft.com/qsharp/q-advent-calendar-2021/). 🎅

One of the very useful, though not very well known, language features of Q# is its support for partial application of callables. In general, we can say that a partially applied function is a function that is manually derived from another function by supplying fewer parameters than the original function expects. This concept is an integral part of almost every functional programming language, though it might be a bit exotic for developers with other backgrounds, especially as many languages of more object-oriented nature, such as C#, do not have native support for partial function application.

### Basics of partial application

Consider the following basic Q# function, which defined three parameters of `Int`, `String` and `Result` type. The body of the function is irrelevant for this discussion, and we only need to focus on its signature:

```
function ThreeArgumentFunction(number : Int, text : String, result : Result) : Unit {
    Message($"{number}");
    Message(text);
    Message($"{result}");
}
```

With the regular function invocation process, we'd need to always supply all three argument values when the function is called. However, using partial application, the function can be transformed, without being evaluated, into another function which accepts fewer parameters than the original one. At this point we have a function pointer that may be invoked with a different parameter set than the source function, or that could be passed around to other functions.

More practically speaking, this new function is created when the calling code passes only the subset of function arguments - those, whose values it can provide already at that point. The rest of the arguments have to be replaced with the discard symbol `_` and the new function only expects parameters corresponding to those for which the discard was used.

For example, by passing a specific `Result` value such as `Zero` as positional argument three, we can convert our `ThreeArgumentFunction` into a new two argument function, which defines two parameters of `Int` and `String` type only, and which internalizes the pass-in argument `Zero`.

```
let twoArgumentFunction = ThreeArgumentFunction(_, _, Zero); 
```

Using the same methodology, by taking the newly created `twoArgumentFunction` and passing a specific value for one of the arguments and using a discard for the other, the function can be converted into a single argument one:

```
let oneArgumentFunction = twoArgumentFunction(100, _); 
```

At that point we may call the resulting function `oneArgumentFunction` with just a `String` parameter:

```
oneArgumentFunction("hello!"); 
```

### Partial application with tuples

Things get even more interesting with tuples, in particular due to singleton tuple equivalence in Q#. Let us first consider a function accepting two parameters, both of them tuples of different kind.

```
function TwoArgumentFunctionWithTuples(
	numbers : (Int, Int, Int), 
	results : (Result, Result, Result)) : Unit {
    Message($"{numbers}");
    Message($"{results}");
}
```

The typical way of calling such a function might look like this:

```
TwoArgumentFunctionWithTuples((100, 200, 300), (Zero, Zero, Zero));
```

With partial application, similarly to what we did before, we may choose to supply one tuple upfront, and create a partially applied function that accepts just one tuple. At that point, because of the singleton tuple equivalence, the new function can be called as if it accepted three top-level parameters, instead of a single three-element tuple - sparing us from the explicit brackets. An example of that is shown next. 

```
let fn = TwoArgumentFunctionWithTuples((100, 200, 300), _);

// both of these work now!
fn((Zero,Zero,Zero)); 
fn(Zero,Zero,Zero); 
```

We can also reach into an even more extravagant example, where nested tuples are used inside tuples. An example of such function definition might be (notice the composition of the `data` parameter, as tuple of tuples):

```
    function TwoArgumentFunctionWithNestedTuples(
    	numbers : (Int, Int, Int), 
    	data : ((String, String), (Result, Result, Result))) : Unit {
        Message($"{numbers}");
        Message($"{data}");
    }
```

In such situation Q# still allows any subset of any of the tuple parameters to be left unspecified in any configuration, which is a very powerful tool. For example we could choose to use a discard in each of the three (one top level, and two nested ones) tuples:

```
let fn1 = TwoArgumentFunctionWithNestedTuples((10, _, 30), (("item1", _), (_, One, Zero)));
```

The resulting partially applied function now takes a single `Int` and a tuple `(String, Result)` as parameters. To take this concept further, we could, for example, supply just the `String` now, thus creating yet another function, which can now be called with two top level arguments only - `Int` and `Result`:

```
let fn2 = fn1(_, ("item2", _));
fn2(20, Zero); 
```

The end result is pretty cool, because we reduced a massive function signature with nested tuples into something very simple to call.

### Usage with built-in function

Partial application allows you to not only massage your own callables into various variants to facilitate specific workflows, but also - naturally - to combine built-in QDK functions and operations in useful and innovative ways. For example, there is a core Q# function `IsCoprimeI(a : Int, b : Int) : Bool`. You could partially apply one of the two parameters to be able to use it as predicate in array filtering. For example, the following code filters an input array of integers based on the fact whether the given element is a co-prime with `5`: 

```
function FilterByFiveAsCoPrime(numbers : Int[]) : Int[] {
    let isFiveCoPrime = IsCoprimeI(_, 5);
    return Filtered(isFiveCoPrime, numbers);
}   
```

### Quantum operations

Of course partial application can be used not just with Q#'s functions, but also with operations. Consider the following operation acting on two qubits, which applies a fixed sequence of gates to them and then prepares an entangled state:

```
operation InitEntangledState(
    q1 : Qubit, q2 : Qubit, 
    q1Preparation : (Qubit => Unit is Adj + Ctl), 
    q2Preparation : (Qubit => Unit is Adj + Ctl)) : Unit is Adj + Ctl {
        q1Preparation(q1);
        q2Preparation(q2);
        H(q1);
        CNOT(q1, q2);
}
```

This is a very basic example, but it shows the general direction in which partial application can be used. We can use this generic operation to create partially applied variants that will be our shortcuts for creating the four Bell states:

```
let initPhiPlus = InitEntangledState(_, _, I, I);
// we can now invoke initPhiPlus(qubit1, qubit2)...

let initPhiMinus = InitEntangledState(_, _, X, I);
// we can now invoke initPhiMinus(qubit1, qubit2)...

let initPsiPlus = InitEntangledState(_, _, I, X);
// we can now invoke initPsiPlus(qubit1, qubit2)...

let initPsiMinus = InitEntangledState(_, _, X, X);
// we can now invoke initPsiMinus(qubit1, qubit2)...
```

A common scenario would be to mark the generic operation as `internal` to hide it, and then expose only the partial variants as visible callables. Notice we can do that using `functions`, even though we are wrapping an `operation`, because partial application has no effect on the quantum state.

```
function InitPhiPlus() : ((Qubit, Qubit) => Unit is Adj + Ctl) {
    return InitEntangledState(_, _, I, I);
}

function InitPhiMinus() : ((Qubit, Qubit) => Unit is Adj + Ctl) {
    return InitEntangledState(_, _, X, I);
}

function InitPsiPlus() : ((Qubit, Qubit) => Unit is Adj + Ctl) {
    return InitEntangledState(_, _, I, X);
}

function InitPsiMinus() : ((Qubit, Qubit) => Unit is Adj + Ctl) {
    return InitEntangledState(_, _, X, X);
}

internal operation InitEntangledState(
    q1 : Qubit, q2 : Qubit, 
    q1Preparation : (Qubit => Unit is Adj + Ctl), 
    q2Preparation : (Qubit => Unit is Adj + Ctl)) : Unit is Adj + Ctl {
        q1Preparation(q1);
        q2Preparation(q2);
        H(q1);
        CNOT(q1, q2);
}
```

Those can now be called in the following way:

```
InitPhiPlus()(qubit1, qubit2);
InitPhiMinus()(qubit1, qubit2);
InitPsiPlus()(qubit1, qubit2);
InitPsiMinus()(qubit1, qubit2);
```

Of course in this simple example, a semantically equivalent result could be achieved by simply adding relevant Bell state specific  overloads to `InitEntangledState`. However, in general, partial application really shines when we are passing around functions as first-class values or when using this methodology to craft new operations from existing ones. One example could be if we needed to use four Bell state preparation callables as oracles conforming to the signature `(Qubit, Qubit) => Unit is Adj + Ctl`.
