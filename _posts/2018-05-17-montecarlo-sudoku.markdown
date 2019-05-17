---
layout: post
title:  "Sudoku à la Monte Carlo"
date:   2018-05-17 13:48:00 +0530
categories: [""]
author: "Brian Yee"
---

## Summary:

In this piece an approach for solving generalized sudoku problems is outlined. 
The target audience for this piece is people already familiar with standard MCMC recipes and would like to refer to this as an illustrative example for those not familiar with MCMC.
The unfamiliar with MCMC can likely parse this piece themselves provided they read up on basic MCMC perhaps tthe introduction of [Katzgraber](https://arxiv.org/abs/0905.1629).
Those wishing to implement an optimal sudoku solver should look into [constraint propagation combined with a heuristic depth first search](http://www.norvig.com/sudoku.html).
For those who want a very concise explanation of the strategy I refer you to the following application outlined [here](https://www.lptmc.jussieu.fr/user/talbot/sudoku.html).

Full code to tinker and play with is available on [GitHub](https://github.com/Brian-Yee/montecarlo-sudoku)

## The Approach

We start by reviewing the rules of sudoku and what is meant by a generalized sudoku system.
For that I shall borrow the abstract of it's [Wikipedia Page](https://en.wikipedia.org/wiki/Sudoku)

>The objective is to fill a 9×9 grid with digits so that each column, each row, and each of the nine 3×3 subgrids that compose the grid (also called "boxes", "blocks", or "regions") contain all of the digits from 1 to 9. The puzzle setter provides a partially completed grid, which for a well-posed puzzle has a single solution. 

A typical sudoku puzzle

![Example Image of Unsolved Sudoku Puzzle](https://upload.wikimedia.org/wikipedia/commons/e/e0/Sudoku_Puzzle_by_L2G-20050714_standardized_layout.svg)

and it's solution

![Example Image of Solved Sudoku Puzzle](https://upload.wikimedia.org/wikipedia/commons/1/12/Sudoku_Puzzle_by_L2G-20050714_solution_standardized_layout.svg)

The reader is likely to have encountered them either in the back of a newspaper or on top of a toilet at a friends house in the 2000s, partially solved with a pencil/pencil that unfortunately no longer appears to be located in the bathroom.
It is fairly intuitive to see how the rules should apply to _generalized_ sudoku systems such as _hexadoku_, which instead of using the digits from `0-9` uses the hexadecimal system or _samurai sudoku_ which interlocks multiple sudokus

![Example Image of Samurai Sudoku](../includes/images/montecarlo-sudoku/samurai-sudoku-example.png)

The outline of this post focuses explicitly on solving a Samurai Sudoku puzzle although the techniques are transferrable to other interolcked sudokus or `n`-dokus like hexadoku.

## MCMC Recipe

We start by demonstrating an MCMC recipe for a standard `9x9` sudoku puzzle. 
Consider the following plaintext formatted standard sudoku puzzle taken from [Wikipedia](https://en.wikipedia.org/wiki/Sudoku_solving_algorithms#/media/File:Sudoku_Puzzle_by_L2G-20050714_standardized_layout.svg)

```
5 3 0 0 7 0 0 0 0
6 0 0 1 9 5 0 0 0
0 9 8 0 0 0 0 6 0
8 0 0 0 6 0 0 0 3
4 0 0 8 0 3 0 0 1
7 0 0 0 2 0 0 0 6
0 6 0 0 0 0 2 8 0
0 0 0 4 1 9 0 0 5
0 0 0 0 8 0 0 7 9
```

where `0` indicates a cell free to be chosen by the user. 
We aim to fill this array by replacing all `0`s with a digit from `1-9` such that there exists no duplicates across cells, rows or within a block for any given cell.
A block is defined to be part of a `3x3` sub-array tiling of the full grid.
A standard sudoku has `9` of them

```
 5 3 0 | 0 7 0 | 0 0 0
 6 0 0 | 1 9 5 | 0 0 0
 0 9 8 | 0 0 0 | 0 6 0
-------|-------|-------
 8 0 0 | 0 6 0 | 0 0 3
 4 0 0 | 8 0 3 | 0 0 1
 7 0 0 | 0 2 0 | 0 0 6
-------|-------|-------
 0 6 0 | 0 0 0 | 2 8 0
 0 0 0 | 4 1 9 | 0 0 5
 0 0 0 | 0 8 0 | 0 7 9
```

The initial state of the system is set so that `0`s in each block are filled while ensuring that every block uses all digits from `1-9`.
A state move is defined to be a random exchange of any two free digits within a randomly selected block.
In this way we ensure the ergodicity of a sub-space containing the solution.
We can therefore define the following equation of energy

```
E = duplicates_over_rows + duplicates_over_columns
```

Note that the zero case of this definition corresponds to identify the solution of the sudoku. 
As such if we encounter an energy of `zero` at any point we can terminate the problem and return the state as the solution.
We can concisely summarize the equation above as simply

```
E = duplicates_over_peers
```

where `peers` are defined to be the rows and columns which intersect a given cell.
This redefinition is beneficial as in the simple case of sudoku the peers can be trivially found, whereas in the case of generalized sudokus peers may be slightly more sophisticated.

With these defintions left all that is left is to define an optimal temperature hyperparameter for solving the system. 
By inspection, `T=0.25` appeared to work quite well, so much so that tuning a simulated annealing or parallel tempering system seemed to add needless complexity to the problem.

## Bookkeeping for Generalization

Up till this point the MCMC recipe has been established and all that's left is the bookkeeping required for generalization.
Without loss of generality the process described here is to solve a samurai sudoku system.
We start by reading in a plaintext file that is arranged as one would visually see a sudoku puzzle.

```
0 7 0 8 0 9 0 4 0 . . . 0 1 0 6 0 7 0 8 0
0 0 0 6 0 1 0 0 0 . . . 0 0 0 2 0 9 0 0 0
0 0 8 0 4 0 1 0 0 . . . 0 0 2 0 5 0 4 0 0
7 8 0 0 0 0 0 6 3 . . . 2 3 0 0 0 0 0 7 6
1 0 0 5 8 7 0 0 4 . . . 7 0 0 5 6 8 0 0 3
5 9 0 0 0 0 0 1 8 . . . 8 9 0 0 0 0 0 5 4
0 0 5 0 6 0 0 0 0 2 0 5 0 0 0 0 3 0 7 0 0
0 0 0 7 0 8 0 0 0 8 0 9 0 0 0 4 0 6 0 0 0
0 6 0 4 0 3 0 0 0 0 6 0 0 0 0 9 0 2 0 3 0
. . . . . . 5 7 0 0 0 0 0 1 4 . . . . . .
. . . . . . 1 0 0 7 5 8 0 0 2 . . . . . .
. . . . . . 2 6 0 0 0 0 0 7 5 . . . . . .
0 5 0 1 0 3 0 0 0 0 7 0 0 0 0 1 0 6 0 2 0
0 0 0 4 0 8 0 0 0 6 0 4 0 0 0 3 0 7 0 0 0
0 0 6 0 7 0 0 0 0 5 0 3 0 0 0 0 2 0 1 0 0
1 6 0 0 0 0 0 4 9 . . . 3 5 0 0 0 0 0 9 4
2 0 0 3 9 4 0 0 8 . . . 1 0 0 5 3 4 0 0 7
4 8 0 0 0 0 0 7 3 . . . 4 6 0 0 0 0 0 5 1
0 0 4 0 3 0 9 0 0 . . . 0 0 4 0 9 0 8 0 0
0 0 0 2 0 5 0 0 0 . . . 0 0 0 6 0 3 0 0 0
0 3 0 9 0 7 0 5 0 . . . 0 3 0 4 0 8 0 1 0
```

Where we have added the additional notation of `.` to indicate a forbidden cell that is never used in the system.
To identify sudoku subsystems we simply search for collections of blocks that contain no forbidden cells.

```
----------------- . . . -----------------
|               | . . . |               |
|               | . . . |               |
|               | . . . |               |
|       1       | . . . |       2       |
|               | . . . |               |
|           ----|-------|----           |
|           |   |       |   |           |
------------|----       ----|------------
. . . . . . |               | . . . . . .
. . . . . . |       3       | . . . . . .
. . . . . . |               | . . . . . .
------------|----       ----|------------
|           |   |       |   |           |
|           ----|-------|----           |
|               | . . . |               |
|       4       | . . . |       5       |
|               | . . . |               |
|               | . . . |               |
|               | . . . |               |
----------------- . . . -----------------
```

These blocks are then used to construct all peers that intersect a cell

```
0 0 0 0 0 0 | 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0
----------- X --------------- 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0
. . . . . . | 0 0 0 0 0 0 0 0 . . . . . .
. . . . . . | 0 0 0 0 0 0 0 0 . . . . . .
. . . . . . | 0 0 0 0 0 0 0 0 . . . . . .
0 0 0 0 0 0 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 . . . 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 . . . 0 0 0 0 0 0 0 0 0
```

There can now be either have `2` or `1` corresponding rows/columns but otherwise everything remains the same and a solution can be found.
Don't believe me -- here it is!

```
3 7 1 8 2 9 6 4 5       5 1 3 6 4 7 2 8 9
4 5 9 6 7 1 8 3 2       6 7 4 2 8 9 3 1 5
6 2 8 3 4 5 1 7 9       9 8 2 3 5 1 4 6 7
7 8 2 9 1 4 5 6 3       2 3 5 1 9 4 8 7 6
1 3 6 5 8 7 2 9 4       7 4 1 5 6 8 9 2 3
5 9 4 2 3 6 7 1 8       8 9 6 7 2 3 1 5 4
9 4 5 1 6 2 3 8 7 2 4 5 1 6 9 8 3 5 7 4 2
2 1 3 7 9 8 4 5 6 8 1 9 3 2 7 4 1 6 5 9 8
8 6 7 4 5 3 9 2 1 3 6 7 4 5 8 9 7 2 6 3 1
            5 7 3 9 2 6 8 1 4
            1 4 9 7 5 8 6 3 2
            2 6 8 4 3 1 9 7 5
7 5 8 1 2 3 6 9 4 1 7 2 5 8 3 1 4 6 7 2 9
9 2 1 4 6 8 7 3 5 6 8 4 2 9 1 3 5 7 4 6 8
3 4 6 5 7 9 8 1 2 5 9 3 7 4 6 8 2 9 1 3 5
1 6 3 7 8 2 5 4 9       3 5 8 7 6 1 2 9 4
2 7 5 3 9 4 1 6 8       1 2 9 5 3 4 6 8 7
4 8 9 6 5 1 2 7 3       4 6 7 9 8 2 3 5 1
5 1 4 8 3 6 9 2 7       6 1 4 2 9 5 8 7 3
6 9 7 2 4 5 3 8 1       8 7 5 6 1 3 9 4 2
8 3 2 9 1 7 4 5 6       9 3 2 4 7 8 5 1 6
```

If you are learning monte carlo and are interested in implementing this or reading the code to find some standard tricks that are not expliticitly stated (i.e. efficient energy calculations), I encourage you to check out the repository on [GitHub](https://github.com/Brian-Yee/montecarlo-sudoku).

Happy Hacking!
