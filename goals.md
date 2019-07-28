# goals

"the primary focus of the language will simply be usability.
no weird obsessions with function purity nor some sense of 'simplicity'.

The language should do as much as possible without getting in your way or adding too much unnessessary overhead." ~ Eva

## tech

c frontend for an llvm backend

## strings

this is a top priority. native support for all common string operations as so much of programming ends up being string parsing.

utf8 is nice but its to be added as time permits.

## f strings in c

interpolation into strings supported natively is a massive speed boost so as to avoid the need of messy and error prone format strings
## OOP

no.

## native bignum

no automatic promotion but native support for bignum types.
to allow sane usage this also means that operator overloading will be required.
as a further expansion of this there needs to be a trait system to implement generics without having to resort to something as slow and as clunky as c++ templates.

## namespaces

so many namespaces. `:` is less typing than `::` and works the same so that will be used.

## pointers and deduction

there is no reason to have `->` and `.`when types are known. this should be an automatic deduction from the compiler

```
(pointer+5).thing = 5;
```

as such pointer math can still be supported while wasting less time.

## c abi

if the c abi is supported this would allow all currently existing c libraries to be used granting immediate support while the native implementations are worked on.

```
import System.IO;
...
import C <stdio.h>;
```

it could look like the above


## modules

1 file 1 module. headers dont provide anything we need

## const by default

while it can be annoying it can save you from enough careless mistakes to be worth while. the other factor to consider is that const by default is easier to optimize for.

## type bound functions

while there is no oop being able to call a method for a specific type is very convenient and will be supported. this is just syntactic sugar to allow ease of use. this does not create objects.

## patterns

while regex support sounds nice the standards of which there are many are a mess and as such making chosing one difficult. with the c abi pcre2 can be used but for native support the ideal situation would be to have a context free grammar to achieve a similar purpose.

## higher order math

TODO eva expand uppon this

## LINQ and native sql

its so frequently used native support should be built in.
