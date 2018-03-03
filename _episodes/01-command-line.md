---
title: "First session with GAP"
teaching: 30
exercises: 10
questions:
- "Working with the GAP command line"
objectives:
- "Time-saving tips and tricks"
- "Using GAP help system"
- "Basic objects and constructions in the GAP language"
keypoints:
- "Use command line editing."
- "Use autocompletion instead of typing names of functions and variables in full."
- "Use `?` and `??` to view help pages."
- "Premature optimization is the root of all evil."
- "Premature optimization is the root of all evil."
- "Permutations are multiplied from left to right."
- "If a calculation takes too long, press `<Ctrl>-C` to interrupt it."
---

If GAP is installed correctly you should be able to start it. Exactly how
you start GAP will depend on your operating system and how you installed
GAP. GAP starts with the banner displaying information about the version of
the system and loaded components, and then displays the command line prompt
`gap>`, for example:

~~~
┌───────┐   GAP 4.8.5, 25-Sep-2016, build of 2016-10-04 11:28:21 (BST)
│  GAP  │   http://www.gap-system.org
└───────┘   Architecture: x86_64-apple-darwin15.6.0-gcc-6-default64
Libs used:  gmp, readline
Loading the library and packages ...
Components: trans 1.0, prim 2.1, small* 1.0, id* 1.0
Packages:   AClib 1.2, Alnuth 3.0.0, AtlasRep 1.5.1, AutPGrp 1.6,
            Browse 1.8.6, CRISP 1.4.4, Cryst 4.1.12, CrystCat 1.1.6,
            CTblLib 1.2.2, FactInt 1.5.3, FGA 1.3.1, GAPDoc 1.5.1, IO 4.4.6,
            IRREDSOL 1.3.1, LAGUNA 3.7.0, Polenta 1.3.6, Polycyclic 2.11,
            RadiRoot 2.7, ResClasses 4.5.0, Sophus 1.23, SpinSym 1.5,
            TomLib 1.2.5, Utils 0.40
Try '??help' for help. See also '?copyright', '?cite' and '?authors'
gap>
~~~
{: .output}

To leave GAP, type `quit;` at the GAP prompt. Remember that all GAP commands,
including this one, must be finished with a semicolon!

> ## Quit and start GAP
> Practice entering `quit;` to leave GAP, and then start a new GAP session.
{: .challenge}

The easiest way to start trying GAP out is as a calculator:

~~~
(1 + 2^32) / (1 - 2*3*107);
~~~
{: .source}

~~~
-6700417
~~~
{: .output}

We can repeat our calculation from above
if we want to record it as well. Instead of retyping it, we will use `<Up>` and `<Down>`
arrow keys to scroll the *command line history*. Repeat this until you will see
the formula again, then press `<Return>` (the location of the cursor in the command
line does not matter):

~~~
(1 + 2^32) / (1 - 2*3*107);
~~~
{: .source}

~~~
-6700417
~~~
{: .output}

You can also edit existing commands. Press up once more, and then use
`<Left>` and `<Right>` arrow keys, `<Delete>` or `<Backspace>` to edit it and replace
32 by 64 (other useful shortcuts are
Ctrl-A and Ctrl-E to move the cursor to the beginning or to the end of the
line, respectively). Now press the `<Return>` key (at any position of the
cursor in the command line):

~~~
(1 + 2^64) / (1 - 2*3*107);
~~~
{: .source}

~~~
-18446744073709551617/641
~~~
{: .output}

> ## Use the history
> Enter any command into your command line and press `<Enter>`.
> Press `<Up>` to repeat the command and modify it.
{: .challenge}

It is useful to know that if the command line history is long, one can
perform a partial search by typing the initial part of the command and use
`<Up>` and `<Down>` arrow keys after that, to scroll only the lines that begin with
the given string.

If you want to store a value for later use, you can assign it to a name
using ` := `:

~~~
universe  :=  6*7;
~~~
{: .source}

> ## `:=`, `=` and `<>`
>
> * In other languages you might be more familiar with using `=` to assign
>   variables, but GAP uses `:=`.
>
> * GAP uses `=` to compare if two things are the same where other languages might
>   use `==`.
>
> * Finally, GAP uses `<>` to check if two things are not equal, rather than the `!=`
>   you might have seen before.
{: .callout}

Whitespace characters (i.e. spaces, tabs and returns) are insignificant in GAP,
except if they occur inside a string. For example, the previous input
could be typed without spaces:

~~~
(1+2^64)/(1-2*3*107);
~~~
{: .source}

~~~
-18446744073709551617/641
~~~
{: .output}

Whitespace symbols are often used to format more complicated commands for
better readability. For example, the following input creates a 3x3 matrix

~~~
m := [[1,2,3],[4,5,6],[7,8,9]];
~~~
{: .source}

~~~
[ [ 1, 2, 3 ], [ 4, 5, 6 ], [ 7, 8, 9 ] ]
~~~
{: .output}

We can instead write our matrix over 3 lines. In this case, instead of the full prompt
`gap>`, a partial prompt `>` will be displayed until the user finishes
the input with a semicolon:

~~~
gap> m := [[ 1, 2, 3 ],
>        [ 4, 5, 6 ],
>        [ 7, 8, 9 ]];
~~~
{: .source}

~~~
[ [ 1, 2, 3 ], [ 4, 5, 6 ], [ 7, 8, 9 ] ]
~~~
{: .output}

You can use `Display` to pretty-print variables, including this matrix:

~~~
Display(m);
~~~
{: .source}

~~~
[ [  1,  2,  3 ],
  [  4,  5,  6 ],
  [  7,  8,  9 ] ]
~~~
{: .output}

In general GAP functions like `Display` are called using brackets
which contain a (possibly empty) list of arguments.

> ## `Display;`
>
> Check what happens if you forget to add brackets,
> e.g. type `Display;` and `Factorial;`
> > ## Solution:
> > GAP tells us that `Display` is an operation and `Factorial` is a function.
> > These are GAP objects just the same as are lists and matrices.
> > We will explain the difference between operations and functions later.
> {: .solution}
{: .challenge}

Here are examples of some other GAP functions:

~~~
Factorial(100);
~~~
{: .source}

~~~
93326215443944152681699238856266700490715968264381621468\
59296389521759999322991560894146397615651828625369792082\
7223758251185210916864000000000000000000000000
~~~
{: .output}

(the exact width of output will depend on your terminal settings),

~~~
Determinant(m);
~~~
{: .source}

~~~
0
~~~
{: .output}

and

~~~
Factors(2^64-1);
~~~
{: .source}

~~~
[ 3, 5, 17, 257, 641, 65537, 6700417 ]
~~~
{: .output}

Functions may be combined in various ways, and may be
used as arguments of other functions, as we will see later.
They can be defined very compactly using the arrow notation:
~~~
f := x -> 2*x + 18;;
f(3);
~~~
{: .source}

~~~
24
~~~
{: .output}

Note that the `;;` suppresses the output of an expression.

A very time-saving feature of the GAP command-line interface is completion
of identifiers when the `<Tab>` key is pressed. For example, type `Fib` and then
press the `<Tab>` key to complete the input to `Fibonacci`:

~~~
Fibonacci(100);
~~~
{: .source}

~~~
354224848179261915075
~~~
{: .output}

In case when there is no unique completion, GAP will perform a
partial completion, and pressing the `<Tab>` key a second time will display all possible
completions of the identifier.

> ## Tab completion
> Try to enter `GroupHomomorphismByImages` or
> `NaturalHomomorphismByNormalSubgroup` using completion.
{: .challenge}

> ## Always use Tab completion
>
> * It will save you time.
>
> * You will notice immediately whether you misspelled a name.
>
{: .callout}

The way functions are named in GAP will hopefully help you to memorize or even guess names
of library functions. If a variable name consists of several words then the
first letter of each word is capitalized.
Further details on naming conventions used in GAP are documented in the GAP
manual [here](http://www.gap-system.org/Manuals/doc/ref/chap5.html#X81F732457F7BC851).
Functions with names which are `ALL_CAPITAL_LETTERS` are internal functions not intended
for general use. Use them only if you know exactly what you are doing!

It is important to remember that GAP is case-sensitive. For example, the following
input causes an error

~~~
factorial(100);
~~~
{: .source}

~~~
Error, Variable: 'factorial' must have a value
not in any function at line 14 of *stdin*
~~~
{: .error}

because the name of the GAP library function is `Factorial`. Using lowercase
instead of uppercase or vice versa also affects name completion.

Now let's consider the following problem: For a finite group _G_, calculate the
average order of its elements (that is, the sum of orders of its elements divided
by the order of the group). Where to start?

Enter `?Group`, and you will see all help entries, starting with `Group`:

~~~
┌──────────────────────────────────────────────────────────────────────────────┐
│   Choose an entry to view, 'q' for none (or use ?<nr> later):                │
│[1]    AutoDoc (not loaded): @Group                                           │
│[2]    loops (not loaded): group                                              │
│[3]    polycyclic: Group                                                      │
│[4]    RCWA (not loaded): Group                                               │
│[5]    Tutorial: Groups and Homomorphisms                                     │
│[6]    Tutorial: Group Homomorphisms by Images                                │
│[7]    Tutorial: group general mapping                                        │
│[8]    Tutorial: GroupHomomorphismByImages vs. GroupGeneralMappingByImages    │
│[9]    Tutorial: group general mapping single-valued                          │
│[10]   Tutorial: group general mapping total                                  │
│[11]   Reference: Groups                                                      │
│[12]   Reference: Group Elements                                              │
│[13]   Reference: Group Properties                                            │
│[14]   Reference: Group Homomorphisms                                         │
│[15]   Reference: GroupHomomorphismByFunction                                 │
│[16]   Reference: Group Automorphisms                                         │
│[17]   Reference: Groups of Automorphisms                                     │
│[18]   Reference: Group Actions                                               │
│[19]   Reference: Group Products                                              │
│[20]   Reference: Group Libraries                                             │
│ > > >                                                                        │
└─────────────── [ <Up>/<Down> select, <Return> show, q quit ] ────────────────┘
~~~
{: .output}

You may use arrow keys to move up and down the list, and open help pages by
pressing the `<Return>` key. For this exercise, open `Tutorial: Groups and Homomorphisms`
first. Note the navigation instructions at the bottom of the screen. Look at the
first two pages, then press `q` to return to the selection menu. Next, navigate to
`Reference: Groups` and open it. Within the two first pages you will find the
function `Group` and a mentioning of `Order`.


Let's now copy the following input from the first example of the GAP Reference
Manual chapter on groups. It shows how to create permutations and assign values
to variables. This is `Reference: Groups`. You can select it by writing `?11`, where
you replace `11` with whatever number appears before `Reference: Groups` on your machine.

If you are viewing the GAP documentation in a terminal, you might find it helpful to
open two copies of GAP, one for reading documentation and one for writing code!

> ## The Help System
>
> * Use `?` and `??` to view help pages.
>
> * After this workshop, you can read the `A First Session with GAP` section in the GAP Tutorial.
>
> * You can view the documentation in the terminal, [online](https://www.gap-system.org/Doc/manuals.html), or download it as
>   a [pdf file](https://www.gap-system.org/Doc/manuals.html).
{: .callout}

This guide shows how permutations in GAP are written in cycle notation, and also
shows common functions which are used with groups.

~~~
a := (1,2,3);; b := (2,3,4);;
~~~
{: .source}

Next, let `G` be a group generated by `a` and `b`:

~~~
G := Group(a, b);
~~~
{: .source}

~~~
Group([ (1,2,3), (2,3,4) ])
~~~
{: .output}

We may explore some properties of `G` and its generators:

~~~
Size(G); IsAbelian(G); StructureDescription(G); Order(a);
~~~
{: .source}

~~~
12
false
"A4"
3
~~~
{: .output}

Our next task is to find out how to obtain a list of elements and their orders.

> ## `?elements`
> Enter `?elements` and explore the list of help topics.
{: .challenge}

After its inspection,
the entry from the Tutorial does not seem relevant, but the entry from the
Reference manual is. It also tells us the difference between using `AsSSortedList`
and `AsList`. So, this is the list of elements of `G`:

~~~
AsList(G);
~~~
{: .source}

~~~
[ (), (2,3,4), (2,4,3), (1,2)(3,4), (1,2,3), (1,2,4), (1,3,2), (1,3,4),
  (1,3)(2,4), (1,4,2), (1,4,3), (1,4)(2,3) ]
~~~
{: .output}

The returned object is a _list_. We would like to assign it to a variable
in order to explore and reuse it. We forgot to do this when we calculated it. Of
course, we may use the command line history to restore the last command, edit
it and call again. But instead, we will use `last` which is a special variable
holding the last result returned by GAP:

~~~
elts := last;
~~~
{: .source}

~~~
[ (), (2,3,4), (2,4,3), (1,2)(3,4), (1,2,3), (1,2,4), (1,3,2), (1,3,4),
  (1,3)(2,4), (1,4,2), (1,4,3), (1,4)(2,3) ]
~~~
{: .output}

This is a list. Lists in GAP are indexed from 1.
The following commands are (hopefully!) self-explanatory:

~~~
gap> elts[1]; elts[3]; Length(elts);
~~~
{: .source}

~~~
()
(2,4,3)
12
~~~
{: .output}

We can access elements of lists and we can also dynamically change the length of a list by adding new elements:

~~~
L := [3,4];;
L;
L[1] := 2;;
L;
Add(L, 3);;
L;
Append(L, [7, 8, 9]);;
L;
~~~
{: .source}
~~~
[ 3, 4 ]
[ 2, 4 ]
[ 2, 4, 3 ]
[ 2, 4, 3, 7, 8, 9 ]
~~~
{: .output}


Note that a list in GAP is not necessarily dense, i.e. it may contain holes:
~~~
[3,,4]
~~~
{: .source}
This is a list of length 3!

An important difference to statically-typed languages you might know is that the elements of a list
need not be of the same type:

~~~
[3, [1,2,3], (4,5)(2,3)]
~~~
{: .source}


> ## Lists are more than arrays
>
> * May contain holes or be empty.
>
> * May dynamically change their length (with `Add`, `Append` or direct assigment).
>
> * Not required to contain objects of the same type.
>
> * See more in [GAP Tutorial: Lists and Records](http://www.gap-system.org/Manuals/doc/tut/chap3.html).
{: .callout}

Many functions in GAP refer to `Set`s. A set in GAP is just a list with
no repetitions, no holes, and elements in increasing order. Clearly, this only works
if GAP knows how to compare the elements.
Here are some examples:

~~~
gap> IsSet([1,3,5]); IsSet([1,5,3]); IsSet([1,3,3]);
~~~
{: .source}

~~~
true
false
false
~~~
{: .output}

We continue with our example -- the average order of elements
of `G`. There are many different ways to do this, we will consider a few of them
here.

A `for` loop in GAP allows you to do something for every member of a collection.
This general form of a `for` loop is:

~~~
for val in collection do
  <do something with val>
od;
~~~
{: .source}

For example, to find the average order of our group `G` we can do.

~~~
s := 0;;
for g in elts do
  s  :=  s + Order(g);
od;
s / Length(elts);
~~~
{: .source}

~~~
31/12
~~~
{: .output}

Actually, we can just directly loop over the elements of G, in general GAP
will let you loop over most types of object. We have to switch to using `Size`
instead of `Length`, as groups don't have a length!

~~~
s := 0;;
for g in G do
  s  :=  s + Order(g);
od;
s / Size(G);
~~~
{: .source}

~~~
31/12
~~~
{: .output}

There are other ways of looping, for example we can instead loop over the integers,
and use `elts` like an array:

~~~
s := 0;;
for i in [1 .. Length(elts)] do
  s  :=  s + Order(elts[i]);
od;
s / Length(elts);
~~~
{: .source}

~~~
31/12
~~~
{: .output}

We can state this in a much more compact way as we will now see:
GAP has very helpful list manipulation tools.
Here we use the fact that functions are objects in GAP and so they
can also be an argument of a function.
`List(L, F)` returns a newly created list where the function `F` is applied to each
member of the list `L`.

~~~
f := x->2*x+18;;
l := [1..5];;
List(l, f);
l;
~~~
{: .source}

~~~
[ 20, 22, 24, 26, 28 ]
[ 1 .. 5 ]
~~~
{: .output}

Note that this does not change `l`.
We now use this to state our computation concisely:
~~~
Sum(List(elts, Order)) / Length(elts);
~~~
{: .source}

~~~
31/12
~~~
{: .output}

Note that `Sum` takes a list as its argument and returns the sum of its entries.

Using `List` to create a copy instead of changing the given list is called a
_functional programming_ style.
Functional programming refers to the idea that the result of a function
_only_ depends on the values of its arguments and does not change _any_ variables
but returns a new object.
This makes programs much more safe to use and to understand.
When writing new code you should always prefer elegance and understandability to
performance.
To say it with Donald Knuth:
> ## Premature optimization is the root of all evil
>
> >Programmers waste enormous amounts of time thinking about, or worrying about, the speed of noncritical parts of their programs, and these attempts at efficiency actually have a strong negative impact when debugging and maintenance are considered.
> We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil.
> Yet we should not pass up our opportunities in that critical 3%.
>
> [Donald Knuth](https://en.wikiquote.org/wiki/Donald_Knuth)
{: .callout}

> ## Functional programming
>
> * Functions do not have side-effects.
> * In other languages the `map` command is an analogue to GAP's `List`.
> * Can be very elegant but nested `List` statements quickly become unreadable.
>   Choose wisely!
{: .callout}

Note that for many list operations there are both functions that create a new list and
functions that change its first input.
E.g.

~~~
L := [2, 4, 3];;
Concatenation(L, [7, 8, 9]);
L;
~~~
{: .source}

~~~
[ 2, 4, 3, 7, 8, 9 ]
[ 2, 4, 3 ]
~~~
{: .output}

~~~
Append(L, [7, 8, 9]);
L;
~~~
{: .source}

~~~
[ 2, 4, 3, 7, 8, 9 ]
~~~
{: .output}

> ## Functional programming in GAP
>
> Convention:
> * Names of functions with side effects are verbs.
>
> * Names of functions without side effects are nouns or adjectives.
{: .callout}

> ## The `->` constructor
>
> * Does the `->` constructor for functions fit into the functional programming
>   paradigm?
>
> > ## Solution:
> > Yes. E.g. the function `f := x -> x^3` does not change its input.
> {: .solution}
{: .challenge}

Let's consider another tool to manipulate lists. Often we need to get all elements
from a list that satisfy a certain condition. For example we might need a list
containing all even numbers between 1 and 20. This is done by the commmand `Filtered`,
where `Filtered(L, F)` is the list containing all elements `l` of `L` for which `F(l)`
is true.

~~~
Filtered([1..20], IsEvenInt]);
~~~
{: .source}

~~~
[2,4,6,8,10,12,14,16,18]
~~~
{: .output}

We study some more methods to get information from lists.

* finding elements of `G` having no fixed points:

~~~
Filtered(elts, g -> NrMovedPoints(g) = 4);
~~~
{: .source}

~~~
[ (1,2)(3,4), (1,3)(2,4), (1,4)(2,3) ]
~~~
{: .output}

* finding a permutation `g` that conjugates (1,2) to (2,3)

~~~
First(elts, g -> (1,2)^g = (2,3));
~~~
{: .source}

~~~
(1,2,3)
~~~
{: .output}

Let's check this:

~~~
(1,2,3)^-1*(1,2)*(1,2,3)=(2,3);
~~~
{: .source}

~~~
true
~~~
{: .output}

> ## From left to right
>
> * In GAP permutations are applied from the right and thus multiplied from left to right!
{: .callout}

Finally, there are the functions `ForAll` and `ForAny` that work just like the quantifiers of the same name:
* checking whether all non-identity elements of `G` move at least 2 points:

~~~
ForAll(elts, g -> g=() or NrMovedPoints(g)>=2);
~~~
{: .source}

~~~
true
~~~
{: .output}

* checking whether there is an element in `G` which moves exactly two points:

~~~
ForAny(elts, g -> NrMovedPoints(g) = 2);
~~~
{: .source}

~~~
false
~~~
{: .output}

> ## List operations
>
> Use list operations to solve the following:
>
> * Select from `elts` the elements that stabilise the point `2`.
> * Select from `elts` the elements that centralise the permutation `(1,2)`.
> * Find the number of elements in `elts` of order `3`.
> * Does `G` contain an element of order `5`?
>
> > ## Solutions:
> > * `Filtered(elts, g -> 2^g = 2);`
> > * `Filtered(elts, g -> (1,2)^g = (1,2));`
> > * `Length(Filtered(elts, g-> Order(g)=3));`
> > * `ForAny(elts, g-> Order(g)=5);`
> {: .solution}
{: .challenge}

