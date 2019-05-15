# Generating Beautiful Aperiodic Rep-Tilings

## Summary
Reading time:
Math Concepts:
Programming Concepts:

## The Meta
Recently I went on a trip to Spain and needed some simple concepts to program while in an offline state.
As a bonus, they would ideally be something new and not directly tied to something productive that I had already done.
I figured a geometric problem would be ideal for this as the visual aspects make the conceptualization easy to remember and I hadn't worked on any more math-for-math-sake's
problems for quite some time.
These two projects turned out to be PolyWhirl and Py-Rep-Tile.
PolyWhirl was written and completed on the plane trip over. 
It's a simple little program but generates some surprisingly elaborate images.

## Background

### What are polygon-spirals?

If one takes the midpoints along the edge of any polygon and connects them they will find a smaller version of the same polygon formed.
When this process is recursed indefinitely, if one follows the edges formed by connecting a _vertex_ to a derived _midpoint_ each iteration a spiral can be formed.
This spiral is called a polygon spiral and the area between two spirals in this project is called a _wedge_.

## PolyWhirl

PolyWhirl which allows you to plot all polygon-wedges it's as simple as that.
There is a _modulus_ argument that allows you to plot polygon-wedges at a defined interval.
The math by which to do this is very simple.
Consider a polygon expressed in polar notation that is

