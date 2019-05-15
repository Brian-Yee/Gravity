## Summary:
In this post I outline what an aperiodic rep-tile is and discuss how to write a program to construct a tiling of them.
Readers will learn about: periodicty, tiling, geometry and using polar coordinates, as well as seeing some pretty beautiful results from recreational mathematics

Full code to tinker and play with is available on [GitHub](https://github.com/Brian-Yee/Py-Rep-Tile)

## The Meta
Recently I went on a trip to Spain and needed some simple concepts to program while in an offline state.
As a bonus, they would ideally be something new and not directly tied to something productive that I had already done.
I figured a geometric problem would be ideal for this as the visual aspects make the conceptualization easy to remember and I hadn't worked on any more math-for-math-sake's
problems for quite some time.
These two projects turned out to be PolyWhirl and Py-Rep-Tile.
While PolyWhirl was written rather quickly Py-Rep-Tile took a bit longer identifying an ideal solution. 

## Background

### What is a Rep-Tile?

A rep-tile is a portmanteau of _repeating tile_ that is it is a tile that one can tesselate a copy of itself.
This property is called [self-similarity](https://en.wikipedia.org/wiki/Self-similarity), which means the whole has the same shape as one or more of its parts.
The tile can be mirrored and rotated but the perimeter is rigid and never augmented.
Here are a few examples of rep-tile's taking from the [wikipedia page](https://en.wikipedia.org/wiki/Rep-tile).

### What is aperiodicity?

[Aperiodicity](https://en.wiktionary.org/wiki/aperiodic) is the trait of not being _periodic_, which is to say that there is no repeating pattern spaced across some given interval such as space or time.
A clock is periodic because after 24 hours the hands will be in the same state as before;
my sneezes are aperiodic because there is no defined interval of time that you can guarantee I will sneeze.

## Aperiodic Tilings

An aperidoic rep-tile is a self-similar a aperiodic tiling across the two-dimensional space.
Given a rep-tile tiles _itself_ it is useful to construct a `subdivide` function that takes a reptile and then partitions it into copies of itself.
The easiest way to do this is to choose three central points of a tile an _origin_, _index_, and a _thumb_.

- An _origin_ is a point common to an _index_ and _thumb_
- A _thumb_ is the shortest edge touching the _origin_
- An _index_ is the longest edge touching the _origin_

The naming of these terms are chose because the _origin_ is used as a reference point to translate points back to the _origin_ of a plane where mathematical operations are easier.
The _index_ and _thumb_ are chosen to highlight the [_chirality_](https://en.wikipedia.org/wiki/Chirality_(mathematics)) present in the problem from flipping a tile.
The sign of the cross product of your _index_ and _thumb_ tells you whether a tile was flipped or not.

There are two aperiodic reptiles I implemented the [pinwheel](https://en.wikipedia.org/wiki/Pinwheel_tiling) and [sphinx](https://en.wikipedia.org/wiki/Sphinx_tiling). Below are the diagrams showing their geometry

### Pinwheel

![Pinwheel Digram](https://raw.githubusercontent.com/Brian-Yee/brian-yee.github.io/master/_includes/images/aperiodic-tiling/pinwheel-diagram.png)

### Sphinx
![Sphinx Digram](https://raw.githubusercontent.com/Brian-Yee/brian-yee.github.io/master/_includes/images/aperiodic-tiling/sphinx-diagram.png)

We can then write all of the coordinates in polar form using the _origin_ as our reference point.

![Pinwheel equations](https://raw.githubusercontent.com/Brian-Yee/brian-yee.github.io/master/_includes/images/aperiodic-tiling/pinwheel-eq.png)

Or in the case of the sphinx tiling, define a subgrid
               
![Sphinx grid](https://github.com/Brian-Yee/brian-yee.github.io/blob/master/_includes/images/aperiodic-tiling/sphinx-grid-eq.png)

and rewrite all equations as part of the grid

![Sphinx equations](https://raw.githubusercontent.com/Brian-Yee/brian-yee.github.io/master/_includes/images/aperiodic-tiling/sphinx-eq.png)

In the case of tiles which have been mirrored the signage of _alpha_ is simply reversed. 
As mentioned above this sign is easily obtained by crossing the vectors resulting from the _thumb_ and _index_ vertices in relation to the _origin_.
After that it is just a simple case of writing iteratively subdividing

```
for _ in range(iterations - 1):
    new_rep_tiles = []
    for rep_tile in rep_tiles:
        new_rep_tiles += rep_tile.subdivide()
    rep_tiles = new_rep_tiles

```

and then rendering the end result!
For additional prettiness the phase of a key edge is assigned a colour to help aid in distinguishing subtiles after subdivision.
It was found to be a surprisingly non-trivial to define a way to colour triangles well due to the chirality of the problem.
Nonetheless a simple method was implemented that is self-consistent if not ideal and can be easily overwritten in the future if anyone else has a better colouring scheme.
Gorgeous!

![Pinwheel tiling](https://raw.githubusercontent.com/Brian-Yee/brian-yee.github.io/master/_includes/images/aperiodic-tiling/pinwheel.png)

![Sphinx tiling](https://raw.githubusercontent.com/Brian-Yee/brian-yee.github.io/master/_includes/images/aperiodic-tiling/sphinx.png)
