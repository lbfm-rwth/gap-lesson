---
title: "Using regression tests"
teaching: 40
exercises: 10
questions:
- "Test-driven development"
objectives:
- "Be able to create and run test files"
- "Understand how test discrepancies and runtime regressions
 could be identified and interpreted"
- "Learn the 'Make it right, then make it fast' concept"
keypoints:
- "It is easy to create a test file by copying and pasting GAP session or use the LogTo-command."
- "Writing a good test suite is an art in itself."
- "Make it right, then make it fast!"
---

The code of `AvgOrdOfGroup` is very simple and we expect it to be correct. It avoids problems with
running out of memory as it iterates over the group instead of creating a list of its elements
(calling `AsList(SymmetricGroup(11))` already results in exceeding the permitted
memory). However, it takes several minutes to calculate the average order of an
element of `SymmetricGroup(11)` and thus needs improvement. This chapter deals with improving `AvgOrdOfGroup`
using some facts from Group Theory and particularly with designing automated tests to check the correctness
of these changes.

At this point we should think about putting `avgorg.g` under version control to be able to revert changes if needed instead
of creating a new function to keep the old one around. Additionally, we should start testing our function
methodically as we move along. This can be done efficiently by using **regression testing**, i.e. testing based on rerunning previously completed tests to check that new changes do not impact their correctness or worsen their performance.

First we need to create a **test file**. The test file looks
exactly like a GAP session. Thus, it is easy to create a test file simply by copying and
pasting the GAP session with all GAP prompts, inputs and outputs into a
text file.  Let us have a closer look at testing files.

GAP test files are just text files but it is common practice to name them with the extension `.tst`. Now create the file `avgord.tst` (for simplicity in the current directory) with the following content:

~~~
# tests for average order of a group element

# permutation group
gap> n := 9;;
gap> S := SymmetricGroup(n);;
gap> A := AlternatingGroup(n);;
gap> D := DihedralGroup(2*n);;
gap> AvgOrdOfGroup(S);
3291487/362880
gap> AvgOrdOfGroup(A);
1516831/181440
gap> AvgOrdOfGroup(D);
79/18
~~~
{: .source}

> ## Floats vs. Ints in AvgOrdOfGroup
> * What would the return value of `AvgOrdOfGroup` be if the variable `sum` would be a
float (e.g. `sum := 0.0`) instead of an integer?
> * What would this mean for the tests?
>
> > ## Solution
> > * The return value would be a float.
> > * Floating point errors could force the test to fail even though the code is correct. 
> {: .solution} 
{: .challenge}

As you see, the test file may include comments with certain rules specifying
where they may be placed, because one should be able to distinguish comments
in the test file from GAP output started with `#`. For that purpose
lines at the beginning of the test file starting with `#` and one empty line
together with one or more lines starting with `#` are considered to be comments.
All other lines are considered GAP output from the preceding GAP input. Please be careful:
By default, the `Test` function considers blank spaces and additional empty lines as a difference in the output. 


To run the test one should use the function `Test`, as documented [here](http://www.gap-system.org/Manuals/doc/ref/chap7.html#X87712F9D8732193C).
For example (assuming that the function `AvgOrdOfGroup` is already loaded):

~~~
Test("avgord.tst");
~~~
{: .source}

~~~
true
~~~
{: .output}

In this case `Test` reported no discrepancies and returned `true`, indicating that `AvgOrdOfGroup` has passed the test.

> ## Creating your own test file.
    
>  Create your own test file and run a test on `AvgOrdOfGroup` for the following groups: `SymmetricGroup(7)`, `DihedralGroup(14)` and `AlternatingGroup(7)`.
> > ## Solution
> > ~~~ 
> >  S := SymmetricGroup(7);; D := DihedralGroup(14);; A := AlternatingGroup(7);;               
> >  gap> AvgOrdOfGroup(S);
> >  31333/5040
> >  gap> AvgOrdOfGroup(D);
> >  57/14
> >  gap> AvgOrdOfGroup(A);
> >  12601/2520
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

Writing good test functions is an art in itself. We only cover the
very basics in this course.

We will now add more groups to `avgord.tst` to demonstrate that the
code also works with other kinds of groups and to show various ways of
combining commands in the test file:

~~~
# tests for average order of a group element

# permutation group
gap> n := 9;;
gap> S := SymmetricGroup(n);;
gap> A := AlternatingGroup(n);;
gap> D := DihedralGroup(2*n);;
gap> AvgOrdOfGroup(S);
3291487/362880
gap> AvgOrdOfGroup(A);
1516831/181440
gap> AvgOrdOfGroup(D);
79/18

# permutation group
gap> S:=SymmetricGroup(9);
Sym( [ 1 .. 9  ]  )
gap> AvgOrdOfGroup(S);
3291487/362880

# pc group
gap> D:=DihedralGroup(512);
<pc group of size 512 with 9 generators>
gap> AvgOrdOfGroup(D);
44203/512
gap> G:=TrivialGroup();; # suppress output
gap> AvgOrdOfGroup(G);
1

# matrix group over a finite field
gap> AvgOrdOfGroup(SL(2,5));
221/40
~~~
{: .source}

Let us check the extended version of the test again and see if it works:

~~~
Test("avgord.tst");
~~~
{: .source}

~~~
true
~~~
{: .output}

We have not used the underlying group structure in our `AvgOrdOfGroup` function yet. It works just as fine 
on a set of permutations not forming a group as the following example shows:

~~~
gap> coll := [(1,2,3),(2,3,4),(8,9,3),(1,2)];
[ (1,2,3), (2,3,4), (3,8,9), (1,2) ]
gap> AvgOrdOfGroup(coll);                
11/4
gap> IsInt(last);
false
~~~
{: .source}

We now want to work on a better implementation of `AvgOrdOfGroup` using the group structure of 
the argument thus improving the runtime of our algorithm. For this purpose, we rename our original function
to `AvgOrdOfCollection` and define a new `AvgOrdOfGroup` function. Using the fact that the order of an element is invariant
under conjugation we compute the conjugacy classes of elements and one representative of each class.
This yields the following improved version of `AvgOrdOfGroup` that should be added into `avgord.g`.

~~~
AvgOrdOfGroup := function(G)
local cc, sum, c;
cc:=ConjugacyClasses(G);
sum:=0;
for c in cc do
  sum := sum + Order( Representative(c) ) * Size(cc);
od;
return sum/Size(G);
end;
~~~
{: .source}

But when we run the test, here comes a surprise!

~~~
Read("avgord.g");
Test("avgord.tst");
~~~
{: .source}

~~~
########> Diff in avgord.tst, line 6:
# Input is:
AvgOrdOfGroup(S);
# Expected output:
3291487/362880
# But found:
11/672
########
########> Diff in avgord.tst, line 12:
# Input is:
AvgOrdOfGroup(D);
# Expected output:
44203/512
# But found:
2862481/512
########
########> Diff in avgord.tst, line 21:
# Input is:
AvgOrdOfGroup(SL(2,5));
# Expected output:
221/40
# But found:
69/20
########
false
~~~
{: .output}

Indeed, we deliberately made a typo and replaced `Size(c)` by `Size(cc)`. The correct version should look as follows:

~~~
AvgOrdOfGroup := function(G)
local cc, sum, c;
cc:=ConjugacyClasses(G);
sum:=0;
for c in cc do
  sum := sum + Order( Representative(c) ) * Size(c);
od;
return sum/Size(G);
end;
~~~
{: .source}

Fixing this error in `avgord.g` reading the file again and running the test one more time yields:

~~~
Read("avgord.g");
Test("avgord.tst");
~~~
{: .source}

~~~
true
~~~
{: .output}

Thus, the approach "Make it right, then make it fast" helps to detect bugs
immediately after they have been introduced.

Another way of creating a test file is by using the `LogTo` command to record a GAP session.
During the test GAP will run all inputs from the test, compare the outputs with those in the test
file and report any differences. Let us first have a look at the `LogTo()` function before concerning ourselves with testing the code.

If you want to record what you did in a GAP session to have a look at it later, you can enable logging with the `LogTo` function:

~~~
LogTo("gap-intro.log");
~~~
{: .source}

This will create a file `gap-intro.log` in the current directory which
contains all subsequent input and output appearing in your terminal.
To stop logging, you can call `LogTo` without arguments:
~~~
gap> LogTo();
~~~
{: .source}

or leave GAP. Note that `LogTo` blanks the file before starting, if it
already exists!

It can be useful to leave some comments in the log file in case you return to it in the future.
You can enter the following after the GAP prompt:

~~~
# GAP Software Carpentry Lesson
~~~
{: .source}

then after pressing the Return key, GAP will display a new prompt and the comment
will be written to the log file.

The log file records all interaction with GAP happening after the call of `LogTo` but nothing that happened
before the function call.

Now we know two ways of creating test files: Copying the relevant lines out of your shell into a document or 
saving the session with `LogTo`.

> ## Creating test-files with the `LogTo`-command
>
> Create your own test file using the `LogTo`-command and test `AvgOrdOfGroup` for the following groups: `SymmetricGroup(9)`, `AlternatingGroup(9)`, `DihedralGroup(18)` and `SL(2,5)`.
> > ## Solution
> > ~~~
> > gap> S := SymmetricGroup(9);; A := AlternatingGroup(9);; D := DihedralGroup(18);; M := SL(2,5);;
> > gap> AvgOrdOfGroup(S);
> > 3291487/362880
> > gap> AvgOrdOfGroup(A);
> > 1516831/181440
> > gap> AvgOrdOfGroup(D);
> > 79/18
> > gap> AvgOrdOfGroup(M);
> > 221/40
> > ~~~
> > {: .source}
> {: .solution} 
{: .challenge}

> ## Continuous integration
>
>
>
{: .callout}
