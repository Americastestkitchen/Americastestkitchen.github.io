---
layout: post
title:  "Dynamic Programming"
author: Anders Dahl
github_username: AndersOLDahl
date:   2015-07-15 20:00:00
---

#Down with Dynamic Programming

##Three Useful Dynamic Programming Examples:

Dynamic programming refers to a very large class of algorithms. The idea is to break a large problem down (if possible) into incremental steps so that, at any given stage, optimal solutions are known to sub-problems. In other words, you want to use previously calculated optimal-solutions from sub-problems in order to solve the current problem. Since dynamic programming covers such a vast array of problems, it can be helpful to have a template for some of the commonly seen ones.

A lot of the simpler dynamic programming problems can be broken down into three categories often seen in code challenges, classwork, and interviews. The goal is to help you recognize these problems quickly and apply the appropriate strategy. In order to grasp how they differ we need to take a look at some examples.

### Simplest, category 1:

**Maximum Value Contiguous Subsequence.**
> Given a sequence of n real numbers A(1) ... A(n), determine a contiguous subsequence A(i) ... A(j) for which the sum of elements in the subsequence is maximized. This is only interesting if we have negative numbers; otherwise, we would simply take the sum of all the numbers.

We will keep an array, M, of the maximum subsequences we have seen so far. In order to find the optimal window ending at position j, we either need to extend the previous optimal window at M[j-1] or start a new window. These are the only two alternatives we need to consider.

In short this can be represented as:  
A: array of numbers  
M(j): max sum over all windows ending at j  
M(j) = max { M(j-1) + A(j) , A(j) }

As such, this is solved in linear time, and the complexity is O(n).

Here, we only use the previous sub-problem (M(j-1)) in order to solve the question. This leads to an efficient algorithm.

### Moderate, category 2:

**Longest Increasing Subsequence.**
>Given a sequence of n real numbers A(1) ... A(n), determine a subsequence (not necessarily contiguous) of maximum length in which the values in the subsequence form a strictly increasing sequence.

We will keep an array, M, of the longest (the most elements) strictly increasing subsequences we have seen so far. 1,4,8,11 is strictly increasing while 1,4,11,8 is not. We will have to have a conditional in our algorithm to check for this. In order to find the longest strictly increasing subsequence ending at M(j), we want to use the solution from our sub-problems. We represent a sub-problem as M(i). We will have to iterate through all of the M(i)'s and find a maximum where the condition A[i] < A[j] is true. Lastly, we add 1 to account for the new element being added.

In short this can be represented as:  
A: array of numbers  
M(j): longest (the most elements) strictly increasing subsequence ending at position j.  
M(j) = max { M(i) + 1 } over all i < j given that A[i] < A[j]

In the end, we need to iterate through M and find the max length overall. Max { M(j) }.

As such, this is solved in quadratic time, and the complexity is O(n^2). It is possible, however, to get complexity of O(nlog(n)) with such an algorithm, but that is beyond the scope of this article.

Here, we use all of the previous sub-problems (M(1 ... j - 1)) in order to solve the question, but this also leads to a slower implementation than the previous method. It is important to recognize that we need to use all of the sub-problems, because it does not ask for a continuous solution. The optimal solution for M(j) can start anywhere!

### Difficult, category 3:

**This problem can also be refered to as the (0/1) knapsack problem.**

>The input is n items, with integer sizes Wi and values Vi. We have a knapsack of capacity C. We want to fill the knapsack with as much value as possible without going over its weight capacity.

We will keep an array, M(i, j), which will represent the optimal value for filling exactly a capacity j knapsack with some subset of items 1 to i. We can see here that we now have an array of arrays to represent our optimal solution sub-problems.

In total, we have 1 ... n for i and 1 ... C for j.  
W: Array of weights for the items  
V: Array of values for the items  
M(i,j): the optimal value for filling exactly a capacity j knapsack with some subset of items 1 to i.  
M(i,j) = max { M(i-1, j), M(i-1, j-Wi) + Vi }

In the first scenario, the item is not used and we simply use the previous optimal solution. In the second scenario, the item is used, and we have to subtract its weight and add it to the optimal solution for that weight.

In the end, we need to iterate through all of j because we do not know how much of the capacity is going to be used in order to find the knapsack with the optimal value.  

Max { M(n,j) } for all j

As such, this is solved in quadratic Time, and the compexity is O(nC). There are n items and a knapsack going up to capacity C.

**In conclusion**

When solving dynamic programming problems try to think of how the optimal solution sub-problems need to be represented. Can we get away with only using the previous sub-problem? Do we need all our previous sub-problems? Do our sub-problems consist of one or two variables? Once that is decided, it is easier to handle the different scenarios that might arise. You might have a conditional that needs to be added or a computation might be much more complex; however, the underlying base is the same. This should help you out greatly when tackling
these types of problems.

* Case 1: Only prev. sub-problem
* Case 2: All prev. sub-problems, one variable, can be represented as an array
* Case 3: All prev. sub-problems, two variables, can be represented as an array
* of arrays
