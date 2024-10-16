---
title: awk-tober tutorial and tips and tricks
sub_title: With applications on genomics files
author: Oct 18th 2024, Rob Bierman, Research Software Engineer
theme:
  name: light
  override:
    footer:
      style: template
      right: "{current_slide} / {total_slides}"
---

Goals for this presentation
---
# We'll start with an awk **amuse-bouche**
<!-- pause -->
# History of the `awk` language
<!-- pause -->
# Learn how `awk` works with a "quota" dataset
<!-- pause -->
# `awk` as the inspiration for perl, python, and more
<!-- pause -->
# The power of `awk` on .bed and .gtf files
<!-- pause -->
# Nice `awk` trick to concatenate files with identical headers
<!-- pause -->
## The power of `awk` in combination with other bash tools
<!-- pause -->
# Using `awk`'s associative arrays
<!-- pause -->
# When to probably not use `awk`

<!-- end_slide -->

Let's see some awk
---

Here's a `data/quotas.tsv` file included in this github repo
```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```
<!-- pause -->
If we want to print just the `User's` column (the 3rd), we can use `awk`!

```bash
awk '{ print $3 }' data/quotas.tsv
```

<!-- pause -->
Output:
```bash
User
tcomi
rb3242
limingli
...
```

<!-- pause -->
(In this case, we could have also used `cut -f3 data/quotas.tsv`)

<!-- end_slide -->

What is awk?
---
* `awk` is a command line text-processing programming language
<!-- new_line -->
<!-- pause -->
* `awk` is pre-installed on basically ALL unix-like OS's
    * linux, mac, windows subsystem for linux
    * `awk` is part of *GNU coreutils*
    * when you have `ls` you'll also have `awk`
<!-- new_line -->
<!-- pause -->
* `awk` is really great when working with tabular text
<!-- new_line -->
<!-- pause -->
* `awk` is great for quick-and-dirty scripts
<!-- new_line -->
<!-- pause -->
* `awk` is old, but not as old as `sed`
    * First appeared in 1977 (`sed` 1973)
    * Developed at AT&T Bell Laboratories by
        * Alfred *A*ho
        * Peter  *W*einberger
        * Brian  *K*ernighan
<!-- new_line -->
<!-- pause -->
* Brian Kernighan is a professor at Princeton!
    * https://www.cs.princeton.edu/~bwk/

<!-- end_slide -->

Understanding the structure of an `awk` program: Column re-order
---
Let's return to the `data/quotas.tsv` dataset

```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```

and this time we want to create a new file where the order of the columns
have `User` first. Does anyone have any suggestions? `cut -f3,1,2` actually
doesn't work.

<!-- pause -->
```bash
awk '{ print $3,$1,$2 }' data/quotas.tsv
```
<!-- pause -->

Output
```bash
User projects scratch
tcomi 136.800 0.000
rb3242 62.000 10.100
limingli 0.000 42.600
...
```
<!-- end_slide -->

Understanding the structure of an `awk` program: Column re-order
---
Let's return to the `data/quotas.tsv` dataset

```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```

and this time we want to create a new file where the order of the columns
have `User` first. Does anyone have any suggestions?

```bash
awk '{ print $3,$1,$2 }' data/quotas.tsv
```

Program structure
```bash
awk '{ COMMANDS }' INPUT
```

<!-- pause -->
The COMMANDS get implicitly run on every line of the file,
you don't have to write a for-loop! This helps keep awk very terse.

<!-- pause -->
But what if we don't want to run on every line in the file?

<!-- end_slide -->

Understanding the structure of an `awk` program: No header
---
Let's say we still want to re-order the columns, but we don't want
the header row anymore. We'll start with:

```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```

and generate:
```bash
tcomi	136.800	0.000	
rb3242	62.000	10.100	
limingli	0.000	42.600	
...
```
<!-- pause -->

we could use a combination of `tail -n+2` to skip the first line
and then pipe that to our previous `awk` command

```bash
tail -n+2 data/quotas.tsv | awk '{ print $3,$1,$2 }'
```
<!-- pause -->

This shows that `awk` can be used in a pipe, which is nice,
but `awk` can handle all of this itself!

<!-- end_slide -->

Understanding the structure of an `awk` program: No header
---
Let's say we still want to re-order the columns, but we don't want
the header row anymore. We'll start with:

```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```

and generate:
```bash
tcomi	136.800	0.000	
rb3242	62.000	10.100	
limingli	0.000	42.600	
...
```

We can tell `awk` to skip the first line with the built in `NR` variable:
```bash
awk 'NR > 1 { print $3,$1,$2 }' data/quotas.tsv
```
<!-- pause -->

`NR` is one of a handful of special variables and keeps track of the current line number!
<!-- pause -->

What if we wanted to calculate the sum quota of a user on scratch and projects?

<!-- end_slide -->

Understanding the structure of an `awk` program: Sums
---
We'll start with the same file:
```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```

but now we'll make:
```bash
tcomi 136.8
rb3242 72.1
limingli 42.6
...
```
<!-- pause -->

We can just add the 1st and 2nd columns!
```bash
awk 'NR > 1 { print $3,$1+$2 }' data/quotas.tsv
```
<!-- pause -->

But what if we wanted a custom header?

<!-- end_slide -->

Understanding the structure of an `awk` program: Custom sums
---
We'll start with the same file:
```bash
projects	scratch	User	
136.800	0.000	tcomi
62.000	10.100	rb3242
0.000	42.600	limingli
...
```

but now we'll make:
```bash
USER	SUM
tcomi 136.8
rb3242 72.1
limingli 42.6
...
```
<!-- pause -->

We can specify a special instruction for just the first row
```bash
awk 'NR == 1 { print "USER SUM"} NR > 1 { print $3,$1+$2 }' data/quotas.tsv
```
<!-- pause -->

We've already done a lot with `awk`, let's have a review.
<!-- end_slide -->

Understanding the structure of an `awk` program: Review
---
* An `awk` program operates on one line at a time and takes the structure:
```bash
awk 'CONDITION { COMMAND } CONDITION { COMMAND } ... ' input.txt
```
<!-- pause -->

* `awk` provides `$1`, `$2`, etc for working with columns
    * awk splits on whitespace by default
    * (Later example where we split on commas)

<!-- pause -->
<!-- new_line -->

* `awk` is happy to do numerical calculations for us!
    * It will try to perform implicit conversions

<!-- pause -->
<!-- new_line -->

* No need to import libraries, `awk` is always installed!

<!-- pause -->
<!-- new_line -->

Next we're going to see how `awk` inspired other famous tools!

<!-- end_slide -->

awk as an inspiration
---
* `perl` (1987) was inspired from `awk` (1977) as well as other languages
> Perl borrows features from other programming languages 
> including C, sh, AWK, and sed.[1]

> Perl takes hashes ("associative arrays") from AWK and 
> regular expressions from sed.
<!-- pause -->

* and since `python` (1991) was inspired by `perl`
> Many professional programmers are turning to Python, 
> often as an alternative to Perl, or other scripting languages. 

> Like Perl, Python is excellent for scripting, 
> and string manipulation, yet its syntax is much less cryptic. [2]


<!-- pause -->
People have made family trees of programming languages!

References
1. https://history.perl.org/PerlTimeline.html
2. https://www.cs.ubc.ca/wccce/Program03/papers/Toby.html

<!-- end_slide -->

Programming language family tree
---
```
      COBOL(1959)  Fortran(1957)                      Lisp(1958)
                      |                                  |
           ╭----------┴---╮                              |
      ALGOL(1960)   BASIC(1964)                ╭---------|
           |                                   |         |
           ╰---╮----------╮                    |         |
          C(1972)  Pascal(1970)     AWK(1977)  | Smalltalk(1972)
           |                           ╰-╮     |
           ╰--╮----------╮------╮------╮ | ╭---|
Objective-C(1986)     C++(1980) |  Perl(1987)  |
                         |      |        |     |
        ╭----------------┴--╮   ╰---╮    | ╭---╯
 JavaScript(1995)    Java(1995)  Python(1990)
```
(Stolen from https://tecky.io/en/blog/evolution-of-programming-languages/)

<!-- pause -->
Let's do some more `awk`! This time on common genomics files!

<!-- end_slide -->


Calculating total number of basepairs covered in a `.bed` file
---
Here's a `data/tiny.bed` file (with a header) included in this github repo
```bash
#chr start end
chr1 100 120
chr1 140 160
chr2 560 580
```
<!-- pause -->
`awk` script:

```bash
awk '{ s+=$3-$2 } END { print s }' data/tiny.bed
```
<!-- pause -->
result: `60`

<!-- pause -->

For pandas users, you could do the same thing with
```python
import pandas as pd
df = pd.read_csv("data/tiny.bed")
(df['end']-df['start']).sum()
```

<!-- pause -->

For R users, you could do the same thing with
```r
library(tidyverse)
dat = read_csv("data/tiny.bed")
sum(dat['end']-dat['start'])
```

<!-- end_slide -->

Understanding the running sum of .bed region spans
---
How is this script is working?
```bash
#chr start end
chr1 100 120
chr1 140 160
chr2 560 580
```

`awk` script:

```bash
awk '{ s+=$3-$2 } END { print s }' data/tiny.bed
```
<!-- pause -->
We can make and use variables in `awk`! It's a programming language!
What about the header line?
<!-- pause -->
`awk` couldn't convert `start` and `end` to numeric, so defaulted to 0!

<!-- pause -->
There's another file called `data/tiny_bad_header.bed` 
which will produce the same output!
```bash
chr1 100 120
chr1 140 160
#chr start end
chr2 560 580
```
<!-- end_slide -->

ASSOC ARRAYS Vernot 2016 TODO SLIDE
---
Need to add info here now
<!-- end_slide -->

GTF TODO SLIDE (SHOW SPLIT LINES)
---
Need to add info here now
<!-- end_slide -->

CONCAT FILES IDENTICAL HEADERS TODO SLIDE
---
Need to add info here now
<!-- end_slide -->

When not to use `awk`
---
* `awk` is best used for relatively simple scripts
<!-- new_line -->
* If your awk script starts getting to be large and ugly 
then maybe it's time to switch to R or python
<!-- new_line -->
* `awk` is great as "glue" between command line tools or scripts
and can feel a bit "unprofessional"
<!-- pause -->
Having said that, even Heng Li suggests using `awk` to processing `hifiasm` output!

Here's an entry in the `hifiasm` FAQ

> How do I get contigs in FASTA?

The FASTA file can be produced from GFA as follows:
```bash
awk '/^S/{print ">"$2;print $3}' test.p_ctg.gfa > test.p_ctg.fa
```

<!-- pause -->
Does anybody want to guess what it's doing?

Here's a few rows of a `.gfa` file:
```
S  ptg000001l  TCCTGGTGAGGC...  ...
A  ptg000001l  0                ...
A  ptg000001l  271              ...
A  ptg000001l  1642             ...
...
```

<!-- end_slide -->

Heng Li's use of `awk` in `hifiasm`
---
> How do I get contigs in FASTA?

The FASTA file can be produced from GFA as follows:
```bash
awk '/^S/{print ">"$2;print $3}' test.p_ctg.gfa > test.p_ctg.fa
```

Does anybody want to guess what it's doing?

Here's a few rows of a `.gfa` file:
```
S  ptg000001l  TCCTGGTGAGGC...  ...
A  ptg000001l  0                ...
A  ptg000001l  271              ...
A  ptg000001l  1642             ...
...
```

Output would be:
```
>ptg000001l
TCCTGGTGAGGC...
```

skipping all the rows that start with "A" until the next "S" row.

<!-- end_slide -->
SUMMARY AND FURTHER RESOURCES TODO SLIDE
---
Need to add info here now
<!-- end_slide -->
