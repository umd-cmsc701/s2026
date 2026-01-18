---
type: assignment
date: 2025-03-06
title: 'Bitvector Rank and Select & Sparse Array'
published: false
due_event: 
    type: due
    date: 2025-03-27 23:59:00
    description: 'Assignment Due'
---

# Overview: Implementing bitvector rank and select, and applying them to a sparse array

This assignment is due by **11:59PM ET on March 27**.  It consists of 4 executables, which center around bitvector rank select and access, as well as putting this data structure to use to represent a sparse array.  
The programming tasks build upon each other, and so should be implemented **in order**. 

## Overall structure

You will submit your assignment as a tarball named `CMSC701_A2.tar.gz`.  When this tarball is expanded, it should create a
**single** folder named `CMSC701_A2`.  This folder must be created in the directory where the decompression (i.e. `tar xzvf`) is done, and must not be nested inside any other folders. The details of how you structure your "source tree" are up to you, but the following **must** hold (to enable proper automated testing of your programs).

 * There should be a script at the top-level of `CMSC701_A2` called `build.sh`.  This should do whatever is necessary to create 4 executables at the top level (one called `rsbuild` and one called `rsquery_rank`, one called `rsquery_select`, and one called `sarray`).  If you're comfortable with Makefiles, this can just call `make`, or it could simply run the commands necessary to compile your programs and copy them to the top-level directory.  You can assume this script is run in a `bash` shell.
 
 * There should be a README.md file in the top level directory.  This README file should contain the following information.
     
     - What language have you written your solution in?
     - What did you find to be the hardest part of this assignment?
     - What resources did you consult in working on this assignment (view this as a form of citation; you shouldn't _copy_ code directly from anywhere in your assignment, but if you consulted other sources please list them here).

**Turnin** : The assignment turnin will be handled using Gradescope.  

**Sample Data** : Sample data with expected input and output is available [here](https://github.com/umd-cmsc701/project_2_sample_data).

## Task 0 — Constructing a rank-enabled bitvector.

Implement a succinct, constant-time, bit-vector rank operation.  The exact details of the design and implementation are up to you, but you should implement Jacobson's rank, as we discussed in class.
**In particular**, it will be **much** easier for you if you use the two-level index (with superblocks and blocks), and solve the within-block rank using the combination of the mask and popcount instruction. In this case
your index will consist of the original bitvector, and the two-level (superblock and block) index.

Also, note that you **must** implement your index using the appropriately-sized packed integers.  That is, if the bitvector you are provided with is 262,144 bits long, then your superblock rank counters should use 18 bits each, not 32
or 64.  In other words, you should use the data structure commonly referred to as an "int vector" --- an array that can represent a collection of integers of some fixed bit-width different than the built-in types.  You are welcome to implement this data structure yourself, but you do not have to.  You can use an existing implementation of an int vector, but be sure to document this in your readme (and, of course, if that int vector also has a rank and select implementation, you **may not** use those directly).     

Your first program will be an executable called `rsbuild`. The `rsbuild` executable will take as input a _binary_ file containing the contents of a bitvector in the format described below, as well as the path to where the output file 
should be written. The program should read the input bitvector from file, build a rank data structure, and write an outputfile containing **both the bitvector itself** (for purposes of access) as well as your rank data structure.  So the inputs to your program are:

 * `input_bitvector_file` : path to a file containing the input bitvector in a binary format (described below)
 * `output_index_path` : path to a file where you should write the **binary** representation of the bitvector and your rank index

The input data is in a little-endian binary file. The first entry in the file is a 4-byte unsigned integer (i.e. a 32-bit integer) that provides the size of the subsequent vector **in bits**.  Let this number be `N`. What follows is an array of `ceiling( n / 8 )` bytes that constitute the actual bitvector.  Whatever pattern of 0's and 1's are encoded in these bytes, themselves, defines the content of the bitvector input.  To make things easier, we have created input so that `n` is always a multiple of 8.
 
## Task 1 — Query for ranks in your bitvector

Given the index you built above, write a program `rsquery_rank`. The `rsquery_rank` program will 3 inputs on the command line. They are as follows:

  * `index` : The path to the bitvector with rank index that was computed by `rsbuild` as described above
  * `query_file` : A text-format query file (structure defined below) that consists of a set of rank queries
  * `output_path` : The path to a file where you should write (in text) the output of the queries

The `query_file` is a line-oriented file where each line contains a single query. Each line is a whitespace separated pair of tokens of the form:

```
rank <idx>
```

where `rank` is the literal string "rank" and `<idx>` is some integer between 0 and the length of the bitvector (**not necessarily less than the maximum rank**).

Your output should be one line for each query of the form:

```
<idx>:<rank>
```

That is, a `:` separated pair of integers where the first integer is the `<idx>` requested in the corresponding query and the second integer is the rank of the bitvector at this index. Your ouput file should have one line per input query.  In the event that the `<idx>` requested is beyond the bounds of the bitvector, you should return `-1` as the result of the rank query.

## Task 2 — Answer select queries in your bitvector

Given the index you built above, write a program `rsquery_select`. The `rsquery_select` program will 3 inputs on the command line. **Note**: We _do not_ expect that you will implement the `O(1)`-time select data structure to answer these select queries. Rather, you should use the rank data structure built in the first part of this assignment (and used in the second part), along with binary search, to answer each select query in `O(lg N)` time (where `N` is the length of the bitvector). The input for your program is as follows:

  * `index` : The path to the bitvector with rank index that was computed by `rsbuild` as described above
  * `query_file` : A text-format query file (structure defined below) that consists of a set of select queries
  * `output_path` : The path to a file where you should write (in text) the output of the queries

The `query_file` is a line-oriented file where each line contains a single query. Each line is a whitespace separated pair of tokens of the form:

```
select <rank>
```

where `select` is the literal string "select" and `<rank>` is some integer between 0 and the length of the bitvector (**not necessarily less than the maximum rank**).  Note, this means that it is expected that you will get select queries beyond the maximum rank of your bitvector. You should detect this appropriately and answer as described below.

Your output should be one line for each query of the form:

```
<rank>:<select>
```

That is, a `:` separated pair of integers where the first integer is the `<rank>` requested in the corresponding query and the second integer is the index of the highest-indexed bit in the bitvector having this rank (i.e. it is the result of the `select1` query for the provided `<rank>`). Your ouput file should have one line per input query.  In the event that the `<rank>` requested is beyond the maximum ranked bit, you should return `-1` as the result of the selectquery.


## Task 3 — Implementing a sparse array using your bitvector rank and select

Using the bitvector rank and select implementation you have built above, you will build a "sparse array" that stores a collection of strings and associates with each one an index.  The idea here is that the values themselves will be tightly packed, but the index associated with each will be derived from a (potentially sparse) bitvector.  Consider the following example:

bitvector:

```
| 0 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 1 |
```

values:

```
["foo", "bar", "baz"]
```

Note that we have a bitvector of length 10, but only 3 non-zero values. This representation will be much more efficient than storing e.g. a list of 10 strings, only 3 of which are non-empty, and even more efficient at this sparsity than storing a list of string, index tuples.  As we will see later in the course, this type of representation is very useful for representing e.g. a _sampled_ suffix array, which we will use for full-text search with the FM-index.

Critically, the sparse array should be able to efficiently return the i-th present element of the array, along with its index of occurrence, it should be able to return the element present at a given index (or an empty indicator if no such element is present).

The exact design of your sparse array program is up to you, but it should use a rank-enabled bitvector from the first part of the assignment above. Your program should be called `sarray`, and it should take 2 inputs on the command line, which are the following:

  * `command_script` : A list of commands to populate and query the sparse array, whose format is detailed below.
  * `output_path` : The path to an ouput location where the results of the commands above will be written.

The `command_script` consists of a series of commands to build a sparse array, and then to issue different queries.  The first command will always be `init N` where `N` is the number of bits that should be in your sparse array's bitvector.  Then, a series of `insert` commands will follow. Each insert command is of the following format:

  * `insert <pos> <key>` : where `<pos>` is some index in `[0, N)` and `<key>` is a string that should be associated with this position.

The insertion commands need not appear in order. That is, it is possible to insert a key at position `p1` and then later `p2` where `p2` < `p1`.  However, all of the insert commands will be issued before any queries or other commands are issued.  At the end of the series of insert commands, a `finish` command will be issued. When you encounter the `finish` command, you should then actually construct your sparse array index (i.e. put the keys in the appropriate order and build the rank data structure on your bit vector).

Finally, after the `finish` command, a series of queries will be issued.  Each possible type of query is listed below, along with the expected output that should appear, corresponding to this query, in the output file. Note that each query occurs by itself on an input line, and so the number of lines in the output should be exactly equal to the number of queries (i.e. the number of lines in the input file, minus the `init`, `finish` and `insert` commands).

Commands
--------

  * `query_index <idx>` : The `query_index` command provides some `<idx>` in `[0,N)` and returns the key (string) associated with this index. If there is no key associated with this index, it should return `-1`. When your program is issued a `query_index <idx>` command, it should output a response of the format `idx:<idx>:<key>` or `idx:<idx>:-1` if `<idx>` is not occupied by a key.

  * `query_rank <rank>` : The `query_rank` command provides some `<rank>` in `[0,N)` and returns the key (string) associated with this **rank**. That is, for rank `<rank>` it should return the `<rank>-th` key in the ordered list of keys. If there is no key associated with this rank (i.e. if `<rank>` is greater than the number of inserted keys), it should return `-1`. When your program is issued a `query_rank <rank>` command, it should output a response of the format `qr:<rank>:<key>` or `qr:<rank>:-1` if `<rank>` is not a valid rank for a key.


  * `rank_at_index <idx>` : The `rank_at_index` command provides some `<idx>` in `[0,N)` and returns the rank in the underlying bitvector at the associated index. That is, for `<idx>` it should return the rank of `b[idx]` in the underlying bitvector `b` (regardless of if `b[idx]` is a `0` or a `1`). When your program is issued a `rank_at_index <idx>` command, it should output a response of the format `rai:<idx>:<rank>`.


  * `index_of_next_key <idx>` : The `index_of_next_key` command provides some `<idx>` in `[0,N)` and returns the index in the sparse bitvector corresponding to the **next** present key. That is, it returns the smallest index `<pos>` of the next `1` in the bitvector such that `<pos>` > `<idx>`. If there is a key present at `<idx>` it does not return `<idx>` itself, but the **next** index containing a present key. If there are no `1`s in the bitvector beyond `<idx>` it returns `-1`.  For every `rank_at_index <idx>` query, your program should output a response of the format `ink:<idx>:<pos>` where `<pos>` is as just described or `ink:<idx>:-1` if `<idx>` is past the last `1` in the bitvector.

  * `density` : The `density` command provides no value. Rather, it asks for you to respond with the density of your bitvector (i.e. the number of `1`s divided by the length of the bivector `N`). For every instance of the `density` command your program recieves, it should respond with an identical result, a single floating point number (with at least 5 digits of precision) providing the density of the sparse array.


## Notes

The bitvector rank and select for part 1 will each be tested on 2 different inputs with a variety of queries. The grading results will be evaluated per-query (like the suffix array query results in the first assignment), though it is unlikely that if your index is incorrect, that it will answer many queries correctly.

Likewise, the sparse array program will be graded on a per-query basis, so you will gain points for correct query results even if all results are not correct. So, for example, the `query_index` and `density` commands are easy to implement even without a working rank index, and you could try to come up with valid implementations of those first.

We will provide some test input and expected output against which you can check your implementation, to allow faster testing without involving gradescope.

Finally, while we won't be doing a complete code review, we do plan to read and evaluate some of the code manually, to ensure that the constraints listed in the specification 
are being met (i.e. that integers are properly bitpacked to the minimal size, etc.).
