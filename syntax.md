# NeOC Syntax Draft v0.1
## Introduction
NeOC is a C-like language focusing on usability, performance, and where possible, safety. NeOC borrows a lot from modern languages such as Rust, C#, Haskell, and many others. However unlike most of the afformentioned languages NeOC is from the C-programmer perspective of actually needing to get things done.

NeOC contains no GC, and as such memory management is manual. This allows for a much greater scope of where NeOC programs can run, such as embedded and realtime applications. However the introduction of the linear pointer offloads much of the memory management onto the compiler while ensuring memory safety.

## Basic Syntax
NeOC primarily consists of a series of statements ending with a semicolon (`;`). Statements consist of keywords, identifiers, and operators. 

## Basic File Structure

The top level of a NeOC source file contains the following:
 - module import statements
 - module declaration
 - global variables & constants
 - data structures and type definitions
 - functions

## Type System
NeOC utilizes a trait-based type system. It consists of two fundemental concepts: Types, and Typeclasses. A type represents some form of object; namely structs, variants, enums, and functions, as well as primitive types such as integers. A typeclass (more commonly known as a trait) represents one or more behaviors a type exerts. Typeclasses are the basis for generics in NeOC.

There are two kinds of types in NeOC: abstract and concrete.
A concrete type directly represents some data. Examples include primitive types like `i32` and `char`.
In contrast an abstract type represents a layout of data, but some or all of the data is undefined. An example is `Maybe<T>`. 

The primary behavior a typeclass can specify are functions that a type must implement. However there exists a class of "magic typeclasses" which, when applied to a type, causes it to exert behavior not normally seen. By far the most prolific example you'll come across is the `Ptr` (Pointer) typeclass, implemented by the `ptr` data type.

## Primitive Types
| Name   |    Size    |       Description                               |
|:------ | ---------- | ----------------------------------------------- |
| `char` |   1 byte   | Represents a single ascii character. Unsigned   |
| `bool` |   varies   | Represents True or False. Size dep. on platform |
| `i8`   |   1 byte   | Signed Integer.                                 |
| `i16`  |   2 bytes  | Signed Integer.                                 |
| `i32`  |   4 bytes  | Signed Integer.                                 |
| `i64`  |   8 bytes  | Signed Integer.                                 |
| `u8`   |   1 byte   | Unsigned Integer.                               |
| `u16`  |   2 bytes  | Unsigned Integer.                               |
| `u32`  |   4 bytes  | Unsigned Integer.                               |
| `u64`  |   8 bytes  | Unsigned Integer.                               |
| `f32`  |   4 bytes  | Single-Precision Floating Point Number.         |
| `f64`  |   8 bytes  | Double-Precision Floating Point Number.         |
| `isize`|   varies   | Unsigned Integer. Size dep. on platform         |
| `ptr`  |   varies   | Pointer to object. Size dep. on platform        |

## Module Declaration And Imports
The beginning of a NeOC source files begins with the following, in order: 
 - Zero or more `import` statements.
 - One `module` declaration.

The import statement consists of the `import` keyword followed by the full name of the module to be imported. (exception for the `import c <>` statement, see later.)

The module declaration consists of the `module` keywork followed by the full name of the current module. Additionally that may be followed up by the `export` keyword and in parentheses the list of symbols to made available outside of the module.

A typical example may look like this
```
import Std.Data.ArrayList;
import System.IO;

module NeOC.Example export (do_something, do_something_else);
```

where `do_something` and `do_something_else` are names of functions.

## Function Syntax

A function declaration consists of the return type, followed by the name of the function (including the namespace it belongs to) followed by in parentheses the list of arguments, followed by any type constraints & generic type declarations, and a opening curly bracket denoting the beginning of the function block.

The following defines a function named "do_something" that takes a 32bit integer called "foo", a single precision floating point number called "bar", and returns a 32bit integer.
```
i32 do_something(i32 foo, f32 bar) {
    ...
}

```

alternatively the keyword `void` can be used in place of the return type to denote the function not returning anything.

To denote the namespace a function belongs to, prefix the name of the function with the namespace name followed by `:`, as so:

```
i32 mynamespace:do_something(i32 foo, f32 bar) {
    ...
}

```

To specify a generic function, you replace the type's of the variables you want to be generic with a temporary identifier, and after the argument parentheses you use the `=>` operator followed by the list of names you used for your generic types, comma seperated, and you also include constraints here as well.

For example, the declaration of the function that compares if two things are equal, would look like this

```
bool compare(A thingOne, A thingTwo) => Eq A {
    ...
}
```
Notice that the same type identifer was used for both arguments; this means that both MUST be the same type.
after the `=>` we specified that `A` must be of the typeclass `Eq` (Equatable). If you wanted to specify a function that takes two arguments of any type, you may do something such as the following:

```
bool compare(A thingOne, B thingTwo) => A, B {
    ...
}
```
Notice that the generic type names must be listed after the `=>` even if no constraints are specified.

The return type may also be of a generic form.

## Data Structures
### Struct

The most fundemental user-defined datatype is the venerable `struct`.

Structs define a wrapping of one or more other data fields into a common supertype.
A struct declarations consists of the `struct` keyword followed by the name of the struct, then optionally any generic types and their constraints, then a new block. Within that block you define a series of fields by specifying the type followed by the field name and a semicolon.

```
struct Foo {
    i32 bar;
    f32 baz;
}
```
defines a struct with two members, a 32bit int "bar", and a float, "baz".

To declare a generic struct, you follow the name by a par of `<>` containing the type parameters, and optionally a `=>` followed by constraints. Note: unlike a function, the `=>` is not required if you aren't specifying any constraints.

For example, a struct that takes 1 type parameters, `T`:

```
struct Foo<T> {
    T bar;
    f32 baz;
}
```
or two with a constraint:
```
struct Foo<T, E> => Ord T, Bounded E {
    T bar;
    E baz;
}
```

#### Breakout Syntax

For convenience with smaller structs, there is a shortform called the "breakout". After the struct name, instead of following it up with a block, in parentheses you list the types, and optionally name, of the contents as if it was a function. For example:

```
struct Foo(i32 bar, f32 baz);
```
Is equivelant to the first struct example.
However the names are optional, and if omitted will be given a simple name of the form "first", "second", "third" etc.

Generics are also valid, of the form
```
struct Foo<T, E>(T, E);
```

This allows the struct to be initialized with `Foo(Some, Value)` as if it was a function.
If not using the breakout syntax a default initializer is created which is just all of the members in order.

If not using the breakout syntax a different order of elements in the initializer can be specified with `this(...)`:
```
struct Foo {
    this(else, something, another);

    i32 something;
    i32 else;
    i32 another;
}
```

Outside of structs, the breakout syntax can be used to both set and retrieve values from a struct. 

```
struct Foo(i32, String);
...

var f = Foo(42, "Hello, World!");

var Foo(i, str) = f;
// do something with variables i and str
```

### Variants
Also known as Tagged Unions, a variant is a type whos contents is one of multiple possabilities.
The syntax is similar to that of structs, however inside them they are split into subtypes, of the form `name:`. For example, a variant which could contain either a `i32` or `f32`:
```
variant Foo {
Bar:
    i32 integer;
Baz:
    f32 float;
}
```
*note: overlapping names between subtypes are allowed.

Which could be utilized as
```
Foo:Baz b;
b.float = 1.24;
Foo f = b;
...
Foo f;
if (f is Foo:Baz) {
    Foo:Baz b = (Foo:Baz)f;
    ...
}
```

Generics can also be used
```
variant Foo<T> => Integral T {
Bar<T>:
    T i;
Baz:
    i32 x;
}
```

Additionally the breakout syntax is valid and counts as a subtype, as so:
```
variant Foo {
    Bar(i32);
    Baz(f32);
}
```
The two syntaxes can be mixed:
```
variant Foo {
    Bar(i32);
Baz:
    f32 float;
}
```
and the `this(...)` can be used to specify an initializer for a subtype.

### Unions
unchanged from C.

### Enums
unchanged from C except members are within the implicit namespace of the enum.

# Bringing Them Together
You can associate a function with a particular struct or variant.

Say we have a struct, `Foo`:
```
struct Foo {
    i32 i;
    f32 f;
    String str;
}
```
Let's say we have a function that takes a `Foo` and returns a boolean:
```
bool does_something(Foo f) {
    ...
} 
```
We can use the following syntax to associate it with the struct:
```
bool Foo:does_something(this Foo f) {
    ...
}
```

It can now be called with
```
Foo f = ...;

bool b = f.does_something();
```

Note: putting the function inside the structs namespace is not required but recommended.

# Statements And Expressions

## Variables And Assignment

Declaring a variable takes the form
```
mut type name;
```
```
[mut] type name = expression;
```

By default in NeOC, all variables are immutable, to specify a variable as being mutable, append the `mut` keywork before the type during declaration. Immutable variables MUST be declared and assigned in the same statement.

Mutable variables can be later assigned with
```
mut i32 i;
...
i = expression;
```

If you are assigning and declaring in the same statement, you may skip typing out the type name and instead substitute it for the `var` keyword.

For example:
```
var i = generateInt();
```

Variables are bound to the scope in which they are declared. 

## Expressions

An expression is a series of one or more operations that yield some value.

An expression can contain one or more of the following: function calls, variable identifiers, literals, and operators (technically they are also function calls). And there are some keywords that may be used inside an expression depending on context.

```
i32 i = 9;
i32 x = 3 * i + get_an_int(i);
```

Additionally certain statements can act as expressions under certain circumstances.
Specifically, `if-else`, `if-elseif-else`, and `match` statements can be used as expressions if the final line in each block is an expression that all return the same type.

```
i32 i = if(something == something else)
            5;
        else
            6;
```

## Specific Statements
### `if`, `if-else`, `if-elseif-else` ...
Pretty much untouched from C.

### Switch Statement
ditto. Except the `case` values are automatically scoped to the type being switched against, if applicable.

### Match Statement

Similar to a switch, however it can match types against types, and any type that implements the Eq typeclass. Also has syntactic surgar for the breakout syntax and the `case` keyword isn't used. Additionally there is no fall-through.

Examples
```
String str = "blah blah";
...
match(str) {
    "something":
        do_something();
    "blah blah":
        do_something_else();
    default:
        die();
}
```
```
variant Foo {
    Bar(i32);
    Baz(String);
}
...
Foo f = Foo:Bar(42);

match (f) {
    Bar(N):
        do_something(N);
    Baz(str):
        printLn(str);
}
```

### For Loop
Same as C.

### While Loop
Same as C.

### Do...While Loop
Same as C.

### Foreach Loop
The `foreach` loop allows a convenient way to iterate over some collection.

```
mut String[] strings = ...;

foreach(String str in strings) {
    ...
}
```
The foreach loop can be used on any concrete type implementing the `Enumerable` typeclass.

# Pointers, References, And Memory Management

## Standard Pointers & Allocation

To define a variable as pointer, you append `*` to the type.
Memory allocation is done with the `new` keyword followed by a concrete type. Additionally `malloc` is also available.

```
String* str = new String;
```

This does in fact change the type of the variable to `ptr<String>` however `ptr` is an instance of the magic typeclass `Ptr` which grants some 'magical' behavior, specifically it is automatically dereferenced as needed.

However because operators are implemented as functions, pointer arithmetic cannot be done normally as it'll try to derefence the pointer and apply the operator function.

Instead, to perform pointer arithmetic you must append `*` to the pointer name, as so:
```
char* chars = malloc(sizeof(char) * 2);
char fst = chars*;
chars* = chars* + 1;
char snd = chars*;
```

To free a pointer, you use the `delete` keyword, as so:
```
char* chars = malloc(sizeof(char) * 2);

delete chars;
```

There is no difference between how `malloc` and `new` allocate memory, so you may use them interchangably. The difference is syntactical.

Passing a pointer to a function is just how you'd expect:
```
void foo(char* chars, i32 len) {

}
```

## Linear Pointers
Linear pointers are the primary pointer you will be utilizing on your adventures.

The syntax with them is similar to regular pointers except you use `~` instead of `*`.
Additionally pointer arithmetic is unavailable.

```
i32~ i = new i32;
```

Linear pointers are special in that they MUST be consumed once and only once. A linear pointer is considered 'consumed' when it is either deleted, goes out of scope (where its deleted automatically) or has ownership transferred.

A function may accept a linear pointer in one of two ways: take ownership or borrow.
At any given time a linear pointer will 'belong' to only 1 thing. Usually a function but it can also belong to another data structure.

A pointer may be implicitly converted to a linear pointer, however convertion of a linear pointer to pointer requires an explicit cast.

The type of pointer a variable delcared with `var` is can be overridden as so:
```
var* i = new i32;
var~ x = ...;
```

### Ownership

For a function to accept a linear pointer and take ownership, you use the following syntax:
```
void take_ptr(char~ chars) {
    // chars is now ours!
}
```
By having ownership of a linear pointer, it is now that functions responsability to free the `chars` pointer - or alternatively transfer ownership somewhere else.

Because consumption must be guaranteed to happen once and only once, a linear pointer may be implicitly consumed in statements.

Consider the following example:
```
void example(bool free) {
    i64~ i = new i64;

    if(free == true)
        delete i;

    i64 x = i + 1;
}
```

This will fail to compile because `i` goes out of scope at the `if` regardless if `free` equals true or not.

Similarly with if-else and ownership transfer

```
void take_ownership(i64~ i) {
    ; // implicitly free'd when functions returns
}

void example(i64~ i, i32 x) {
   i64~ i = new i64; 

   if(x == 1) {
       take_ownership(i);
   } else if (x == 2) {
       do_something();
       // i implicitly deleted here
   } else {
       do_something_else();
       // i implicitly deleted here
   }
}
```

A function may return a linear pointer, where the caller takes ownership.
Additionally data structures may contain linear pointers, to where the structure itself is the owner of the pointer.

```
struct Foo {
    char[100]~ array;
}
```

To take ownership back from a structure, you must destruct it with the breakout syntax. (subject to change)

### Borrowing

As an alternative to taking ownership, a function can borrow a linear pointer instead.
The syntax to do that is
```
void example(char[]& array) {
    // ...we've borrowed array! 
}
```
When borrowing a linear pointer, you cannot perform ANY operation that 'consumes' it. This includes passing it to a function that takes ownership, deleting it, assigning it to another variable, or packing it into another structure. It also is not automatically deleted when it goes out of scope.

You can however make borrowed references to linear pointers. This is essentially an alias however.
```
i32~ i = new i32;
i32& x = i&;

// both i and x are in scope!
// but if i goes out of scope than so does x.
```

# Typeclasses
To create a new typeclass, you use `class` keyword followed by the name of the typeclass, followed by any type parameters and constraints, than optionally a new block containing a list of function prototypes any implementor must implement.

The following is an example of how one may declare the `Eq` typeclass.
```
class Eq a {
    bool operator==(a, a);
    bool operator!=(a, a);
}

```

To make a type part of a typeclass, you use the `instance` keyword followed by the typeclass name and name of the function you want to make an instance of it. You are than required to make sure all functions are implemented or you'll get a compiler error.

```
struct CustomNumber { ... }

instance Eq CustomNumber;

bool CustomNumber:operator==(CustomNumber a, CustomNumber b) {
    ...
}
bool CustomNumber:operator!=(CustomNumber a, CustomNumber b) {
    ...
}
```

# Miscellaneous

## Lambdas & Closures

The syntax for declaring a lambda is as follows:
```
Fn<i32, i32, i32> lambda = { i32 x, i32 y -> x + y};
```
The contents of a lambda may be either a function or an expression.

If it is an expression an internal semicolon isn't required.

If the lambda references any outside variable, it becomes a closure and all references are copied at the time of lambda declaration.

If a lambda is 'borrowed' with the `&` syntax, that means it can only either be executed or passed to other functions that borrow it.

This allows the lambda to be inlined at compile time in cases that a dynamic reference is not nesssessary. Additionally it is a closure that is passed as an argument to a function that borrows it, making a copy of the references isn't nessessary.

If a lambda is passed after a function is called, like so:
```
mapOver(listOfThings) { i32 x, i32 y ->
    x + y;
};
```

that means it is passed as the last argument to the function, and is equivalent to
```
mapOver(listOfThings, { i32 x, i32 y -> x + y});
```

## String Interpolation

Prefixing a string literal with `$` makes it an interpolated string.
Implemented as a closure that executes immediatly and returns a string, an interpolated string provides an easy, convenient, and performant way to format strings.

```
i32 i = ...;
f32 f = ...;

println($"Value of i is {i} and the value of f is {f}.");
```
To apply a format to an interpolated value, append a double colon and the format specifier.

```
println($"{someFloat::0.##} is a float rounded to two decimal places!");
```

Interpolated strings create a string of type `String~` unless casted to a c-string.

### String Interpolation Format Specifiers
TBD

## Boxed and Unboxed Types

When you declare a type such as `char[]`, unlike what you may have expected, you didn't declare a raw array, instead you delared a `Std:Array<char>`. This is a "boxed" type.

To declare an unboxed type, append `#` to the type, such that `char[]#` represents a raw array.

## Ranges

You may declare a `Range<T>` through the syntactic surgar of `[from..too]` for inclusive, and `[from...too]` for exclusive ranges. `Range<T>`, among others, implements the `Enumerable` typeclass so it can be used from a `foreach` statement:

```
foreach (var i in [1..10]) {
    ...
}
```

which is equivelant to
```
foreach (var i in Range(1, 10, 1, true, true)) {
    ...
}
```

The type of `Range<T>` is

```
struct Range<T> => Eq T, Ord T {
    this(start, end, step, inclusive, countUp);

    T start;
    T end;
    T step;
    bool inclusive;
    bool countUp;
}
```

## Comments

Standard C/C++ style comments

`// ... comment` for single line comments.
`/* ...comment ... */` for block comments.



