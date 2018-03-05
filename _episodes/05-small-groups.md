---
title: "Small groups search"
teaching: 40
exercises: 15
questions:
- "Modular programming: putting functions together"
- "How to check some conjecture for all groups of a given order?"
objectives:
- "Using Small Groups Library"
- "Designing a system of functions to fit together"
keypoints:
- "Organise the code into functions."
- "Create small groups one by one instead of producing a huge list of them."
- "Using `SmallGroupsInformation` may help to reduce the search space."
- "GAP is not a magic tool: theoretical knowledge may help much more than brute-force approach."
---

In this section we are interested in finding some non-trivial groups whose average order of 
elements is an integer.

The GAP distribution includes a number of data libraries, an overview of which
can be found [here](http://www.gap-system.org/Datalib/datalib.html).
One of them is the [Small Groups Library](http://www.gap-system.org/Packages/sgl.html) by
Hans Ulrich Besche, Bettina Eick and Eamonn O'Brien which we want to use in this chapter. First,
we have a look at how to work with the Small Groups Library before using it to methodically search for groups with the
afore mentioned property.
This library contains all groups of a certain 'small' order, i.e. order less than a certain bound
and orders whose prime factorisation is small in some sense.

Let us have a look at some key functions. To call a group of order `n` out of the Small Groups Library,
you also need to know its identifying number. For example

~~~
gap> H := SmallGroup(64,1)
~~~
{: .source}

~~~
<pc group of size 64 with 6 generators>
~~~
{: .output}
calls the first group of order 64 out of the Small Group Library.

~~~
gap> NrSmallGroups(64)
~~~
{: .source}

~~~
267
~~~
{: .output}

shows that there are 267 groups of order 64. Here are a few other key functions of the package:
~~~
gap> SmallGroupsInformation(64);

  There are 267 groups of order 64.
  They are sorted by their ranks.
     1 is cyclic.
     2 - 54 have rank 2.
     55 - 191 have rank 3.
     192 - 259 have rank 4.
     260 - 266 have rank 5.
     267 is elementary abelian.

  For the selection functions the values of the following attributes
  are precomputed and stored:
     IsAbelian, PClassPGroup, RankPGroup, FrattinifactorSize and
     FrattinifactorId.

  This size belongs to layer 2 of the SmallGroups library.
  IdSmallGroup is available for this size.

gap> G:=SmallGroup(64,2);
<pc group of size 64 with 6 generators>
gap> AllSmallGroups(Size,64,NilpotencyClassOfGroup,5);
[ <pc group of size 64 with 6 generators>, <pc group of size 64 with 6 generators>,
  <pc group of size 64 with 6 generators> ]
gap> List(last,IdGroup);
[ [ 64, 52 ], [ 64, 53 ], [ 64, 54 ] ]
gap> AllSmallGroups(64, IsAbelian);;
gap> List(last, IdGroup);
[ [ 64, 1 ], [ 64, 2 ], [ 64, 26 ], [ 64, 50 ], [ 64, 55 ], [ 64, 83 ], [ 64, 183 ], [ 64, 192 ], [ 64, 246 ], [ 64, 260 ], [ 64, 267 ] ]
gap> Size(last);
11
gap> AllSmallGroups(64, IsSolubleGroup);;
gap> Size(last);
267
~~~
{: .output}

`SmallGroupsInformation` gives us basic information about the small groups of a given size as well as precomputed
information about these groups.
The function `AllSmallGroups` returns all small groups with a given size provided as the argument. However, it is possible
to use a filter to search for small groups of a given size with additional properies such as abelian groups, 
groups with given nilpotency class, cyclic groups and soluble groups.
One can also use this library to identify a given group with a small group. For example

~~~
gap> H := Group((1,2,3),(3,6,9),(2,6));;
gap> IdSmallGroup(H);
~~~
{: .source}

~~~
[ 120, 34 ]
~~~
{: .output}


~~~
gap> SmallGroup(120,34);
~~~
{: .source}

~~~
Group([ (1,2,3,4,5), (1,2) ]);
~~~
{: .output}

Now we are prepared to use the Small Group Library to search for more groups whose average order of elements is an integer.

We are going to use inline notatation to define a test function for the above property of a given group.
Defining a function in this way is possible for one-argument functions.

~~~
TestOneGroup := G -> IsInt( AvgOrdOfGroup(G) );
~~~
{: .source}

Now let us try

~~~
List([TrivialGroup(),Group((1,2))],TestOneGroup);
~~~
{: .source}

~~~
[ true, false ]
~~~
{: .output}

~~~
gap> AllSmallGroups(Size,24,TestOneGroup,true);
~~~
{: .source}

~~~
[ ]
~~~
{: .output}

> ## Modular programming begins here
>
> Why is returning booleans a good design decision for such functions,
> instead of just printing information or returning a string such as `"YES"` ?
{: .callout}

Let us first design a rudimentary function testing all groups of a given order and returning
a group whose average order of elements is an integer as soon as one is found and `fail`, which is a special boolen variable
in GAP, otherwise. 

~~~
TestOneOrderEasy := function(n)
local i;
for i in [1..NrSmallGroups(n)] do
  if TestOneGroup( SmallGroup( n, i ) ) then
    return [n,i];
  fi;
od;
return fail;
end;
~~~
{: .source}

For example,

~~~
TestOneOrderEasy(1);
~~~
{: .source}

~~~
[ 1, 1 ]
~~~
{: .output}

~~~
TestOneOrderEasy(24);
~~~
{: .source}

~~~
fail
~~~
{: .output}

> ## `AllSmallGroups` runs out of memory - what to do?
>
> * Use iteration over `[1..NrSmallGroups(n)]` as shown in the function above
> * Use `IdsOfAllSmallGroups` which accepts the same arguments as `AllSmallGroups`
> but returns ids instead of groups.
{: .callout}

Iterating over `[1..NrSmallGroups(n)]` gives you more control over the progress of the calculation.
For example, the next version of our testing function prints additional information about the number of the
group being tested. It also supplies the testing function as an argument which allows us to plug
whatever testing function we want into `TestOneOrder`. 

~~~
TestOneOrder := function(f,n)
local i, G;
for i in [1..NrSmallGroups(n)] do
  Print(n, ":", i, "/", NrSmallGroups(n), "\r");
  G := SmallGroup( n, i );
  if f(G) then
    Print("\n");
    return [n,i];
  fi;
od;
Print("\n");
return fail;
end;
~~~
{: .source}

For instance,

~~~
TestOneOrder(TestOneGroup,64);
~~~
{: .source}

will display a changing counter during calculation and then return `fail`:

~~~
64:267/267
fail
~~~
{: .output}

We suspect that groups whose average order of elements is an integer are rather rare. Hence it is practical to 
write a function checking the groups of order 2 up to `n` for the desired property. Additionally
we want the function to return a group whose average order of elements is an integer as soon as one is found.

~~~
TestAllOrders:=function(f,n)
local i, res;
for i in [2..n] do
  res:=TestOneOrder(f,i);
  if res <> fail then
    return res;
  fi;
od;
return fail;
end;
~~~
{: .source}

Let us check all groups of order up to 128 for this property:

~~~
TestAllOrders(TestOneGroup,128);
~~~
{: .source}

~~~
2:1/1
3:1/1
4:2/2
5:1/1
6:2/2
7:1/1
8:5/5
...
...
...
100:16/16
101:1/1
102:4/4
103:1/1
104:14/14
105:1/2
[ 105, 1 ]
~~~
{: .output}

Our function tells us that the average order of elements of`SmallGroup(105,1)` is an integer.

Let us have a closer look at the group we just found. For example, its isomorphism type is of interest. 
It can be computed using the GAP function `StructureDescription`. Check [here](http://www.gap-system.org/Manuals/doc/ref/chap39.html#X87BF1B887C91CA2E) for further information.

~~~
G:=SmallGroup(105,1); AvgOrdOfGroup(G); StructureDescription(G);
~~~
{: .source}

~~~
<pc group of size 105 with 3 generators>
17
"C5 x (C7 : C3)"
~~~
{: .output}

Let us continue our hunt for more such groups. As there are only 2 groups of order 105:

~~~
gap> NrSmallGroups(105);
2
~~~
{: .source}
we check the 2nd group of order 105 manually and continue checking the groups of order 106 up to 256 methodically.

~~~
gap> TestOneGroup(SmallGroup(105,2));
false
~~~
{: .source}

We already checked the groups of order up to and including 105 for the desired property. Let us modify our function
slightly to be able to set a different starting point than order 2.

~~~
TestRangeOfOrders:=function(f,n1,n2)
local n, res;
for n in [n1..n2] do
  res:=TestOneOrder(f,n);
  if res <> fail then
    return res;
  fi;
od;
return fail;
end;
~~~
{: .source}

Let us check the groups of order 106 up to 256:
~~~
TestRangeOfOrders(TestOneGroup,106,256);
~~~
{: .source}

We notice that checking 2328 groups of order 128 and 56092 groups of order 256 is not feasible
and stop the computation.

This is again a situation where theoretical knowledge helps much more than
a brute-force approach. If the group is a _p_-group then the order of each
conjugacy class of a non-identity element of the group is divisible by _p_;
therefore, the average order of a group element can not be an integer. Therefore
_p_-groups can be excluded from our calculation. Let us change our code accordingly using the function
`IsPrimePowerInt`:

~~~
TestRangeOfOrders:=function(f,n1,n2)
local n, res;
for n in [n1..n2] do
  if not IsPrimePowerInt(n) then
     res:=TestOneOrder(f,n);
     if res <> fail then
       return res;
     fi;
   fi;
od;
return fail;
end;
~~~
{: .source}

We can now check all groups of order 106 up to 512, excluding _p_-groups:

~~~
gap> TestRangeOfOrders(TestOneGroup,106,512);
~~~
{: .source}

~~~
106:2/2
108:45/45
...
350:10/10
351:14/14
352:195/195
354:4/4
355:2/2
356:5/5
357:1/2
[ 357, 1 ]
~~~
{: .output}

So we found a group of order 357 satisfying our designated property. Let us have a look at that group:

~~~
AvgOrdOfGroup( SmallGroup(357,1)); StructureDescription( SmallGroup(357,1));
65
"C17 x (C7 : C3)"
~~~
{: .source}

Our next goal is to make our function as flexible and as user friendly as possible.
To begin with, we modify `TestOneOrder`.

Firstly, we are going to make `TestOneOrder` variadic, i.e. it may accept two or more
arguments. We will set 2 as minimum number of arguments, the first two being `f` and `n`,
as we need to know a testing function `f` as well as the order `n` we want to check.
We additionally allow for optional arguments that will be stored in the list `r`, indicated
by `r..` in the code below. The entries of `r` represent the indentifying number of 
the first and last small group of order `n` we want to check. By default these numbers are equal
to `1` and `NrSmallGroups(n)`. Because of these modifications we have to validate our input and return
user-friendly error messages in case of a wrong input.

Secondly we are going to introduce the concept of `Info` messages and `Info` levels. `Info` messages
may be switched on and off depending on the `Info` level set by the user. The need
we address here is the ability to switch the levels of verbosity of the
output without the error-prone approach of walking through the code and commenting
`Print` statements in and out. Let us have a quick look at `Info` classes before changing the code of 
our `TestOneGroup` function.

We have to create an `InfoClass` first:

~~~
gap> InfoSmallGroupsSearch := NewInfoClass("InfoSmallGroupsSearch");
~~~
{: .source}

~~~
InfoSmallGroupsSearch
~~~
{: .output}

Now instead of `Print("something");` one can use
`Info( InfoSmallGroupsSearch, infolevel, "something" );`
where `infolevel` is a positive integer specifying the level of verbosity.
This level could be changed to `n` using the command
`SetInfoLevel( InfoSmallGroupsSearch, n);`. See actual calls of `Info` in
the code below:

~~~
TestOneOrderVariadic := function(f,n,r...)
local n1, n2, i;

if not Length(r) in [0..2] then
  Error("The number of arguments must be 2,3 or 4\n" );
fi;

if not IsFunction( f ) then
  Error("The first argument must be a function\n" );
fi;

if not IsPosInt( n ) then
  Error("The second argument must be a positive integer\n" );
fi;

if IsBound(r[1]) then
  n1:=r[1];
  if not n1 in [1..NrSmallGroups(n)] then
    Error("The 3rd argument, if present, must belong to ", [1..NrSmallGroups(n)], "\n" );
  fi;
else
  n1:=1;
fi;

if IsBound(r[2]) then
  n2:=r[2];
  if not n2 in [1..NrSmallGroups(n)] then
    Error("The 4th argument, if present, must belong to ", [1..NrSmallGroups(n)], "\n" );
  elif n2 < n1 then
    Error("The 4th argument, if present, must be greater or equal to the 3rd \n" );
  fi;
else
  n2:=NrSmallGroups(n);
fi;

Info( InfoSmallGroupsSearch, 1,
      "Checking groups ", n1, " ... ", n2, " of order ", n );
for i in [n1..n2] do
  if InfoLevel( InfoSmallGroupsSearch ) > 1 then
    Print(i, "/", NrSmallGroups(n), "\r");
  fi;
  if f(SmallGroup(n,i)) then
    Info( InfoSmallGroupsSearch, 1,
          "Discovered counterexample: SmallGroup( ", n, ", ", i, " )" );
    return [n,i];
  fi;
od;
Info( InfoSmallGroupsSearch, 1,
      "Search completed - no counterexample discovered" );
return fail;
end;
~~~
{: .source}

The following example demonstrates how the output may be controlled
by switching the info level for `InfoSmallGroupsSearch`:

~~~
gap> TestOneOrderVariadic(IsIntegerAverageOrder,24);
fail
gap> SetInfoLevel( InfoSmallGroupsSearch, 1 );
gap> TestOneOrderVariadic(IsIntegerAverageOrder,24);
#I  Checking groups 1 ... 15 of order 24
#I  Search completed - no counterexample discovered
fail
gap> TestOneOrderVariadic(IsIntegerAverageOrder,357);
#I  Checking groups 1 ... 2 of order 357
#I  Discovered counterexample: SmallGroup( 357, 1 )
[ 357, 1 ]
gap> SetInfoLevel( InfoSmallGroupsSearch, 0);
gap> TestOneOrderVariadic(IsIntegerAverageOrder,357);
[ 357, 1 ]
~~~
{: .output}

This causes some problems for our test file since the `Test` function
compares the actual output to the reference output. To resolve
this problem we run the tests at info level 0 to suppress all additional output. 

~~~
# Finding groups with integer average order
gap> SetInfoLevel( InfoSmallGroupsSearch, 0);
gap> res:=[];;
gap> for n in [1..360] do
>      if not IsPrimePowerInt(n) then
>        t := TestOneOrderVariadic( IsIntegerAverageOrder,n,1,NrSmallGroups(n) );
>        if t <> fail then
>          Add(res,t);
>        fi;
>      fi;
>    od;
gap> res;
[ [ 1, 1 ], [ 105, 1 ], [ 357, 1 ] ]
~~~
{: .output}

> ## Does the Small Groups Library contain another group with this property?
> 
> * Create proper exercises.
>
> * What can you say about the order of the groups with this property?
>
> * Can you estimate how long it may take to check all 408641062 groups of order 1536 ?
>
> * How many groups of order not higher than 2000 may you be able to check,
>   excluding _p_-groups and those of order 1536?
>
> * Can you find in the Small Groups Library another group (of order not equal
>   to 1536) with this property?
{: .challenge}
