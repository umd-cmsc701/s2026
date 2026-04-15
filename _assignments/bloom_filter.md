---
type: assignment
date: 2026-04-15
title: 'Building and Querying a Bloom Filter'
published: true
due_event:
  date: 2026-04-29
  type: due
  description: "Assignment 3 due"
---

# Overview: Implmenting a Bloom Filter

This assignment is due by **11:59PM ET on April 29**.  It consists of 1 executable (which implements 2 sub-commands). One for building a 
Bloom filter and the other for querying the Bloom filter.

## Overall structure

You will submit your assignment as a tarball named `CMSC701_A3.tar.gz`.  When this tarball is expanded, it should create a
**single** folder named `CMSC701_A3`.  This folder must be created in the directory where the decompression (i.e. `tar xzvf`) is done, and must not be nested inside any other folders. The details of how you structure your "source tree" are up to you, but the following **must** hold (to enable proper automated testing of your programs).

 * There should be a script at the top-level of `CMSC701_A3` called `build.sh`.  This should do whatever is necessary to create an executable at the top level called `bfilt`.  If you're comfortable with Makefiles, this can just call `make`, or it could simply run the commands necessary to compile your programs and copy them to the top-level directory.  You can assume this script is run in a `bash` shell.
 
 * There should be a README.md file in the top level directory.  This README file should contain the following information.
     
     - What language have you written your solution in?
     - What did you find to be the hardest part of this assignment?
     - What resources did you consult in working on this assignment (view this as a form of citation; you shouldn't _copy_ code directly from anywhere in your assignment, but if you consulted other sources please list them here).

**Turnin** : The assignment turnin will be handled using Gradescope.  

## Sample data

Sample data for testing your implementation locally is available [here](https://github.com/umd-cmsc701/bloom_filter_test_data).

## Task 1 — Constructing a Bloom Filter.

Implement the construction of a Bloom filter from a target false positive rate (FPR) and an input file containing a list of elements in the approximate set.  The specific hash function that you choose to use for your Bloom filter is up to you, but you should use a proper bit vector for the actual Bloom filter implementation and should construct the Bloom filter with the optimal number of hash functions and size based on the target FPR and the number of distinct elements in the input set. **Hint**: Recall that when building multiple different hash functions `h_1`, `h_2`, ..., `h_k`, it is sufficient to just call the same hash function with a different set of distinct (but fixed) seed values.  The set of seed you use, if not hard-coded in the program somehow, should be stored along with your Bloom filter to ensure that you can properly query the data structure.

Your `bfilt` program will accept a sub-command called `build` that takes as input 3 additional arguments. These inputs will be, respectively, a floating point number representing the desired false positive rate of the Bloom filter, and the path to an input file containing a collection of strings (one per line) which are the keys to be inserted into the constructed filter.  **Note**: The input **collection** need not be a set (i.e. it can contain duplicates).  You should de-duplicate the keys before computing the optimal parameters for your Bloom filter and constructing it. The final argument is the path to an output file where your constructed Bloom filter should be written.  So the inputs to your program are:
 
 * `build` : The literal string "build" specifies that the build sub-command of the `bfilt` binary should be executed.
 * `fpr` : The false positive rate for your Bloom filter; a floating point number between 0 and 1
 * `input_collection` : path to a file from which you should read the collection of keys on which your Bloom filter should be constructed.
 * `output_path` : path to a file where you should write the **binary** representation of the Bloom filter

Your program should read in the `input_collection`, deduplicate the keys, and then write the following to standard out:

```
n  <distinct_keys>
m  <optimal_width>
k  <optimal_num_hashes>
```

where `k`, and `m` are as we discussed in class, the optimal number of hash functions `k` and bitvector width `m` required to achieve the target false positive rate `fpr` on an input set of `n` elements.
Recall that these values (`k` and `m`) can be derived analytically from `n` and `fpr`. **Of particular importance**, since we will redirect standard out to record these values and evaluate your answers during the `build` step of your Bloom filter, it's important that you not write any other information to standard out in your `bfilt build` command. If you need debugging output, write to standard error. Finally, when computing the valeus for `m` and `k`, remember that these are expected to be integers. When you need to obtain an integer in your final calculation, we recommend taking the ceiling of the relevant fractional quantities (this can lead to a slightly more conservative filter, but this is what we expect).

## Task 2 — Query your Bloom filter

Given the index you built above, write a subcommand `bfilt query`. In query mode, the `bfilt` command will take 3 arguments:

  * `query` : The literal string "query" specifies that the query sub-command of the `bfilt` binary should be executed.
  * `bloom_file` : The serialized Bloom filter that you build above using the `build` sub-command.
  * `query_file` : A text format file containing a collection of strings, one per line, that will be used to query the Bloom filter.

Your program should load your Bloom filter from `bloom_file` and then read through the queries in `query_file` one by one. This file will contain both positive and negative keys (i.e. elements in your original set and elements not in your original set).  For each query, you should write a single output line to standard out.  Specifically, for a key `K`, if your Bloom filter answers yes for the presence of `K` in the filter, you should write:

```
K  PROB_YES
```

to standard out.  On the other hand, if your Bloom filter answers no to the presence of `K` you should write:

```
K  NO
```

Recall that your Bloom filter may have some false positives (close to the target FPR rate), but should have no false negatives.  **Note**: There should be one output line written to standard out for each input line in the query file, and the lines should appear in the same order (i.e. the first output line should correspond to the first query, etc.). Finally, as with the `build` command, since we'll be redirecting standard out to record your program's output for evaluation, you shouldn't use it to print any other information. If you need to write diagnostic information, you should use standard error for that purpose.

## Evaluation

Your program will be evaluated by assessing (a) the accuracy of the `n`, `m`, `k` values computed in the `build` command and (b) the "accuracy" of your solution's `query` command on a set of query files. For accuracy, we will assess both the observed FPR (which should be _close_ to the requested FPR with which the filter was built) and the presence of any false negatives.
