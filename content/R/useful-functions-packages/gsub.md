+++
title = 'gsub'
date = 2024-02-20T08:53:29-05:00
draft = false
+++

### GSUB
I use gsub all the time to swap in or out patterns or to get parts of a string

```r
# simple usage of gsub

gsub(pattern to match, pattern you want, what you want to change)
# For example, let's say we need to get rid of "-" in sample names and replace them with "_"
gsub("-", "_", sample_names)
```

```r
# using gsub to gather part of a string:

# Let's say you need to get the unique ID number from each patient ID

Patient_ID <- c("001-111", "001-112", "001-113")

gsub("001-", "", Patient_ID)
# Which will return "111", "112" and "113"
```

```r
# removing everything before or after a pattern

# Let's say we're in the same situation where we need to get the unique portion of a patient ID
patient_ids <- c("001-abc-123", "001-abc-124", "001-abc-125")

# I can choose to remove everything before the "c-" to just keep the unique portion
# the ".*" before the pattern idicates remove everything before this pattern including the "-c"
gsub(".*c-", "", patient_ids)
# [1] "123" "124" "125"


# You can do the opposite as well and remove everything after your pattern by the moving the ".*" to after your pattern
# here I'll remove everything after the -1 in the string (including the -1)
gsub("-1.*", "", patient_ids)
# [1] "001-abc" "001-abc" "001-abc"
```