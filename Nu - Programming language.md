---
Nu - V1 -> compiler nu: writen in go, translate code in c++17/20 then compile
---

# Nu - Programming language 

---

[TOC]



---

## 0 - Before Starting

One of the goal is to built a programming language that can be adapted if we low level development where memory matters a lot, and another case when we can be more permissif. That's why some functionality of the language can be desable when we said we are working on a low level development. Therefore Project configured with the "low level development" mode won't be able to use Libs or Project without the same mode but the inverse is false.

Another goal is to built a language that whould be able to be compiled and interpreted (when high level) ! It would be also really cool if Nu would be able to use as a script (as high level).

---

There are two way to make code in Nu. By doing a Project or by doing a Script. A Project has a well defined directory tree and can be used from other Nu code; it is meant to be used for middle and big project. A Script is just a Nu file out of the Nu Project tree and it can be reused but can't be used as it; it is meant to be used for little project.

A Nu Project has a specific directory which named the project and has the extension: *.nuproj*. Every code of a Nu Project must be write in a `package`. A `package` is represented by a directory of the `package` name and with the extension: *.pkg*, and all Nu file of that directory must start with: `package` PackageName. It is possible to make `package` inside `package`, the rule is the same, except the files must start with: `package` PackageName.PackageName2. We replace  the '*/*' used in Unix by a '.' like in Java. It is possible to make directory that aren't `package`, they juste mustn't have the extension: *.pkg*.

It is possible to have a ProjectName*.config* in the root of the Nu Project. This file is used to make configuration, for example if you want disable some features or make a "low level development". This configuration file is also used to say if it's a library or an executable. By default for executable, the main function is located in the `package` main. But you need to edit that config file if you want that to be otherwise. The configuration file also allows to not consider some files (if we use them for tests for example). That file allows to make more, but we'll see that at the end of the Nu Presentation.

---

Nu must provide an executable to make at least the compilation, and the Standard of the langage.

## 1 - Types

### 1.1 Types builtin

- `int`: natural number
- `float`: floating point number
- `bool`: `true` or `false` 
- `char`: character
- `string`: an assemblage of `char`
- `auto`: High level type, it is automatically deduce by the runtime and can be change
- `Type`: High level type, it is used to store datas of the types in Nu, and may be use to create dynamic type.
- `Error`: base type for exceptions
- `void`: nothing

It is possible to specify more precisely the types:

- `int<Encoding>`, where Encoding can be: 16, 32, 64 (depending on the machine). By default `int` refer to `int<64>`.

- `float<Encoding>`, where Encoding can be: 32, 64 (depending on the machine). By default `float` refer to `float<64>`.

- `char<Encoding>`, where Encoding can be: ascii, utf-8 (or other encoding that could be added). By default `char` refer to `char<utf-8>`

- `string<Char_T>`, where CharVal refer a type from `char`. By default `string` refer to `string<char>`, which is `string<char<utf-8>>`

### 1.2 Users Type

- typedef:

  ```go
  type Name = BuiltinType
  type Name = BuiltinType: Constraint
  ```

- `enum`:

  ```kotlin
  enum Name {/*...*/}
  enum Name: Types... {/*...*/}
  ```

- `struct`:

  ```swift
  struct Name {/*...*/}
  ```
  
- `interface`:

  ```kotlin
  interface Name {/*...*/}
  ```
  
- `extension`:

  ```swift
  extension TypeName {/*...*/}
  extension TypeName: InterfaceName... {/*...*/}
  ```
  
- `class`:

  ```c++
  class Name {/*...*/}
  class Name: ClassStructOrInterface... { /*...*/}
  ```
  

### 1.3 Derived Type

- `[]T` or `[N]T`: fixed array of N elements of the type T
- `[...]T`: dynamic array of type T's element(s) 
- `[K]V`: map/dictionary from type K to type V
- `*T`: pointer to element of type T
- `T?`: optional value of type T
- `T&`: a rvalue only 

### 1.4 Function Types

- `func[/*capture*/](/*Types Parameter*/) ReturnType`: lambda function with specific scope capture, that take *Types Parameter* in input and return *ReturnType*; both can be empty to match `void`.

- `func(/*Types Parameter*/) ReturnType`: a function or a lambda with scope capture by reference.
- `func!(/*Type and Name Parameters*/)`: a scope

## 2 - Variable, Constant, Function

### 2.1 Variable

```go
var VarName Type					// (1) declaration only
var VarName Type = Value	// (2) declaration with specification of type and assignation 
var VarName = Value				// (3) declaration with inference of type due to the assignation
VarName := Value					// (4) same as (3) but without 'var'
```

In Nu, everything has a default value. Therefore, (1) has a value, even if it hasn't be explicitly initialized. To know if a variable has been initialized or not, all variables can be compare to `default`. if it is `false` the variable has been initialized, and `true` otherwise.

```go
var VarName Type
_ = VarName == default // the expression is true
VarName = Value
_ = VarName == default // the expression is false
```

`_` is a special identifier meaning "unnamed".

A variable declared with (2, 3, 4) will alwayse have an `.inDefault` to `false`.

It is possible to declare multiple variable, with multiple type:

```go
var Name1, Name2 Type12, Name3 Type3												// (1)
var Name1, Name2 Type12, Name3 Type3 = Value, Name4 = Value	// (2)
var Name1 Type1, Name2 = Value1, Value2											// (3)
Name1, Name2 := Value1, Value2															// (4)
```

(1): Name1 and Name2 are type Type12, where Name3 is type Type3.

(2): Name3 has a specific type and is assigned, therefore, all variable after it must be initialized, that the case of Name4 which use inference of type.

(3) it is possible to set the value of all the variable declared before.

(4) same as (3) without `var`.

### 2.2 Constant

```go
const CstName Type = Value
const CstName = Value
```

A constant must be initialized, it is possible to use `.isDefault`, but it will alwayse be `false`.

It is also possible to declare multiple constants, like variables:

```go
const Name1 Type1 = Value, Name2 = Value
const Name1 Type1, Name2 = Value1, Value2
```



### 2.3 Function

```go
func FuncName (Parameter) ReturnType Body
```

A function can have no "Parameter", and no "ReturnType". If no "ReturnType", then the return type is actually `void`.

The "Parameter" are variable like declaration of (1) and (2) exept we don't have to put the word `var`.

```go
func FuncName (P1, P2 Type12, P3 = Value) /*...*/
```

The body can be set through a new scope:

```go
func FuncName () { /* body */ }
```

or via `=>` if there is only one expression, in that case the return type can be deduced:

```go
func FuncName () => /* expression */
func FuncName () Type => /* expression of type Type */
```

To call a function:

```go
FuncName(Argument)
```

where Argument depends on the parameter of the function. Arguments are expression separated by ',' following the Parametter declaration. If a parametter has a default value, it is not necessary to put an argument for it.

It is also possible to set parametter by name:

```go
func FuncName(P1, P2 Type12, P3 = Value) /*...*/
FuncName(P2: V2, V1) // <=> FuncName(V1, V2) <=> FuncName(P2: V2, P1: V1)
```

## 3 - Typedef and RValue

A typedef in nu:

```go
type NewType = BuiltinTypeBased
```

creats a new type and not an `alias` of the type. The "BuiltinTypeBased" must be a builtin type (`int`, `char`, ...) which is not the `bool` type or a typedef.

The type can be constrainted statically by a static boolean expression or (and more specifically xor ) with an rvalue array of the value it can hold. When making constraint it is possible to use binary operator (other than boolean operator and assign operator) with the new type and the base builtin type

For example, in the standard it will have:

```rust
type natural = int: self >= 0 with default = 0
type binary = int: [0, 1] // <=> self == 0 || self == 1 with default = 0
```

Therefore, with the example above:

```go
var a int = 0
var b binary = a	// $$$ Error: a is int but b is binary and there is not implicit cast between int<16> and binary.§
var n natural = -1 // $$$ Error: -1 doesn't respect the constraint of a natural: self >= 0.§
var b binary = 0 // ok c++ -> var<binary> b = 0;
b = 1 + 0 // $$$ Error: (1 + 0) is int but b is binary and there is not implicit cast between int and binary.§
```

In the last line, its fail because as soon as an rvalue is evaluate, a type is given to it. And in lake of information, it will give the simpliest type corresponding in Nu.

To make that line works we should write:

```go
b = binary{1} + binray{0} 
// or
b = 1 binary + 0 binary
// or
b = (1 + 0) as binary // using the cast: cf: 5
```

Below the example that show how it is possible to use builtin type value with operator when making a constraint:

```rust
type test = int: [0, 1, 3] // test = int: self == 0 || self == 1 || self == 3 with default = 0
```

```go
t := test{1} // ok
t += 1 // ok since test allows += int, but it will fail in runtime 'cause 1 + 1 = 2 which is not test
t = 1 // nop, = is the assign operator and types must match
t = 0 test // ok
t += 1 test // ok même au runtime 'cause 0 + 1 = 1 and 1 is in test
```

In Nu it is possible to make `alias` to exesting type:

```C#
alias Alias -> Type
```

In that case, "Alias" and "Type" refer to the same type. That allows to make code easier to read.

example with the same type that will be in the standard:

```C#
alias N -> natural
```

```go
a := 99 N // <=> a := N{99} <=> a := natural{99} <=> a := 99 natural
```



##4 - Structure, Interface and Extension

### 4.1 Structure

In Nu, a structure (`struct`) is a set of variable that may or may not be visible from the outside of the structure.

```c++
struct Name {
  Attribut1 int // public Attribut1 int
}
```

In a `struct`, no need to specify the keyword `var`, as we can't declare otherthing than `var`. 

A `struct` is a type that can be use in the code. Bellow an example:

```c++
struct Position {
  x	float
  y float
}
```

```c#
var p1 Position																				// (1)
var p2 Position = {10, 10.0} // {x: 10, y: 10.0}				 (2) only possible to hide the type name in declaration.
p3 := Position{10, 10.0} // Position{x: 10, y: 10.0}		 (3)
p1.x = 18 float
p1 == default		// false
p1.y == default	// true
```

The (1) is the default value of "_Position_", which set all it's attributes to there default value. In (3) it is a call to the __Constructor__ of the `struct`. The Constructor is what's used to initiate the data structures, and all types avec one. Actually, _`p1`_ in the code is equal to _`Position{}`_; as `var i int` set the variable _`i`_ equal to _`int{}`_. Structure allways have a default constructor that allows to build the `struct` by setting the differents attributes.

The structure _Position_ define above is equivalent in Nu as the declaration bellow:

```swift
struct Position {
  x flaot
  y float
  static init = default
  init() = default
  init(x, y float) = default
  
  delete = default
}
```

 Now there are more to talk.

`static` is a keyword that indicate something is shared by all instance of the same type. It is possible to declare attributes that are commun to all instance of the same type:

```c#
struct ExampleStatic {
  static Target int
}
```

Then to access to the `static` attribute, juste write:

```go
TypeName.StaticAttributeName
```

`init` is the keyword that allow us to refer the Constructor. Constructor are in the frontier between the `static` and the *instance*. It can have many different constructor that take different parametter (it means different signatures and not different paramtter names).

`static init` is a constructor that is call before every constructor. By default it does nothing.

`init()` is the constructor use for the default value of the type. It can't be delete. The use of `default` indicates to let the compiler set the definition of the `init`.

`init() = default` <=> `init() {}`, and `init(x, y float) = default` <=> `init(x , y float) { .x = x; self.y = y}`. The 2nd constructor can be set as `default` because it is with parametter of the same name and type than the attributes. By the way, the keyword `self` allows to refer to the current *instance* it*self*, as much a putting a '**.**' with nothing before.

```swift
struct Position {
  x, y float
  init(y float) = default 	// ok
  init(z float) = default 	// ko: because z is not a attribute name and because there already has a constructor with one float.
}
```

When a constructor takes only one parameter, it is possible to use the "rval typing" (put the type after the value). For example, if the there are not the line: `init(z float) = default` above, we could use "*Position*" like that:

```go
p := 10.0 Position // <=> p := Position{10.0} <=> p := Position.init(10.0)
```

> As the `init` is frontier between the `static` and the *instance*, it is also possible to call it in a `static` way.

Finaly we've see `delete = default`. It is for the __Destructor__. The Destructor is the exact opposit of the Constructor. There is a `static init`, there isn't "`static` `delete`", it is possible to have many `init`, it can only have one `delete`. The Destructor is called when the "*Object*" is deleting itself.

There is one more thing about the `struct`. It is possible to set variable that can't be change from outside the structure, but still be accessed. They are called the `get` attributes. Like a classic attribut they can be `static` or not, but there isn't visibility to indicate as a `get` attribute must be visible from the outside, but as a `const`.

```C#
struct Position {
  get x, y float
}
```

That definition allows the constructor to use _x_ and and _y_ like a classic attribute, but they are seen as constant outside the `struct`. It can be usefull to manage `operators` in `struct`; but w'll talk about `operators` later.

### 4.2 Interface

An `interface` is a set of methods. A method is like a function but linked to an object. An `interfarce` can't be build but it can be used; We'll see that later.

```go
interface Name {
  NameMethod ( TypeParameter ) ReturnType
}
```

No need to prefixed with `func` as it can only hold functions. Also, the body of the function can't be declare here.

With the `interface` it is possible to declare methods that `protected`. It means that they can't be seen from out of the `interface`, but will be usable for anyone who implement this `interface`.

```c#
interface Name {
  protected NameMethod( /*...*/ ) /*...*/
}
```

An `interface` can also inherit from other(s) `interface`.

```go
interface Name: InheritingInterface {
  /*...*/
}
```

Now considering a type that is extended (as we are going to see in **4.3**) and which as the same methods than the methods in an `interface`. Then that type can be explicitly `cast`(see **5**) as an `interface`.

### 4.3 Extension

Nu allows the extension of all the type. Even the builtin ones as long as they are named. With an `extension` it is possible to add attributes and/or methods to a type. But it is not possible to override an attribute nor a method.

It is also not possible to define `operators`.

Here an example:

```swift
extension int {
  	func const sqrt() int {
      /*...*/
    }
}
```

The `const` indicate that the method won't change the value of `self` (the current object)

```go
a := (25 * 25).sqrt() // 25
b := a.sqrt() // 5
c := 25.sqrt() // 5
```

In Nu when a methods doesn't take parameter but `return` a value, we can omit the parentheses if it's not confusing.

```go
a := (25 * 25).sqrt
b := a.sqrt
c := 25.sqrt
```

An `extension` can implement an `interface`:

```swift
interface Math {
  power(x int) Math
  incress(n int)
}

extension int: Math {
  func const power (x int) Math {
    /*...*/
  }
  
  func incress(n int) {
    self += n
  }
}
```

Therefore it is possible to make:

```go
var i1 *Math = new 5 // i1 is an int with the val 5
i1 = i1.power(2) // ok, i1 now has the value 25
i2 := 18	// i2 is int
i1 -> i2 // now i1 is i2
i1.incress(34) // i1 now has the value 42 so it is for i2 !
```

We'll see the operator `->` , `new` and why the value of i2 also change later (cf *Pointer*)!

The only for an `interface` to receive a `const` value, is if all the methods of the `interface` are specified `const` in the type of the value. As seen in the example, the function extension *power* in `int` is specified `const`.

```c#
interface MathOp {
  power(x int) MathOp
  root(x int) MathOp
}
```

```swift
extension int: MathOp {
  func const power(x int) MathOp {
    /* ... */
  }
  func const root(x int) MathOp {
    /* ... */
  }
}
```

```go
const i = 5
var m MathOp = i // valid since m as no way to change the value of i which is const
```

Also, it is not possible to make an `const` of an `interface` type.

## 5 - Cast

### 5.1 Make your cast

As we saw, it is not possible to mixt created type and builtin type... But we also saw, and maybe you didn't notice, an attribute of type `float` receiving a value of type `int`  (in 4.1)! That is possible due to the `cast ` in Nu.

There are 3 kind of `cast`. The `implicit` (as it is for `int` to `float`), the `explicit` (actually the default one when we creating a `type`) and the `delete`.

As said, when a `type` is created, it as an `explicit`  `cast` from and to the builtin based type.

To use the `explicit` `cast` we juste put: `as` after the expression and then place the type to cast to:

```C#
exp as type
```

example:

```C#
type kilogram = float
alias kg -> kilogram
```

```C#
pound := 50 kg						// the RValue is typed as a kg
pound_f := pound as float // pound_f == 50.0
pound_f = 18							// 18 is int, but there is an implicit cast from int to float
```

We can define our `cast`.

```C#
cast TypeSrc as TypeDst {
  /* ... */
} is [[ explicit or implicit ]]
```

To define manually the cast, or

```C#
cast TypeSrc as TypeDst is [[explicit or implicit or delete ]]
```

To let the compiler make the job. The compiler can only do the job in some particular case. If a cast already exist or a path to cast exists, then it is ok.

example:

```C#
type natural = int: self >= 0 with default = 0

cast natural as int is implicit // explicit cast is the default. it will allways work
cast int as natural is implicit // same, but it may fail
  
type positive = natural: self > 0 with default = 1
  
cast positive as int is implicit	// the compiler make: 'positive exp' as natural as int
```

But it is also possible to make `cast` that make special action. For example if we need to manipulate distances and we have kilometres and meters, it would be cool to have a conversion that respect the real conversion.

To do that we need to manually define the cast, in the scope as seen above. In this scope the notion of type is weaker. The value of the type _TypeSrc_  is represented by: `cast.from` and in the opposite, the value returned by the `cast` is represented by: `cast.to`.

example:

```C#
type kilometer = float
type meter = float

alias kilometer -> km
alias meter -> m

cast km as m {
  cast.to = cast.from / 1000
} is explicit
  
cast m as km {
  cast.to = cast.from * 1000
} is explicit
```

```go
distance := 42 km as meter // distance == 42000 meter
```

### 5.2 Nu default casts

Nu as some base `cast `.

- `cast int as float is implicit`

- `cast float as int is explicit`

- `cast int< X > as  int < Y: with Y > X > is implicit`

- `cast int< X > as  int < Y: with Y < X > is explicit`

- `cast int as char is explicit`

- `cast int as string is explicit`

- `cast float as string is explicit`

- `cast bool as string is explicit`

- `cast char as int is explicit`

- `cast int as bool is implicit`

- `cast bool as int is explicit`

- `cast char as string is implicit`

  ---

- `cast [N]T as [...]T is implicit`

- `cast [...]T as [N]T is explicit`

- `cast [N]T as *T is explicit`

- `cast *T as *void is explicit`

- `cast string as []char is emplicit`

- `cast T? as T is delete`

- `cast [int]T as [...]T is explicit`

- `cast [...]T as [int]T is explicit`

- `cast [N]T as [int]T is explicit`

The type `auto` is not a properly spoken a type, therefore it is not possible to make a `cast` with it.

A `cast` from a type T1 to a type T2 can be define only one time by the *owner* of the type T1.

Also, it is possible to `cast` any type to an `interface` where all the methods of the `interface` are implemented by the type. But the `cast` is explicit if the type hasn't be properly described as an implementation of the `interface`.

```go
interface Example {
  method()
}

type exp = int
type exp2 = int

extension exp2: Example {
  func method() => /* ... */
}

extension exp {
  func method() => /* ... */
}
```

```swift
e := 18 exp
E := e as Example // ok, E is Example since exp implement method, therefore E === e 
E -> 42 exp2 // ok and no need cast, now E === a value with 42 exp
```

It is called a *up-cast* as we lose information. It is possible to make a *down-cast* but *down-cast* in Nu are mandatorily `explicit`.

### 5.3 Cast failed ?

It is possible that a `cast` fail at runtime. For example:

```go
type natural = int: self >= 0 with default = 0
```

```swift
i := -10
var n natural = i as natural	// it will fail at runtime since -10 is not a natural
```

We'll see how to handle that when we'll talk about the `try` expression. But for now we can introduce the `is` expression. We'll also see how to make a defined `cast` fail.

```swift
var n natural
if i is natural {
  n = i
}
```

Well, what just happen... First, it is simple to understand `if` statement we do what's inside only `if` a condition is `true`. Then `is`, once again, easy to know in that contest than `is` allows to know if an _**expression**_ `is` a certain type.

But after, why don't we use a `cast`? Spoiler, the `cast` is made by the compiler when the `is` is made. Actually the `cast` is made only because *natural* is more restrictif than `int`. If a value has no link with the type that `is` tests it will fail. Even if a `cast` exist. In that case the only way to be sure it won't `throw` an `Error` is with the `try` exp.

```go
struct Example {
  i int
}
```

```c#
cast Example as int {
  cast.to = cast.from.i
}
```

```swift
var e Example
var i int
if e is int { // $$$ Error: No link between Example and int.§
  /* ... */
}
```

> Actually here it will never fail but it's just for the example.

## 6 - Class

### 6.1 Base

A `class` is a mixt between a `struct` and an `interface`, meaning it has attributs and methods. But what's the point of having a `class` when we simply need to extend a `struct` ? Firstly, a `struct` can only have value `public` or `private`, so if we have `private` data in a `struct` and we extend that `struct`, we won't be able to access those data. Secondly an that's the main reason, because `class` have something more. They have the `sensor`/`signal`.

Unlike the `struct` and the `interface`, `class` must precise the visibilty of theire data. That's why there is one more way to set the visibility, that way is by block of visibility:

```C#
class ClassName {
public:
  /*
  ...
  everything here is public
  ...
  */
private:
  /*
  same for private
  */
protected var a int
// visibility has to be set
}
```

 inside the block of *`public`* no need to specify the visibility, it is as soon as another visibility occurs.

but with the `var` *a*, the visibility apply only for him, and therefore, visibility need to be set after it, even if it's also `public` or `protected`. By the way, a `class` can have both attributs and methods, so it is necessery to put `var` to refer an attribute, and `func` to refer a method. A `class` can also have `const` attributes and `static` methods !

The `signal`/`sensor` is one of the real strength of Nu. It cames from the library *Qt* in *C++*. It is possible for a `sensor` of an object created by a `class`, to `watch` a `signal` that would be `emit` from another `class` object.

```swift
class ClassName {
public:
  sensor SensorName( TypeParameters ) {}
  signal SignalName( TypeParameter ) /* with default = ( DefaultValue ) */
  func something() {
    emit SignalName( ValueOfTypeParameter ) // emit SignalName
  }
}
```

```go
var c1, c2 ClassName
c1.SensorName( TypeParameter ) watch c2.SignalName( TypeParameter )
```

Some explanation.

A `sensor` is like a `func` except it can't `return` something. A `signal` takes *TypeParameter*, but can't **_Named_** those parameters, and a `signal` can have a default value associate to each one of its parameters. `emit` is the keyword used to "send" the signal. It can only be used "inside" the `class` but not in a `static func`. By the way, both `signal` and `sensor` can't be `static`. To "connect" a `sensor` to a `signal`, we use the keyword `watch`. The type parametters of the `signal` and the `sensor` must match. If they don't, the `sensor` must set manually the values.



Here an example:

```swift
class HttpRequest {
private:
  var data string
public:
  init(ipv4 [4]int<8>, port = 3000) { /* ... */ }
  func Send(/* ... */) { /*... emit DataReceived ...*/ }
  signal DataReceived (string) with default = (data)
}

class MyClass {
private:
  var http HttpRequest
  /* attributes */
  sensor OnReception(data string) { /* ... set attributes ...*/ }
public init(http HttpRequest) = default with {
  	.OnReception(string) watch http.DataReceived(string) // <=> self.OnReception ...
  	http.Send(/*...*/)
	}
}
```

In the function *Send*  of *HttpRequest* there is a *coroutine* we'll see that later. Juste know that *Send* won't stop the thread. Then, as soon as the data are received for the object *http*, MyClass will have the data and "set attributes".

In the example we see another way to make *Constructor*. As *http* in the *`init`* have the same type of the attribute of the same name, it is possible to use `= default`. But what if we want do something more in that *`init`* ? That's why there is the `with` keyword

`public init(http HttpRequest) = default with { ...` <=> `public init(http HttpRequest) { .http = http; ...`

### 6.2 Inheritance 

As it is possible to implement an `interface`, it is possible to inherit a `class`. But the inheritance is more complexe with `class`es than with `interface`s. Because `interface` can't have conflict with `var` nor with method name.

When inheriting from another `class`, it must override all methods that are `public` or `protected` in the inherited `class` and it can't change the visibility of them. It may be possible to inherit from 2 (or more) `class` but there are conditions and that point is tricky.

A `class` `extension` is when a `class` inherits from another `class` but don't add any attributes. For multiple inheritance, all `class`es must be `class` `extension` of a same commun `class`.

```C#
class A {
public:
  var val = 23
  func action(a int) => /*... */
protected func action2() => /* ... */ 
}

class A1: A {
public:
  var val = 42
  func action(a int) => A.action(a) // we say it's the same behavior than the definition of A
protected:
  func action2() => /* another behavior */
}

interface I {
  action3()
}

class A2: A, I {
public:
  var val = A.val // we say it's the same default carateristic than A
  func action(a int) => /* another behavior */
  func action3() => /* implementation of the interface I */
protected:
  func action2() => A.action2()
}

class B: A1, A2 {
public:
  var val = 18
  func action(v int) => A1.action(v)
  func action3() => /* ... */
protected:
  func action2() => A2.action()
  var v int
}

```

*A1* and *A2* are both `class` `extension`, cause they don't introduce new `public`/`protected` attributes. As they have the same base `class` (*A*), it is possible to use both in an multiple inheritance.

*B* adds the `protected` attribute *v* then, the `class` *B* and all the `class` whom inherit from *B* won't be usable in multiple inheritance

Basically multiple inheritance looks like inheriting one `class` and some `interface`. But an `interface` can't have `sensor` nor `signal`, therefore, multiple inheritance is mainly used to add `sensor` and `signal`.

### 6.3 Abstract class

An `abstract class` is a `class` which can't be created. Juste like `interface`. They will still be useable, we'll see that later when we'll talk about pointer.

```java
abstract class MyClass {/*...*/}
```

## 7 - Anonymity

### 7.1 Lambda

Lambda are function without name. In Nu no distinction between lambda and closure, we need to specify the capture scope.

```swift
var lambda = func( Parameters ) TYPE => Body
var lambda = func( Parameters ) => Body
var lambda = func( Parameters ) TYPE { /* Body */ }
var lambda = func( Parameters ) { /* Body */ }
```

By default lambda in Nu try to capture the current scope as a reference. But it is possible to change that with `[ = ]` to capture a copie of the scope, `[!]` to not capture the scope. The scope capture must be placed right after the keyword `func`. It is also possible to specify the kind of capture for some variable/constant, and even rename them.

```swift
var a = 42, b = "the answer to the univers", c = "i don't want to be use please"
var lambda = func[&, find = a, c = !](a int) {
  if find == a {
    return b
  }
  return "try again"
}
```

in that case, we capture by reference (`[&]`) the current scope, but we capture *a* as copy, copied in the variable *find*, and we don't capture *c*. Therefore, if we change the value of *b* it will change inside the lambda too, but if we change the value of *a* it won't change in the lambda.

> to specifically capture a variable/constant in reference, we write: `[&VarName]` or `[NewName & VarName]` to capture a variable in copy without changing its name, we can simply do: `[=VarName]`. There is no other way to not capture a variable.

The type of the lambda is the same as the type of a function:

```go
func function(a int) string => /* code */
```

```go
var f func(int)string
f = function // ok as function is a function
f = func(b int) string => /* code */ // ok
```

We call a lambda the same way we call a classic function. The default value of a lambda is an empty function or a function returning the default value of its `return` type.

```go
var f func(int)string, f2 func(int)string => string{} // f and f2 have the same behavior.
```

The only operator available on a function is `()`.

It is possible to call a lambda function right after created:

```go
var v int = func(a int) { /* code returning an int */ }(18)
```

function can be seen as a constant lambda !

```go
package main
var a int
func function(b int) { 
  a = b // ok since a is a global variable 
  /* ... */
}
const f = func(b int) {
  a = b
  /* ... */
}
// f and function have the same behavior. except it is possible to decalre f in a function bu not function.
func main() { // the entry point of the executable
  f(42)
  const f2 = func() => f2(18) // possible
  function(42)
}
```

### 7.2 Structures

Structures, even if more complex than typedef, are simple types. It is possible to make them anonymous.

```go
var Structure struct{ /* struct def */}
```

There is however differencies between the named structures. Anonyme structure can't define `static` data, since `static` refer to the type and not the variable. They can't easer define `get` and the can't be expended. And finaly it is not possible to define `init`.

```go
var point struct{x, y int}
// point == default // true
```

But it is possible to make anonyme `struct` value !

*point* above could have been write:

```go
point := struct{x: int{}, y: int{}}
// point == default // false
```

It is also possible to not named the attributes of an anonyme structure ! To do so, we simply use the special identifier `_` of the Nu, which is an identifier that indicate "unnamed".

```go
var point struct{_, _ int} // point := struct{_: int{}, _: int{}}
```

in that case, to acced the first element of *point*, we simply put *.1* after *point* (same logic for all anonyme attributes).

```go
point.1
point.2
```

That method of using number, works on all structures !

```go
package main
struct Position {
  x, y int
}

func main() {
  var p1 Position
  var p2 struct{a, b int}
  p3 := struct{_: 18, _: 42}
  p1.1 = p3.1
  p2.2 = p1.b + p3.2
}
```

And of course, it is possible to name some attribute but not others (only with anonyme structures).

> Note for me:
>
> in c++ it will only be with .AttributeName, it is at the compile time that the binding is resolved.

## 8 - Multiple value, advanced functions

### 8.1 structure binding

A Structure is just a set of variable. It is possible to bind the attributes of a structure, with variables.

```go
var {a, b}, c = struct{_: 18, _: 2}, 99 // a is binded to 1, b to 2
{a, b}, c := struct{_: 18, _: 2}, 99   // then, a == 18, b == 2 (and c == 99)
```

it is also possible to bind, structure stored in `var` or `const`:

```go
t := struct{e: 31, c: 18}
{a, b}, c := t, 99 // {a, b} will be bind to t, {a --> t.1, b --> t.2}
```

It is possible to have less value in the binder ( *{}* ), but not in the binded (the right side).

```go
t := struct{e: 31, c: 18, m: 23}
{a}, b := t, 42 // {a} --> t; {a --> t.1}
```

As shown, it will bind to the first attribute of the `struct`. If we want only some specific value, we can put `_`  (*unnamed identifier*) in the binder, to tell the compiler we don't want those data.

```go
t := struct{e: 31, c: 18, f: 23, r: 42}
{_, a, _, r} := t // {_ --> t.1, a --> t.2, _ --> t.3, r --> t.4}
```

### 8.2 volatile structure

You thought it was the end of the structures ? It is not. Even if voloatile struture are really special. A volatile structure is a structure with no identifier, that can't be store in a variable nor a constant. It may seems useless, but wait and see.

```go
{beg, end} := struct{39, 42} // {beg --> 39, end --> 42}
```

```go
a := struct{39, 42} // $$$ Error ! can't store a volatile structure
```

Of course it is possible like classic `struct` to bind only some value.

### 8.3 Advanced functions

#### 8.3.1 Multiple return value

To return multiple value through a `func` in Nu, we can `return` an anonymous `struct`.

```go
func F() struct{/*...*/} => // ...
```

In that case, the `return` can be set by calling the constructor, or simply separate value with `,`.

```go
func F() struct{_, _: int} => return {31, 8} // ok
func F() struct{_, _: int} => return struct{_: 31, _: 8} // ok
func F() struct{_, _: int} => return struct{_, _: 18}{31, 8} // ok, same as the 1st

func F() struct{_, _: int} => return 31, 18 // (i) also ok ! it will be bind (but reversely)
```

but in that case, the `return` of the `func` can be stored as one single value of type *nu anonymous struct*. What if we want to `return` multiple value that wouldn't be stored in one sigle `var`/`const`? By using volatil struct.

```go
func example() struct{int, int} => // ...
```

in that case the `return` must be the value separated by `,`.

```go
func example() struct{int, int} => return 23, 4
```

> It is possible to `return` a `struct{Type}` (with only one value), but it is not really useful. 

#### 8.3.2 init the return

So fare we've see only one way to return value from a `func`. Using `return` followed by the values. In Nu it is possible to use `return` as a value ! But it need to be `init`.

```go
func fillArray(from, to int) [...]int {
  return.init // return is usable as a value
  for i := from; i < to; i++ do return.append(i) // for var_init; condition; incrementation do action
  // for i := from; i < to; i++ { return.append(i) } // do allows only one statment, but {} is a new scope.
 	return // return the value.
}
```

It is still possible to place values after the `return`. With that method, consider `return` as much as a *keyword* as a *value*, but in both case it is for the `return` value of a function/method. The only exception for that method is with volatile structure, because they can't be store in `var`, and `return` is basically, a (special) `var`.

#### 8.3.2 contracts with your functions

A contract is something that link the user (of a function) and the developer (of that function). It's a specification that will be checked on runtime, and that require specific state in input, and guarante another state in ouput.

```go
func neg(a int: with a >= 0) int: with [&a] -> a <= 0 => return -a
```

There are a lot to process. The input part (if there is one) comes after all parameters and is separated with `:`. Then `with` indicate the beginning of the input contract. it must be a boolean, juste like typedef constraint. The guarantee comes after the `return` type (if there none it comes after the closing parentheses) also separated with `:`. But then it must capture datas like in lambda. Except here, it doesn't capture anything by default, except for the `return` value stored in the `return` `var`.

Contracts also works with `class`, `extension`, and `interface`. When inheriting from a method that specify contract, the input contract must be the same or more tolerant than the inherited one, and the guarantee must be the same or more restrictive. If two `interface` have the same function but with different contract, it will fail, so it is with multiple inheritance.

Example of a Stack `class` with contract:

```swift
class Stack_int {
private:
  const N = 10
  var arr [N]int
public:
  init(N int) = default
	get MaxLen = N // reminder: get allows to get it, but it can be modify only inside the class
  get Len = 0
  func Push(elem int: with .Len < .MaxLen): with [&.Len, oldLen = .Len] -> .Len = oldLen + 1 {
    .arr[.Len] = elem
    .Len++
  }
  
  func Pop(:with .Len > 0) int: with [&self.Len, oldLen = self.Len] -> .Len = oldLen - 1 {
    return.init
    return = .arr[.Len - 1]
    .Len--
    return
    // .Len--; return .arr[.Len]
    // but i wanted to show the use of return.init
  }
  
  func const Last(:with .Len > 0) => .arr[.Len - 1]
}
```

That way when the user call Push but the stack is full, it will `throw` an `Error` saying that there is a violation of the contract. It is possible to call function or method constant inside the definition of contracts, and even iside the capture scope `[rfunc = function()]` (it is necessary by copy).

## 9 - Auto, Scope, Optional and RValue Only

### 9.1 Optional and try

#### 9.1.1 optional

Optional is a type behavior, meaning it takes an existing type and add some specificity. In that case, it's the possibility to not hold value. The optional types are represented with `?` after the type.

```go
var opt int?
```

The default value of all optional is `nil`, it means: "No value". It is possible to set a value of the optional type classically, but the usage of a `T?` change a little from `T`. `T?` needs to be checked before being used ! Because if a `var` of type `T?` is `nil`, the programme will crash.

```go
var opt int? // opt == nil

opt++ // $$$ Error, need to check opt before using. §
opt?++ // ok, that line do something only if opt != nil

a1 := opt // ok, a is int?
var a2 int = opt // $$$ Error: need to check opt before using it as an int. §
var a2 int = opt? // ok, if opt == nil, then a2 == default is true !

a2 = 10 + a1? // a2 = 10 if opt == nil
```

The operator `?` is called the *ask* operator. It is also possible to put a default value if the optional `var` is `nil`. By calling the *ask or* operator: `??`.

```go

var opt int?
a3 := opt ?? 42 // a1 == opt if opt != nil and 42 otherwise
```

All those operator can be translate by using an `if` expression.

```go
var opt int?

if opt? then opt++ // ok: opt?++

var a2 int
if opt? then a2 = opt // ok: var a2 int = opt?

a1 = 10 + if opt? then opt else int{} // ok: a1 = 10 + opt? <=> 10 + opt ?? int{}

a3 := if opt? then opt else 42 // ok: a3 := opt ?? 42
```

As shown, the operator `?` can be use as a condition. If it's true, then the optional is automatically remove inside the scope of the `if`, `for`, `while`.

#### 9.1.2 try

Now you may think it is not possible to declare an optional with `:=`. But it is. When we talked about `cast`, i mention the `try` exp. `try` is a keyword that allows to avoid all the `Error` that may be thrown in the execution. When used, `try` exp (may) return an optional depending on the type of the expression. I said it *may* because if the expression is type `void`, `try` doesn't return anything.

```swift
opt := try 42 // ok, even if 42 is an expression that will never throw anything, opt is int?
```

Therefore with `cast`, it is possible to write:

```rust
type natural = int: self >= 0 with default = 0
```

```swift
n := (try -10 as natural)?
/* <=>
n := if opt := try -10 as natural; opt? then opt else natural{} 
*/
```

There is another way to make optional; with `if` exp.

```go
opt := if true then expression // no else ! 
```

Now what if we want to `catch` the `Error` thrown so we can adapte the code for failure.

it is with `try` `if catch`. If a `try` expression is used, it is possible to use the `if catch` statement juste after, that will... `catch` the error thrown if there is erreor thrown. But how to `throw` an `Error` ? Simply by tapping: `throw` followed by the message error or `Error(`*message*`)` (or with custom Error `class` that implement the `Error` `interface` builtin of Nu)

```swift
func exampleThrow() {
  throw "error message" // <=> throw Error("error message")
  var str = "this code will never be reached"
}
```

 ```swift
try exampleThrow() // always throw Error
if catch err { // if an error has been caught
  // code goes here
}
 ```

The `if catch` is linked to the last `try` expression written. It is possible however to `try` a scope, it is called the `try` statement (in opposition of the `try` expression).

```swift
try {
  // multiple code line, with potential multiple thrown
}
if catch e {
  // code
}
```

Inside a `try` scope, no `Error` can stop the code. But as soon as there is an `Error`, the following code won't be executed. If a `if catch` statement is after the `try` statement, then the `catch` correspond to the first `Error` thrown (it is the logical deduction).

`if catch` is actually an `if` where `catch` can be considered like a condition. By default, it will be true as soon as an `Error` is detected. But it possible to set it `true` only when some specific kind of `Error` are thrown. It is also to continue with an `else` or even write: `if !catch` to do something if nothing has been caught.

```go
type MyError = string
```

```swift
extension MyError: Error {
  /* ... implementation of the Builtin interface ... */
}
```

```swift
func example(a int) {
  if a % 2 == 0 then throw MyError{"my implem of error"}
  else throw Error{"builtin nu error"}
}
```

```swift
try example(42)
if catch e is MyError {
	// only here when the Error thrown is MyError 
} else if catch e {
  // here when it is any kind of Error that haven't been handle yet.
}
```

This is particularly usefull with `try` statement.

As it is possible to declare `if` variable scope, it is possible to combine `try` exp and `if catch`:

```swift
if v := try exp(); !catch { /* ... */}
```



### 9.2 auto and typeof

#### 9.2.1 typeof

`typeof` is a keyword with 2 function. It return datas about a type and it can be a type.

```c#
t := typeof(42) // t is type Type !
```

All type in Nu are represented with a unique value of builtin `class` `Type`. There are two way to get on of those `Type`.

First by typing the name of the type:

```go
t_int = int // t_int is Type !
```

The other way is with `typeof`.

But `typeof` can also be used as a type !

```C#
var i typeof(42)
```

Even more. All Type value can be used as a real type through `typeof`.

```C#
package main

func main() {
  type_t := typeof(42) // <=> type_t := int
  var i typeof(type_t) // <=> var i typeof(int) <=> var i int
}
```

it is possible to compare two Types:

```C#
if typeof(42) == int { /*...*/ }
```

It is also possible to use it after an `is` or an `as`:

```C#
v := (if 18 is typeof(42) then 18 as typeof(42))? // useless but possible
```

When we talk about optional we saw the `nil` value. In the optional context `nil` is typed by the value or the type of the `var`. But witout that context, `typeof(nil)` is `void` ! 

#### 9.2.2 auto

`auto` is the only way in Nu to make dynamic type. It names `auto` because it adapts automatically to the type it receives. `auto` is not a *real* type, but all value are automatically compatible with `auto`.

```go
var a auto		// default value is nil ! But it's not an optional !
var a2 auto = 18 // ok a1 is int !
var a3 auto = "Nu auto is cool but may be dangerous and less performant" // a3 is string !
```

So far it was also possible to write it without the `auto`. But what `auto` can do, is change it's type !

```go
var a auto		// a == nil and typeof(a) == void
a -> 42				// a == 42 ! and typeof(a) == int !
a -> string		// a == "" but a == default is false, typeof(a) == string !
```

The arrow `->` operator is used to change the type of a `var` `auto`.

It is not possible to define a *cast* from or to `auto`, it is necessarily possible. The `if` `is` and `typeof` can also apply !

```swift
var a auto = 42 // when initializing it not need to use the -> to set the type.
if a is int {
  // here a won't be an auto but it will be an int ! So no operator ->
}
if typeof(a) == int {
  // here a is still an auto, but developer knows a is int (until it changes it.)
}
// type kilograme = int
// alias kg -> kilograme
cast kg as int is {
  cast.to = cast.from * 1000
} is implicit

//cast int as kg is implicit

a -> kg
var b kilograme, c int
b = 18; c = 42;
b = c // FAUX
c = b
b = c as kg
var d int = b + c //--> b and c must be cast in int --> b.operator + (c)
e := b + c // $$$ Error: typeof(b) != typeof(c) and type is not specify. §

struct Pkg1.Matrix {
  mat [...][...]int
  
  init(N, M int) {/*...*/}
  
  operators {
    + (val Matrix) Matrix {
      /* code */
    }
    
    * (val) {
      /* code */
    }
  }
}

extension Pkg1.Matrix {
  func plus(val Matrix) Matrix {/* code */}
}

struct Matrix {
	Pkg1.Matrix
  
  operators {
    + (val) => m.plus(val)
  }
}

interface MatrixOp {
  func dot(val Matrix)
}
extension Matrix: MatrixOp {
  func dot(val Matrix) Matrix { /* code */ }
  func plus(val Matrix) Matrix { /* code */ }
}


var m Matrix
var m2 *MatrixOp = &m
var m2 *MatrixOp = &m as *MatrixOp

b += a
b += a
```

of course functions can use `auto` as a parameter and `return` type, it can be used in `class`, `interface`, `extension`, and in all kind of `struct`. The only way it can't be used is for typedef (or `alias`).

It is possible to use `typeof` type with an `auto` `var` ! In that case the new `var` will be dynamically typed, but it won't be able to *change* like `auto` can do.

```C#
var a auto = function_returning_auto() // we don't know the type in compile time, and it may change !
var d typeof(a) // we don't know the type in compile time ! But the type of d won't change
```

## 10 - Pointer

Here we are, finally. Pointer has always been considered like a difficult subject. Mostly present in C/C++, there are also pointer in Nu. And when we talk about Pointer, we talk about *Memory* and more precisely, *Memory management*. With Pointer we also see the notion of ownership in Nu.

A pointer is basically an address of the memory, where datas are stored. The syntax to get the address of a variable is very simple `&varaible`. Actually it is possible to write: `&(variable)`, in that case the result is the same, but with pointer it is different. The operator `&` is used to take the pointer to, where `&( . )` is take the address of ! The pointer to a classic variable and its address are the same. But not with pointer variable.

Now let's see what happen in a function without pointer.

```go
package main

func inc(v int) {
  v++
}

func main() {
  val := 18
  inc(val)
 	// nothing happen... val still equal to 18
}
```

*val* is still equal to 18 because the `func` *inc* take a *__copy__* of the variable *val*. Then it's not *val* which is modified, but its copy !

As pointer represent value by theirs address, to make it works, we need to use pointer.

```go
package main

func inc(v *int) {
  v++ // the usage of a pointer is the same !
}

func main() {
  val := 18
  inc(val) // $$$ Error: func inc expect *int but int was provided.§
  // we need to pass the pointer to val !
  inc(&val)
 // now val is equal to 19 !
}
```

Pointers in Nu acte like references. Meaning a `*T` value can be use as if it was a value of type `T`.

What's strong with Pointers, is that they can hold dynamic information about a Type. It can hold the Dynamic type of a value. Like we quickly saw with `interface`. It is not possible to *create* a variable of an `interface`. But it is possible to make a pointer to an `interface`!

```go
package main

interface ConsoleDrawable {
  stringDraw() string
}

type c_int = int

extension c_int {
  func stringDraw() string => self as string
}

func main() {
  var console ConsoleDrawable // Error ! ConsoleDrawable can't be init
  var console *ConsoleDrawable // ok
  c := c_int{18}
  console -> &c as *ConsoleDrawable // ok ! we need to cast as c_int doesn't explicitly implement ConsolDrawable.
  // it is possible to write: console.stringDraw()
}
```

But what if we want use an `interface` that not pointe to an existing `var`? The answer is simple, we can "*create*" a pointer. Actually we are asking the computer for us to use some memory.

```go
var console *ConsoleDrawable = new c_int as *ConsoleDrawable
```

The keyword `new` allows to allocate memory. Here are the usage:

```go
new TYPE // like new c_int
new VAL // like new 42 or new c_int{42} or new 42 c_int
```

it is exactly like using a `var` exept we put a `new` before. Note that `new` will return a pointer of the given type. so `new 42` will be typed `*int`.

So we allocated memory. But how to desallocat that memory ? how to give back the memory the computer lend us ? In *Golang* and *Java* there is a *Garbage Collector* that do it for us; in *C*, *C++* it is done manually but (in C++) it is possible to make `class` that manage it for us. Nu is between both. In Nu there is no need to delete manually the pointer. It will be automatically deleted when no access is possible (to the same groupe).

```C#
struct N {
  n *N
}
```

```swift
func main() {
	P := N{}
  P.n -> new N
  P.n.n -> N
}
```

