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
- "Understand how to adjust tests to check randomised algorithms"
- "Learn the 'Make it right, then make it fast' concept"
keypoints:
- "It is easy to create a test file by copying and pasting GAP session."
- "Writing a good and comprehensive test suite requires some efforts."
- "Make it right, then make it fast!"
---

The code of `AvgOrdOfGroup` is very simple and possibly nothing could go wrong
with it. It avoids problems with running out of memory because of iterating
over the group instead of creating a list of its elements
(calling `AsList(SymmetricGroup(11))` already results in exceeding the permitted
memory), however it takes several minutes to calculate an average order of an
element of `SymmetricGroup(11)`. But at least we are confident that it is
correct.

Now we would like to write a better version of this function using some
theoretical facts we know from the Group Theory. We may put
`avgorg.g` under version control to revert changes if need be;
we may create a new function to keep the old one around and compare the
results of both; but this could be made even more efficient if we will
use **regression testing**: this is the term for the testing based on
rerunning previously completed tests to check that new changes do not
impact their correctness or worsen their performance.

To start with, we need to create a **test file**. The test file looks
exactly like the GAP session, so it is easy to create it by copying and
pasting the GAP session with all GAP prompts, inputs and outputs into a
text file (a test file could also be created from the log file with a
GAP session recorded with the help of `LogTo`). During the test, GAP will
run all inputs from the test, compare the outputs with those in the test
file and report any differences.

GAP test files are just text files but the common practice is to name
them with the extension `.tst`. Now create the file `avgord.tst` (for simplicity in the current directory) with the following content:

~~~
# tests for average order of a group element

# permutation group
gap> S:=SymmetricGroup(9);
Sym( [ 1 .. 9 ] )
gap> AvgOrdOfGroup(S);
3291487/362880
~~~
{: .source}

As you see, the test file may include comments, with certain rules specifying
where they may be placed, because one should be able to distinguish comments
in the test file from GAP output started with `#`. For that purpose
lines at the beginning of the test file that start with `#` and one empty line
together with one or more lines starting with `#` are considered as comments.
All other lines are considered GAP output from the preceding GAP input. Please be careful:
By default, the `Test` function considers blank spaces and additional empty lines as a difference in the output. 
TODO: Bsp mit Whitespaces und FAIL


To run the test, one should use the function `Test`, as documented [here](http://www.gap-system.org/Manuals/doc/ref/chap7.html#X87712F9D8732193C).
For example (assuming that the function `AvgOrdOfGroup` is already loaded):

~~~
Test("avgord.tst");
~~~
{: .source}

~~~
true
~~~
{: .output}

In this case `Test` reported no discrepancies and returned `true`, so we
conclude that the test has passed.

We will neither cover the topic of writting a good and comprehensive test suite
nor discuss options provided by the `Test` function. For instance, it is possible
to ignore differences in the output formatting as well as displaying the progress of the test.
See the `Test` documentary to learn more about the features of the this function.

Instead, we will now add more groups to `avgord.tst` to demonstrate that the
code also works with other kinds of groups and to show various ways of
combining commands in the test file:

~~~
# tests for average order of a group element

# permutation group
gap> S:=SymmetricGroup(9);
Sym( [ 1 .. 9 ] )
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

# finite matrix group over integers

# matrix group over a finite field
gap> AvgOrdOfGroup(SL(2,5));
221/40
~~~
{: .source}

Let us test the extended version of the test again and check that it works:

~~~
Test("avgord.tst");
~~~
{: .source}

~~~
true
~~~
{: .output}

We have not used the underlying group sructure in our `AvgOrdOfGroup` function. It works just as fine 
on a set of permutation not forming a group as the following example shows:

~~~
gap> coll := [(1,2,3),(2,3,4),(8,9,3),(1,2)];
[ (1,2,3), (2,3,4), (3,8,9), (1,2) ]
gap> AvgOrdOfGroup(coll);                
11/4
gap> IsInt(last);
false

~~~

We now want to work on a better implementation of `AvgOrdOfGroup` using the group structure of 
the argument. For this purpose, we rename our original function to `AvgOrdOfCollection` and define 
a new `AvgOrdOfGroup` function. Using the fact that the order of an element is invariant
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

but when we run the test, here comes a surprise!

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

Thus, the approach "Make it right, then make it fast" helped to detect a bug
immediately after it has been introduced.
