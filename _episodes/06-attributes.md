---
title: "Attributes and Methods"
teaching: 40
exercises: 10
questions:
- "How to record information in GAP objects"
objectives:
- "Declaring an attribute"
- "Checking an object for attributes"
- "Installing a method"
- "Understanding method selection"
keypoints:
- "_Positional_ objects may accumulate information about themselves during their lifetime. This is done by using _Attributes_."
- "This means that next time the stored information may be retrieved at zero costs."
- "_Methods_ are bunches of functions; the _method selection_ will choose the most efficient method based on the type of all arguments."
---

Of course, for any given group the average order of its elements needs to
be calculated only once, as the next time it will return the same value.
However, as we see from the runtimes below, each new call of `AvgOrdOfGroup`
will repeat the same computation again, with slightly varying runtime:

~~~
A:=AlternatingGroup(10);
~~~
{: .source}

~~~
Alt( [ 1 .. 10 ] )
~~~
{: .output}

~~~
AvgOrdOfCollection(A); time; AvgOrdOfCollection(A); time;
~~~
{: .source}

~~~
2587393/259200
8226
2587393/259200
8118
~~~
{: .output}

As you can see, we only called `AlternatingGroup(10)` once but GAP apparently
did not store the result in `A` and did all the work again when we called 
`AvgOrdOfCollection(A)` for the second time.

If you need to reuse this value, one option is to store it in some
variable but then you must be careful about matching such variables
with corresponding groups and the code could become quite convoluted
and unreadable.

Luckily GAP provides a good solution to the above problem: Attributes of objects. 
These are used to accumulate information that objects learn about themselves during their lifetime.
Let us have a look at a first example:

~~~
G:=Group([ (1,2,3,4,5,6,7,8,9,10,11), (3,7,11,8)(4,10,5,6) ]);
gap> NrConjugacyClasses(G);time;NrConjugacyClasses(G);time;
~~~
{: .source}

~~~
Group([ (1,2,3,4,5,6,7,8,9,10,11), (3,7,11,8)(4,10,5,6) ])
10
39
10
0
~~~
{: .output}

In this case, the group `G` has 10 conjugacy classes, and it took 39 ms to
establish that in the first call. The second call has zero cost since the
result was stored in `G`, since `NrConjugacyClasses` is an attribute:

~~~
NrConjugacyClasses;
~~~
{: .source}

~~~
<Attribute "NrConjugacyClasses">
~~~
{: .output}

To see which attributes are known of a given object, GAP provides the function
`KnownAttributesOfObject`. Let us have a look at the following example.

~~~
gap> G:=Group([ (1,2,3,4,5,6,7,8,9,10,11), (3,7,11,8)(4,10,5,6) ]);
Group([ (1,2,3,4,5,6,7,8,9,10,11), (3,7,11,8)(4,10,5,6) ])
gap> KnownAttributesOfObject(G);
[ "LargestMovedPoint", "GeneratorsOfMagmaWithInverses", "MultiplicativeNeutralElement" ]
~~~
{: .source}
Let us now compute the number of conjugacy classes of `G` again and have a look at the known attributes
of `G`:

~~~
gap> NrConjugacyClasses(G);
10
gap> KnownAttributesOfObject(G);
[ "Size", "PseudoRandomSeed", "OneImmutable", "SmallestMovedPoint", "LargestMovedPoint", "NrMovedPoints", "MovedPoints", 
  "GeneratorsOfMagmaWithInverses", "TrivialSubmagmaWithOne", "MultiplicativeNeutralElement", "ConjugacyClasses", "PerfectResiduum", 
  "DerivedSeriesOfGroup", "DerivedSubgroup", "NrConjugacyClasses", "IsomorphismTypeInfoFiniteSimpleGroup", "BlocksAttr", 
  "StabChainMutable", "StabChainOptions", "DataAboutSimpleGroup" ]
~~~
{: .source}

Just after creating the group `G`, GAP only knows a few basic attributes of `G`, not even its size.
Computing the number of conjugacy classes involves computing a few informations about `G`. For example the 
conjugacy classes of `G` as well as its derived subgroup are known:

~~~
gap> ConjugacyClasses(G);
[ ()^G, (1,3)(2,11)(4,6)(7,8)^G, (1,11,3,2)(4,8,6,7)^G, (1,7,11,4,3,8,2,6)(5,10)^G, (1,6,2,8,3,4,11,7)(5,10)^G, 
  (1,8,5,2,7)(3,11,10,9,4)^G, (1,2,7)(3,8,11)(4,6,9)^G, (1,9,2,4,7,6)(3,11,8)(5,10)^G, (1,6,5,10,4,11,2,3,7,8,9)^G, 
  (1,9,8,7,3,2,11,4,10,5,6)^G ]
gap> time;
1
gap> DerivedSubgroup(G);
Group([ (1,9,4,7,3)(5,10,8,6,11), (1,9,10,11,7)(3,4,8,6,5), (1,3,2,5,8)(4,10,6,9,11) ])
gap> time;
0
~~~
{: .source}

This shows us that GAP stores a lot of results produced during a computation in `G`. However, as
seen earlier, the result of `AvgOrderOfCollection` is not stored in the group we apply the function to.
Thus, we now want to focuss on how to create our own attributes and how to use them.

Since we already have a function `AvgOrdOfCollection` which
does the calculation, the simplest example of turning it into
an attribute looks like this:

~~~
AverageOrder := NewAttribute("AverageOrder", IsCollection);
InstallMethod( AverageOrder, "for a collection", [IsCollection], AvgOrdOfCollection);
~~~
{: .source}

In this example we first declared an attribute `AverageOrder` for
objects in the category `IsCollection`, and then installed the function
`AvgOrdOfCollection` as a method for this attribute. Instead of calling
the function `AvgOrdOfCollection` we may now call `AverageOrder`.

Let us see if defining this new attribute and installing a method for this attribute
has the desired effect: Calling `AvgOrder` for a second time on the same collection
comes at zero cost and the result is stored in `S`:

~~~
S:=SymmetricGroup(10);; KnownAttributesOfObject(S); AverageOrder(S); time; KnownAttributesOfObject(S); AverageOrder(S); time;
~~~
{: .source}

~~~
[ "Size", "NrMovedPoints", "MovedPoints", "GeneratorsOfMagmaWithInverses", "MultiplicativeNeutralElement" ]
39020911/3628800
16445
[ "Size", "NrMovedPoints", "MovedPoints", "GeneratorsOfMagmaWithInverses", "MultiplicativeNeutralElement", "StabChainMutable", 
  "StabChainImmutable", "AverageOrder" ]
39020911/3628800
0
~~~
{: .output}

So indeed, the cost for the second call of our function is zero and GAP stores our newly defined 
attribute in `S`. We told GAP that `AverageOrder` is applicable to collections in general, so the input need not be a group:

~~~
gap> A := [ (1,2,3),(2,3,4,5,6),(9,10)];
[ (1,2,3), (2,3,4,5,6), (9,10) ]
gap> IsGroup(A); IsCollection(A);
false
true
gap> AverageOrder(A);
10/3
~~~
{: .source}

So far we have only implemented our `AvgOrdOfCollection` function and not its improved version for groups.
As groups are collections as well, GAP applies the method `AvgOrder` to groups, using
`AvgOrdOfCollection`, as seen above.

We would like to install `AvgOrdOfGroup` as a method for groups as well:

~~~
InstallMethod( AverageOrder, "for a group", [IsGroup], AvgOrdOfGroup);
~~~
{: .source}

For a newly created group, GAP will use `AvgOrdOfGroup` when `AverageOrder` is called:
~~~
S:=SymmetricGroup(10);; AverageOrder(S); time; AverageOrder(S); time;
~~~
{: .source}

~~~
39020911/3628800
26
39020911/3628800
0
~~~
{: .output}

As there usually are several methods applicable to the same object to compute
the same information, for instance `AvgOrdOfCollection` is applicable two groups
and collections in general, one could ask GAP for the most suitable method to performe
the desired task. GAP provides the function `ApplicableMethod` to answer that question. 
Here is a first example:

~~~
gap> ApplicableMethod(AverageOrder, [G],1);
~~~
{: .source}

~~~
#I  Searching Method for AverageOrder with 1 arguments:
#I  Total: 3 entries
#I  Method 2: ``AverageOrder: for a group'', value: 35
function( G ) ... end
~~~
{: .output}

The integer we provide as the third argument is the print level. Print level 1 means that
GAP only shows us the highest ranked method to compute the attribute `AverageOrder`. Calling
`ApplicableMethod` with the highest print level, i.e. level `6`, yields

~~~
gap> ApplicableMethod(AverageOrder, [G],6); 
~~~
{: .source}

~~~
#I  Searching Method for AverageOrder with 1 arguments:
#I  Total: 3 entries
#I  Method 1: ``AverageOrder: system getter'', value: 2*SUM_FLAGS+5
#I   - 1st argument needs [ "Tester(AverageOrder)" ]
#I  Method 2: ``AverageOrder: for a group'', value: 35
#I  Function Body:
function ( G )
    local  cc, sum, c;
    cc := ConjugacyClasses( G );
    sum := 0;
    for c  in cc  do
        sum := sum + Order( Representative( c ) ) * Size( c );
    od;
    return sum / Size( G );
endfunction( G ) ... end
~~~
{: .output}
See the documentation of `ApplicableMethod` [here](https://www.gap-system.org/Manuals/doc/ref/chap7.html#X7FDA1D4B87BD25A8).


> ## Which method is being called
>
> * `ApplicableMethod` in combination with `PageSource` may point you to
>   the source code with all the comments.
{: .callout}

So far we have declared `AverageOrder` as an attribute and associated it with every group
we compute it for. We would also like to create an attribute that tells us directly if 
a given group has a whole numbered average order of elements.

In GAP a _property_ is a boolean-valued attribute. It can be created using `NewProperty`
as shown in the following example:

~~~
IsIntegerAverageOrder := NewProperty("IsIntegerAverageOrder", IsCollection);
~~~
{: .source}

Now we will install a method for `IsIntegerAverageOrder` for a collection.
Observe that neither below nor in the examples above it is necessary to create
a function first and then install it as a method. The following command installes a method
to compute the above defined property: 

~~~
InstallMethod( IsIntegerAverageOrder,
  "for a collection",
  [IsCollection],
  coll -> IsInt( AverageOrder( coll ) )
);
~~~
{: .source}

Note that because `AverageOrder` is an operation it will take care of selecting
the most suitable method.

As a wrap up of this session, let us have a last look at another example, a so 
called _pc group_. These are groups admitting a subnormal series with cyclic factors. The
first 1000 groups of order 1536 are pc groups.

~~~
l:=List([1..1000],i->SmallGroup(1536,i));; List(l,AvgOrdOfGroup);;time;
~~~
{: .source}

~~~
56231
~~~
{: .output}

~~~
l:=List([1..1000],i->SmallGroup(1536,i));; List(l,AvgOrdOfCollection);;time;
~~~
{: .source}

~~~
9141
~~~
{: .output}

This is surprising: Apparantly iterating over the elements of a pc group is faster
than computing its conjugacy classes and summing up the orders of elements of the
group that way.

> ## Exercises
> 
> * Install a method for `IsPcGroup` that iterates over the group elements
>   instead of calculations its conjugacy classes.
>
> * Can you find an example of a pc group for which iterating is slower than calculating conjugacy classes?
>
> > ## Solution
> > * TBA1
> >
> > * TBA2 
> {: .solution}
{: .challenge}
