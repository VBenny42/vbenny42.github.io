---
title: "Advent of Code 2024 Day 12 – Garden Groups"
layout: post
date: 2024-12-12 22:46
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 12 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 12 solution in Python."
seo_title: "Advent of Code 2024 Day 12 -- Garden Groups by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 12 solution in Python."
# %s/\v([D|d]ay.{0,1})11/\112/gc
# %s/\vGarden Groups/Garden Groups/g
---

# Day 12: [Garden Groups](https://adventofcode.com/2024/day/12)

## Part 1

Today's puzzle took me a long time to solve, but I finally got it! The input was
a map of garden plots, with each letter on the map representing a garden plot
with only that type of plant growing on it. When multiple garden plots are
growing the same type of plant and are touching (horizontally or vertically),
they form a region. For example:

```
AAAA
BBCD
BBCC
EEEC
```

The above map has `5` regions: `A`, `B`, `C`, `D`, and `E`. For a given map, we
need to calculate the price to fence in every region. The price to fence in a
region is calculated by multiplying the area of a region by the perimeter of the
region. The area of a region is the number of garden plots in the region, and
the perimeter of a region is the number of garden plots on the edge of the
region, i.e., the number of garden plots that are not touching another garden
plot of the same type.

So for the example above, we can visualize the regions as follows, where `|` and
`-` represent the perimeter of the region:

```
+-+-+-+-+
|A A A A|
+-+-+-+-+     +-+
              |D|
+-+-+   +-+   +-+
|B B|   |C|
+   +   + +-+
|B B|   |C C|
+-+-+   +-+ +
          |C|
+-+-+-+   +-+
|E E E|
+-+-+-+
```

Calculating the price of fencing for the map above, we get:

- `A`: `4 * 10 = 40`
- `B`: `4 * 8 = 32`
- `C`: `4 * 10 = 40`
- `D`: `1 * 4 = 4`
- `E`: `3 * 8 = 24`
- Total price: `40 + 32 + 40 + 4 + 24 = 140`

Looking at a slightly more complex example:

<div class="side-by-side">
    <div class="toleft">
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>OOOOO
OXOXO
OOOOO
OXOXO
OOOOO</code></pre></div></div>
    </div>

    <div class="toright">

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>+- - - - -+
|O O O O O|
  +-+ +-+
|O|X|O|X|O|
  +-+ +-+
|O O O O O|
  +-+ +-+
|O|X|O|X|O|
  +-+ +-+
|O O O O O|
+- - - - -+
</code></pre></div></div>
    </div>
<figcaption>Left: Input map. Right: Map with fenced regions.</figcaption>
</div>

There are `5` regions in the map above: `1` region of `O` and `4` regions of
`X`. The perimeter of the `O` region is `20` for the outer edges, plus `4` for
each of the boundaries with the `X` regions. The perimeter of each `X` region is
`4`. Thus, the total price of fencing for the map above is
`756 + 4 + 4 + 4 + 4 = 772`.

So, the task is to calculate the total price of fencing for the given map.

### My Solution

The first part was pretty straightforward to do:

- I read the input and stored it in a 2D list.
- I then iterated over the 2D list and used a depth-first search to find all the
  regions in the map, keeping track of which cells have boundaries at the same
  time.
- For every region, calculate the perimeter by counting the number of boundary
  cells and the area by counting the number of cells in the region.

Reading the input and storing it in a 2D list:

```python
with open("input.txt", "r") as f:
	grid = [list(line.strip()) for line in f]
```

Building all the regions in the map:

```python
def build_regions(
    grid: Grid
) -> tuple[dict[str, list[set[Coordinate]]], list[list[set]]]:
    m, n = len(grid[0]), len(grid)
    regions = DefaultDict(list)
    visited = [[False for _ in range(m)] for _ in range(n)]
    not_neighbors = [[set() for _ in range(m)] for _ in range(n)]
    for i in range(n):
        for j in range(m):
            if visited[i][j]:
                continue
            region = set()
            stack = [(j, i)]
            while stack:
                position = stack.pop()
                region.add(position)
                visited[position[1]][position[0]] = True
                for direction in (
                    Directions.UP,
                    Directions.RIGHT,
                    Directions.DOWN,
                    Directions.LEFT,
                ):
                    try:
                        next_position = get_next_position(grid, position, direction)
                        if not visited[next_position[1]][next_position[0]]:
                            stack.append(next_position)
                    except CoordinateError:
                        not_neighbors[position[1]][position[0]].add(direction)
            regions[grid[i][j]].append(region)
    return regions, not_neighbors
```

This is where most of my logic was. Breaking down the code:

- `m, n = len(grid[0]), len(grid)` Get the dimensions of the grid.
- `regions = DefaultDict(list)` A dictionary to store the regions for each
  plant.
- `visited = [[False for _ in range(m)] for _ in range(n)]` A 2D list to keep
  track of which cells have been visited.
- `not_neighbors = [[set() for _ in range(m)] for _ in range(n)]` A 2D list to
  keep track of which cells have boundaries with other regions.
- `for i in range(n) ... stack.append(position)` Iterate over the grid and use a
  depth-first search to find all the cells that should be in the same region.
- `except CoordinateError` If the next cell is out of bounds, add the direction
  to the `not_neighbors` set for the current cell.
- `regions[grid[i][j]].append(region)` Add the region to the dictionary.
- `return regions, not_neighbors` Return the regions and the `not_neighbors`
  list.

My `get_next_position` function is a simple function that returns the next
position given a direction, if it is valid (I've used this approach for I think
three of the days already):

```python
def get_next_position(
    grid: Grid, position: Coordinate, direction: Directions
) -> Coordinate:
    m, n = len(grid[0]), len(grid)
    value = grid[position[1]][position[0]]
    next_position = None
    match direction:
        case Directions.UP:
            if position[1] == 0:
                raise CoordinateError
        case Directions.LEFT:
            if position[0] == 0:
                raise CoordinateError
        case Directions.DOWN:
            if position[1] == n - 1:
                raise CoordinateError
        case Directions.RIGHT:
            if position[0] == m - 1:
                raise CoordinateError
        case Directions.UP_LEFT:
            if position[1] == 0 or position[0] == 0:
                raise CoordinateError
        case Directions.UP_RIGHT:
            if position[1] == 0 or position[0] == m - 1:
                raise CoordinateError
        case Directions.DOWN_LEFT:
            if position[1] == n - 1 or position[0] == 0:
                raise CoordinateError
        case Directions.DOWN_RIGHT:
            if position[1] == n - 1 or position[0] == m - 1:
                raise CoordinateError
    dx, dy = direction.value
    next_position = (position[0] + dx, position[1] + dy)
    if (
        direction
        in {
            Directions.UP,
            Directions.RIGHT,
            Directions.DOWN,
            Directions.LEFT,
        }
        and value != grid[next_position[1]][next_position[0]]
    ):
        raise CoordinateError
    return next_position
```

It will raise a coordinate error if the next position is out of bounds or if the
next position is not the same value as the current position.

Now that I have all the regions, and in which direction the cells have
boundaries I can calculate the perimeter of each region:

```python
def calculate_perimeter(region: Set[Coordinate], not_neighbors: List[List[Set]]) -> int:
    perimeter = 0
    for coordinate in region:
        perimeter += len(not_neighbors[coordinate[1]][coordinate[0]])
    return perimeter
```

For each cell in the region, I add the number of directions in the
`not_neighbors` set to the perimeter. This works because a fence is needed
whenever a cell has a boundary with a cell of a different value.

Now I can calculate the total price of fencing for the map:

```python
def main1():
    with open("input.txt", "r") as f:
        grid = [list(line.strip()) for line in f]

    regions, not_neighbors = build_regions(grid)
    price = sum(
        calculate_perimeter(r, not_neighbors) * len(r)
        for region in regions
        for r in regions[region]
    )

    print(f"LOG: { price = }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day12/solution.py).

## Part 2

For the second part of the puzzle, the price of fencing is calculated
differently. Instead of multiplying the area of the region by its perimeter, we
need to multiply the area by the _number of sides_ that the region has.

Looking at the small example from part 1:

```
AAAA
BBCD
BBCC
EEEC
```

The `A` region has `4` sides, which the `B`, `D`, and `E` regions also have. The
`C` region however, has `8` sides. So the new price calculations are:

- `A`: `4 * 4 = 16`
- `B`: `4 * 4 = 16`
- `C`: `4 * 8 = 32`
- `D`: `1 * 4 = 4`
- `E`: `3 * 4 = 12`
- Total price: `16 + 16 + 32 + 4 + 12 = 80`

Here's a more complex map:

<div class="side-by-side">
    <div class="toleft">
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>AAAAAA
AAABBA
AAABBA
ABBAAA
ABBAAA
AAAAAA</code></pre></div></div>
    </div>

    <div class="toright">

<!-- deno-fmt-ignore-start -->
<div class="language-plaintext highlighter-rouge">
	<div class="highlight"><pre class="highlight"><code>
+- - - - - -+
|A A A A A A|
      +- -+
|A A A|B B|A|

|A A A|B B|A|
  +- -+- -+
|A|B B|A A A|

|A|B B|A A A|
  +- -+
|A A A A A A|
+- - - - - -+
</code></pre></div></div>
<!-- deno-fmt-ignore-end -->
</div>

<figcaption>Left: Input map. Right: Map with fenced regions.</figcaption>
</div>

The single `A` region has `4` sides on the outside and `8` sides on the inside,
while both `B` regions have `4` sides each. Even though the fence through the
middle of the map looks like it should count as `1` line, it actually counts as
`2` lines since where both the `B` regions touch diagonally, they are _not
actually connected_. So the total price of fencing for the map above is `368`.

Our task is to calculate the total price of fencing for the given map using the
number of sides as the multiplier.

### My Solution

A couple things to note:

- The number of sides a region has is the same as the number of corners it has.
- For a cell to be a corner, it must have a boundary in two adjacent directions.
- A cell can be an inside corner _and_ an outside corner. In the below map, the
  upper left `A` cell is both an inside and outside corner for the `A` region.

```
CCC
CAA
CAB
```

My solution for part 2 was done in a very inelegant way, but hey it works! In
the process of solving part 1, I had already made functions to visualize the
regions and the boundaries of the regions, including the corners. All the
visualizations shown in this post besides the first one were generated by my
code, so I know it works. I just needed to count the corners for each region
now.

So let's go over those functions:

I first have a function to create a map with adjacent cells separated by a space
in every direction. The adjacent spaces will be filled in by the fences as
needed later on.

```python
def make_adjacent_grid(grid: Grid) -> Grid:
    grid_with_adjacent_spaces = [[" " for _ in range(2 * len(grid[0]) + 1)]]
    for row in grid:
        grid_with_adjacent_spaces.append(
            list(chain.from_iterable([[" ", cell] for cell in row])) + [" "]
        )
        grid_with_adjacent_spaces.append([" " for _ in range(2 * len(grid[0]) + 1)])
    return grid_with_adjacent_spaces
```

I then have a function that marks the perimeter of a region with `|` and `-`:

```python
def mark_perimeter(
    grid_with_adjacent_spaces: Grid, region: Set[Coordinate], grid: Grid
) -> None:
    for x, y in region:
        # Calculate corresponding coordinates in grid_with_adjacent_spaces
        adj_x, adj_y = 2 * x + 1, 2 * y + 1
        # Check all four directions
        for direction, marker in [
            (Directions.LEFT, "|"),
            (Directions.RIGHT, "|"),
            (Directions.UP, "-"),
            (Directions.DOWN, "-"),
        ]:
            dx, dy = direction.value
            nx, ny = x + dx, y + dy
            adj_nx, adj_ny = adj_x + dx, adj_y + dy
            # Check if the neighboring cell is out of bounds or not part of the region
            if (
                nx < 0
                or nx >= len(grid[0])
                or ny < 0
                or ny >= len(grid)
                or (nx, ny) not in region
            ):
                grid_with_adjacent_spaces[adj_ny][adj_nx] = marker
```

This function determines if a cell is on the perimeter of a region by checking
if the neighboring cell should be out of bounds or not part of the region. If it
is, it marks the adjacent space with a `|` or `-`.

I then have a function that marks the corners with `+`:

```python
def add_corners(grid_with_adjacent_spaces: Grid) -> None:
    rows = len(grid_with_adjacent_spaces)
    cols = len(grid_with_adjacent_spaces[0])

    for y in range(rows):
        for x in range(cols):
            # Check if this position should be a corner
            if grid_with_adjacent_spaces[y][x] == " ":
                if (y > 0 and grid_with_adjacent_spaces[y - 1][x] == "|") and (
                    x > 0 and grid_with_adjacent_spaces[y][x - 1] == "-"
                ):
                    grid_with_adjacent_spaces[y][x] = "+"
                elif (y > 0 and grid_with_adjacent_spaces[y - 1][x] == "|") and (
                    x < cols - 1 and grid_with_adjacent_spaces[y][x + 1] == "-"
                ):
                    grid_with_adjacent_spaces[y][x] = "+"
                elif (y < rows - 1 and grid_with_adjacent_spaces[y + 1][x] == "|") and (
                    x > 0 and grid_with_adjacent_spaces[y][x - 1] == "-"
                ):
                    grid_with_adjacent_spaces[y][x] = "+"
                elif (y < rows - 1 and grid_with_adjacent_spaces[y + 1][x] == "|") and (
                    x < cols - 1 and grid_with_adjacent_spaces[y][x + 1] == "-"
                ):
                    grid_with_adjacent_spaces[y][x] = "+"
```

This function iterates over the entire grid with adjacent spaces and adds a `+`
to the cell if there are fences in two adjacent directions.

Now that I have all the functions to visualize the regions and the corners, I
use them to build a fresh grid with adjacent spaces for each region and count
the corners that appear in each region:

```python
def main2():
    with open("input.txt", "r") as f:
        grid = [list(line.strip()) for line in f]

    regions, not_neighbors = build_regions(grid)

    price = 0
    for region in regions:
        for r in regions[region]:
            g_a_s = make_adjacent_grid(grid)
            mark_perimeter(g_a_s, r, grid)
            add_corners(g_a_s)
            price += calculate_corners(r, g_a_s, not_neighbors) * len(r)
    print(f"LOG: { price = }")
```

My `calculate_corners` function is as follows:

```python
def calculate_corners(
    region: Set[Coordinate], grid_with_adjacent_spaces: Grid, not_neighbors
) -> int:
    corners = set()
    for coordinate in region:
        x, y = coordinate
        adj_x, adj_y = 2 * x + 1, 2 * y + 1
        for direction in (
            Directions.UP_RIGHT,
            Directions.UP_LEFT,
            Directions.DOWN_RIGHT,
            Directions.DOWN_LEFT,
        ):
            try:
                next_position = get_next_position(
                    grid_with_adjacent_spaces, (adj_x, adj_y), direction
                )
                if grid_with_adjacent_spaces[next_position[1]][
                    next_position[0]
                ] == "+" and is_corner_of_region(coordinate, direction, not_neighbors):
                    corners.add((coordinate, next_position))
            except CoordinateError:
                pass
    return len(corners)
```

It iterates over all the cells in the region and checks if there is a corner in
any of the four corners of the cell. If there is, it validates if the corner is
a corner of the current region and adds it to the set of corners. By adding a
tuple of the coordinate and the corner coordinate, I can ensure that the corner
can be counted as both an inside and outside corner (Look at the `AB` figure-8
example to see why this matters).

My `is_corner_of_region` function is as follows:

```python
def is_corner_of_region(c: Coordinate, direction: Directions, not_neighbors) -> bool:
    try:
        match direction:
            case Directions.UP_RIGHT:
                return (
                    Directions.UP in not_neighbors[c[1]][c[0]]
                    and Directions.RIGHT in not_neighbors[c[1]][c[0]]
                ) or (
                    Directions.DOWN in not_neighbors[c[1] - 1][c[0] + 1]
                    and Directions.LEFT in not_neighbors[c[1] - 1][c[0] + 1]
                )
            case Directions.UP_LEFT:
                return (
                    Directions.UP in not_neighbors[c[1]][c[0]]
                    and Directions.LEFT in not_neighbors[c[1]][c[0]]
                ) or (
                    Directions.DOWN in not_neighbors[c[1] - 1][c[0] - 1]
                    and Directions.RIGHT in not_neighbors[c[1] - 1][c[0] - 1]
                )
            case Directions.DOWN_RIGHT:
                return (
                    Directions.DOWN in not_neighbors[c[1]][c[0]]
                    and Directions.RIGHT in not_neighbors[c[1]][c[0]]
                ) or (
                    Directions.UP in not_neighbors[c[1] + 1][c[0] + 1]
                    and Directions.LEFT in not_neighbors[c[1] + 1][c[0] + 1]
                )

            case Directions.DOWN_LEFT:
                return (
                    Directions.DOWN in not_neighbors[c[1]][c[0]]
                    and Directions.LEFT in not_neighbors[c[1]][c[0]]
                ) or (
                    Directions.UP in not_neighbors[c[1] + 1][c[0] - 1]
                    and Directions.RIGHT in not_neighbors[c[1] + 1][c[0] - 1]
                )
    except IndexError:
        return True
    return False
```

If the corner is a corner of the region, it returns `True`, otherwise `False`. I
have two cases for each corner, one for when the corner is an inside corner and
one for when it is an outside corner.

Looking at the bottom-left `A` cell in the map below, it is an inside corner for
the `A` region and an outside corner for the `B` region. I can see that there is
a `+` in the `UP_RIGHT` direction from the cell, but the `A` cell's neighbors
are both `A` as well, so the first case will return `False`. However, for the
cell on the opposite side of the `+`, it has boundaries in the `DOWN` and `LEFT`
directions, so the second case will return `True`, and the corner is then
correctly counted as a corner for the `A` region.

```
A|B
A+-
AAA
```

Using these functions, I was able to calculate the total price of fencing for
the map. This took me a while to figure out, and I'm sure there are far more
elegant solutions out there, but I'm happy with what I came up with on my own.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day12/solution.py)
for the full solution.

---

That's it for day 12 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
