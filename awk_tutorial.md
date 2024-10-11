---
title: awk tutorial and tips and tricks
sub_title: With applications on genomics files
author: Oct 18th 2024, Rob Bierman, Research Software Engineer
theme:
  name: light
  override:
    footer:
      style: template
      right: "{current_slide} / {total_slides}"
---

Goals for this session
---

# History of `awk` as a language and motivator for perl/python/etc
<!-- pause -->
# Learn how `awk` works
<!-- pause -->
# The power of `awk` on .bed and .gtf files
<!-- pause -->
# The power of `awk` in combination with other bash tools
<!-- pause -->
# When to probably not use `awk`

<!-- end_slide -->

Something to look forward to
---
Here's a `data/example.bed` file (with a header) included in this github repo
```bash
chr,start,end
chr1,100,120
chr1,140,160
chr2,560,580
```
<!-- pause -->
`awk` script:

```bash
awk -F',' '{ s+=$3-$2 } END { print s }' data/example.bed
```
<!-- pause -->
result:

```bash
60
```
<!-- pause -->

For pandas users, you could do the same thing with
```python
import pandas as pd
df = pd.read_csv("data/example.bed")
(df['end']-df['start']).sum()
```

<!-- pause -->

For R users, you could do the same thing with
```r
library(tidyverse)
dat = read_csv("data/example.bed")
sum(dat['end']-dat['start'])
```

<!-- end_slide -->

TODO SLIDE
---

Need to add info here now
