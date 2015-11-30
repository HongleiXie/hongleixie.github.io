---
layout: post
title:  "SAS tricks: Assign the same value within group"
date:   2015-11-30
---
<p><span class="dropcap">W</span>e often use `NODUPKEY` or `NODUP` with `BY` statement to filter out duplicates in terms of specified variables in `BY`. However, the problem I dealt with today seemed to be a bit tricky. And I surprisingly found out how powerful `RETAIN` statement is! Let me simplify the problem as follows: </p>
The dataset, `test`, just for an example, looks like:
```sas
data test;
 input id bus$;
 datalines;
 1 a
 1 a
 2 b
 2 c
 3 d
 3 d
 3 d
 4 e
 4 e
 4 f
 5 a
 5 b
 5 a
 6 c
 7 e
;
```  
Each observation is grouped by `id`.  I want to add a variable`flag` following the logic: 
- **Rule 1**: If it's unique in the combinations of `id` and `bus`, `flag = 1`;
- **Rule 2**: If any one of records flag is assigned to `1` then all records within the same group (i.e. having the same `id` value) will also have `flag = 1`
So the desired output table should look like: 
```sas
id bus flag 
1 a 0 
1 a 0 
2 b 1 
2 c 1 
3 d 0 
3 d 0 
3 d 0 
4 e 1 
4 e 1 
4 f 1 
5 a 1 
5 b 1 
5 a 1 
6 c 1 
7 e 1 
```
I firstly came up with a very simple solution:
```
proc sort data = test nodupkey dupout = dup out = out;
  by id bus;
 run;
 
 proc sort data = test;
  by id bus;
 run;

data temp;
  merge test dup(in = a);
  by id bus;
  if a then flag = 0; else flag = 1;
 run;
```
Do you find the problem here? By doing so, it will not meet the second requirement because the output will flag record `4 e` to `0` but flag observation `4 f` to `1`! 
The trick I later thought of was to sort by `flag` in descending order and assign the rest of observations within the group `flag` value as same as the first one in the group. 
```
proc sort data = temp;
 by id descending flag bus;
run;

data output;
  retain first_id;
  set temp;
  by id;
  if first.id then do;
    first_id = flag;
  end;
  else do;
    flag = first_id;
  end;
run;
```
The `RETAIN` prevents `first_id` from being reset to missing for each iteration. 
