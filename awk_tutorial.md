---
title: awk-tober tutorial and tips and tricks
sub_title: With applications on genomics files
author: Oct 18th 2024, Rob Bierman, Research Software Engineer
theme:
  name: catppuccin-latte
  override:
    footer:
      style: empty
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
# Using `awk`'s associative arrays
<!-- pause -->
# Two nice `awk` "tricks" for multiple files
<!-- pause -->
# When to probably not use `awk`

<!-- end_slide -->

Let's see some awk
---

Here's a `data/quotas.tsv` file included in this github repo
```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
...
```
<!-- pause -->
If we want to print just the `User's` column (the 3rd), we can use `awk`!

```bash
awk '{ print $3 }' data/quotas.tsv
```

<!-- pause -->
Output:
```
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
    * when you have `ls` you'll also have `awk` [citation-needed]
<!-- new_line -->
<!-- pause -->
* `awk` is really great when working with tabular text like CSV or TSV
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

```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
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
```
User     projects   scratch
tcomi     136.800     0.000
rb3242     62.000    10.100
limingli    0.000    42.600
...
```
<!-- end_slide -->

Understanding the structure of an `awk` program: Column re-order
---
Let's return to the `data/quotas.tsv` dataset

```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
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

```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
...
```

and generate:
```
tcomi     136.800     0.000
rb3242     62.000    10.100
limingli    0.000    42.600
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

```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
...
```

and generate:
```
tcomi     136.800     0.000
rb3242     62.000    10.100
limingli    0.000    42.600
...
```

We can tell `awk` to skip the first line with the built in `NR` variable:
```bash
awk 'NR > 1 { print $3,$1,$2 }' data/quotas.tsv
```
<!-- pause -->

`NR` is one of a handful of special variables! It keeps track of the current line number!
<!-- pause -->

What if we wanted to calculate the sum quota of a user on scratch and projects?

<!-- end_slide -->

Understanding the structure of an `awk` program: Sums
---
We'll start with the same file:
```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
...
```

but now we'll make:
```
tcomi    136.8
rb3242    72.1
limingli  42.6
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
```
projects  scratch   User
 136.800    0.000   tcomi
  62.000   10.100   rb3242
   0.000   42.600   limingli
...
```

but now we'll make:
```
USER     SUM
tcomi    136.8
rb3242    72.1
limingli  42.6
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
    * awk splits on whitespace by default, or specify with `-F`

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

* and since `python` (1991) was inspired by `perl` (1987)
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
      COBOL(1959)  Fortran(1957)            Lisp(1958)
                      |                        |
           ╭----------┴---╮                    |
      ALGOL(1960)   BASIC(1964)                |---------╮
           |                                   |         |
           ╰---╮----------╮                    |         |
          C(1972)  Pascal(1970)     AWK(1977)  |  Smalltalk
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
df = pd.read_csv("data/tiny.bed", sep=" ")
(df['end']-df['start']).sum()
```

<!-- pause -->

For R-tidyverse users, you could do the same thing with
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

We create a variable `s` which defaults to `0` and then we add
the value of the span, end minus start, or `$3-$2`.

At `END` of the program, we print out the value of `s`.


What about the header line?
<!-- pause -->
`awk` couldn't convert `start` and `end` to numeric, so defaulted to 0!
Quirky!

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

`awk` arrays on Vernot 2016 `S*` calls
---
For our next `awk` example we'll use a file of `S*` called introgression
regions from Vernot et al 2016.

I've made a copy of this file on della here:
```
/projects/AKEY/akey_vol2/rbierman/callset.bed
```


this is again a `.bed` file with chrom, start, end, but also with
individual, category, and ancient-source:
```
1 1880284 1891263 1.NA18992.2 NA18992 EAS neand
1 2250695 2299059 1.HG01851.2 HG01851 EAS neand
1 2250695 2300081 1.HG00701.1 HG00701 EAS neand
1 2250695 2300081 1.HG01867.2 HG01867 EAS neand
1 2250695 2300081 1.HG01874.1 HG01874 EAS neand
1 2250695 2300081 1.NA18617.2 NA18617 EAS neand
...
1 1904910 1919381 1.UV500.1   UV500   PNG den
1 1927030 1959261 1.UV919.2   UV919   PNG den
1 1927030 1959261 1.UV929.2   UV929   PNG den
```
<!-- pause -->
How many times does `neand`, `den`, or anything else come up
as the last column?

<!-- pause -->
Honestly, I'd normally do this with `cut`, `sort`, and `uniq -c`:
```
cut -f7 callset.bed | sort | uniq -c
```

<!-- end_slide -->
Counting occurrences of neand, den, etc with `awk`
---

Piped approach:
```
cut -f7 callset.bed | sort | uniq -c
```

Dataset:
```
1 1880284 1891263 1.NA18992.2 NA18992 EAS neand
1 2250695 2299059 1.HG01851.2 HG01851 EAS neand
1 2250695 2300081 1.HG00701.1 HG00701 EAS neand
1 2250695 2300081 1.HG01867.2 HG01867 EAS neand
1 2250695 2300081 1.HG01874.1 HG01874 EAS neand
1 2250695 2300081 1.NA18617.2 NA18617 EAS neand
...
1 1904910 1919381 1.UV500.1   UV500   PNG den
1 1927030 1959261 1.UV919.2   UV919   PNG den
1 1927030 1959261 1.UV929.2   UV929   PNG den
```

Let's make an `awk` solution using arrays
<!-- pause -->

```
awk '{ a[$7]+=1 } END { for(k in a){ print k,a[k] } }' callset.bed
```

We've created an array called `a`, but we could have used any name.
Then for each line we increment the value of the `a` array using column
`7` as the key.

Finally, at the end, we loop through the keys of `a` and print out the info.

<!-- end_slide -->


How about calculating the per-person bed coverage of `den`?
---
Starting with:
```
1 1880284 1891263 1.NA18992.2 NA18992 EAS neand
1 2250695 2299059 1.HG01851.2 HG01851 EAS neand
...
1 1904910 1919381 1.UV500.1   UV500   PNG den
1 1927030 1959261 1.UV919.2   UV919   PNG den
1 1927030 1959261 1.UV929.2   UV929   PNG den
```

We want:
```
PERSONCODE1 SUM_DEN_SEQ_RANGES
PERSONCODE2 SUM_DEN_SEQ_RANGES
PERSONCODE3 SUM_DEN_SEQ_RANGES
...
```

<!-- pause -->
We can make an array `a` keyed by person (`$5`) which stores a running
total of the sequence span (`$3-$2`) ONLY for rows where the 7th column is `den`.
```
awk '$7=="den" {a[$5]+=$3-$2} END {for(k in a){print k,a[k]}}' callset.bed
```

<!-- pause -->
with pandas, if you wanted an ugly one-liner (columns start from 0):
```python
df = pd.read_table("callset.bed", header=None)
df[df[6].eq('den')].groupby(4).apply(lambda g: (g[2]-g[1]).sum())
```

<!-- end_slide -->

`awk` trick to split one file into many
---
Starting with the same `callset.bed`:
```
1 1880284 1891263 1.NA18992.2 NA18992 EAS neand
1 2250695 2299059 1.HG01851.2 HG01851 EAS neand
...
1 1904910 1919381 1.UV500.1   UV500   PNG den
1 1927030 1959261 1.UV919.2   UV919   PNG den
1 1927030 1959261 1.UV929.2   UV929   PNG den
```

Let's say we wanted to put the `neand`, `den`, etc lines
into separate files like `neand_lines.bed`, `den_lines.bed`?

<!-- pause -->
with `awk` you can print to files, not just `stdout`!

```bash
awk '{ print > $7"_lines.bed" }' callset.bed
```

<!-- pause -->
this is printing the entire current line to a file
we are creating with a name that depends on the
`$7` column: `$7"_lines.bed"` (this is str concat).


<!-- pause -->
Only one more example!

<!-- end_slide -->

`awk` trick to concatenate files, keeping only first header
---
We have the GDP in millions of dollars for the 50 states split
across 5 different files `data/f1.tsv`, `data/f2.tsv`, ... , `data/f5.tsv`

Here's what each file looks like, but with different states:

<!-- column_layout: [2, 2] -->
<!-- column: 0 -->
`data/f1.tsv`
```
State        GDP_2022
Alabama       281,569
Alaska         65,699
Arizona       475,654
Arkansas      165,989
California  3,641,643
Colorado      491,289
Connecticut   319,345
Delaware       90,208
```
<!-- column: 1 -->
`data/f2.tsv`
```
State        GDP_2022
Florida     1,439,065
Georgia       767,378
Hawaii        101,083
Idaho         110,871
Illinois    1,025,667
Indiana       470,324
Iowa          238,342
Kansas        209,326
```
<!-- reset_layout -->
<!-- pause -->
If we wanted to create a single file, we could try `cat data/f*.tsv > all.tsv`,
but then we'd get the header multiple times, and internally.

<!-- pause -->
Instead we can use an `awk` trick

```
awk 'NR > 1 || NR == FNR { print }' data/f*.tsv > all.tsv
```

This relies on `awk`'s `NR` and `FNR` variables.
* `NR` : row number that DOESN'T start over between files
* `FNR`: row number that starts over between files
* so `NR == FNR` is true for all rows of just the first file

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
Having said that, even Heng Li suggests using `awk` to process `hifiasm` output!

Here's an entry in the `hifiasm` FAQ

> How do I get contigs in FASTA?

The FASTA file can be produced from GFA as follows:
```bash
awk '/^S/{print ">"$2;print $3}' test.p_ctg.gfa > test.p_ctg.fa
```

<!-- pause -->
What is this command doing?

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

The command operates only on lines that start with `S` (`/^S/`).

Then it prints the 2nd column, newline, then the 3rd column.

<!-- end_slide -->
Summary and further resources
---
Here's what we spoke about today:
# We'll start with an awk **amuse-bouche**
# History of the `awk` language
# Learn how `awk` works with a "quota" dataset
# `awk` as the inspiration for perl, python, and more
# The power of `awk` on .bed and .gtf files
# Using `awk`'s associative arrays
# Two nice `awk` "tricks" for multiple files
# When to probably not use `awk`

If `awk` is something you want to commit to learning more of,
then I'd suggest working with chatGPT.

It's really good at writing and explaining `awk`!

<!-- end_slide -->
