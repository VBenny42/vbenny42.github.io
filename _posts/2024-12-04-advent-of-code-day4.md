---
title: "Advent of Code 2024 Day 4 – Ceres Search"
layout: post
date: 2024-12-04 20:29
image: /assets/images/favicon/apple-touch-icon.png
headerImage: false
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 4 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 4 solution in Python."
seo_title: "Advent of Code 2024 Day 4 -- Ceres Search by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 4 solution in Python."
---

# Day 4: [Ceres Search](https://adventofcode.com/2024/day/4)

## Part 1

Today's puzzle is called "Ceres Search".
The input for today's puzzle is a file where it represents a word search where we have to find the word `XMAS`.
This is the example input that was given.

```plaintext
MMMSXXMASM
MSAMXMSMSA
AMXSXMAAMM
MSAMASMSMX
XMASAMXAMM
XXAMMXXAMA
SMSMSASXSS
SAXAMASAAA
MAMMMXMMMM
MXMXAXMASX
```

`XMAS` can appear horizontal, vertical, diagonal, written backwards, or even overlapping with other words.
From the example, there are 18 instances of `XMAS`.
Here's the example where all the letters not involved with `XMAS` words have been replaced with `.`.

```plaintext
....XXMAS.
.SAMXMS...
...S..A...
..A.A.MS.X
XMASAMX.MM
X.....XA.A
S.S.S.S.SS
.A.A.A.A.A
..M.M.M.MM
.X.X.XMASX
```

Our task is to find the number of instances of `XMAS` in the word search.

### My Solution

For my solution, I went a bit overboard with the code needed, so there are a few functions that I will be explaining.
But first let's start with the pseudocode to solve the problem:

1. Read the input file and store the word search in a 2D list. We will be treating this list as a grid.
2. For every coordinate in the grid, check if the word `XMAS` can be formed in any direction (horizontal, vertical, diagonal, and reverse diagonal), starting from that coordinate.
3. If the word `XMAS` can be formed, add the amount of times it can be formed to a counter.
4. Return the counter as the result.

Starting with step 1, let's read the input file and store the word search in a 2D list.

```python
with open("input.txt") as f:
    lines = f.readlines()
grid = [list(line.strip()) for line in lines]
```

Easy enough, on to step 2.
For every coordinate in the grid, we need to check if the word `XMAS` can be formed in any direction.
To do this, I created a function called `get_adjacent_letters` that returns the letters adjacent to a given coordinate in the grid.

```python
def get_adjacent_letters(
    coord: Coordinate, grid: list[list[str]]
) -> dict[Directions, tuple[str, Coordinate]]:
    x, y = coord.x, coord.y
    adjacent = {}
    if x > 0:
        c = Coordinate()
        c.x = x - 1
        c.y = y
        adjacent[Directions.LEFT] = (grid[y][x - 1], c)
    if x < len(grid[y]) - 1:
        c = Coordinate()
        c.x = x + 1
        c.y = y
        adjacent[Directions.RIGHT] = (grid[y][x + 1], c)
    if y > 0:
        c = Coordinate()
        c.x = x
        c.y = y - 1
        adjacent[Directions.UP] = (grid[y - 1][x], c)
    if y < len(grid) - 1:
        c = Coordinate()
        c.x = x
        c.y = y + 1
        adjacent[Directions.DOWN] = (grid[y + 1][x], c)
    if x > 0 and y > 0:
        c = Coordinate()
        c.x = x - 1
        c.y = y - 1
        adjacent[Directions.UP_LEFT] = (grid[y - 1][x - 1], c)
    if x < len(grid[y]) - 1 and y > 0:
        c = Coordinate()
        c.x = x + 1
        c.y = y - 1
        adjacent[Directions.UP_RIGHT] = (grid[y - 1][x + 1], c)
    if x > 0 and y < len(grid) - 1:
        c = Coordinate()
        c.x = x - 1
        c.y = y + 1
        adjacent[Directions.DOWN_LEFT] = (grid[y + 1][x - 1], c)
    if x < len(grid[y]) - 1 and y < len(grid) - 1:
        c = Coordinate()
        c.x = x + 1
        c.y = y + 1
        adjacent[Directions.DOWN_RIGHT] = (grid[y + 1][x + 1], c)
    return adjacent
```

This function takes a coordinate and the grid as input and returns a dictionary
where the keys are the directions relative to the coordinate and the values are
tuples containing the letter at the adjacent coordinate as well as the actual
coordinate.
If the coordinate is at the edge of the grid, the function will not return the
adjacent coordinate that would be out of bounds.
This function seems like overkill, and it is, but I think it ends up being more
readable than if I just hardcoded the directions.

Now that I can get the adjacent letters, I created a function called `is_xmas_match` that checks if the word `XMAS` can be formed starting from a given coordinate in a given direction.

```python
def is_xmas_match(
    grid: list[list[str]],
    coord: Coordinate,
    current_match: list[Coordinate],
    current_direction: Directions,
) -> list[Coordinate]:
    xmas = "XMAS"

    adjacent = get_adjacent_letters(coord, grid)
    adjacent_letter = adjacent.get(current_direction, None)
    if adjacent_letter is None:
        return []

    potential_xmas = (
        "".join(grid[c.y][c.x] for c in current_match)
        + grid[coord.y][coord.x]
        + adjacent_letter[0]
    )

    if xmas.startswith(potential_xmas):
        if len(potential_xmas) == len(xmas):
            return current_match + [coord, adjacent_letter[1]]
        return is_xmas_match(
            grid,
            adjacent_letter[1],
            current_match + [coord],
            current_direction,
        )
    return []
```

This function takes the grid, a coordinate, the current match, and the current direction as input.
Let's break down the function:

-   `xmas = "XMAS"` The word we are looking for.
-   `adjacent = get_adjacent_letters(coord, grid)` Get the adjacent letters to the current coordinate.
-   `adjacent_letter = adjacent.get(current_direction, None)` Get the letter in
    the direction we are currently checking.
    If the letter is `None`, this means we are at the edge of the grid and cannot
    form the word `XMAS`, so return empty list.
-   `potential_xmas = "".join(grid[c.y][c.x] for c in current_match) + grid[coord.y][coord.x] + adjacent_letter[0]`
    This creates a string of the letters we have matched so far, the letter at
    the current coordinate, and the adjacent letter in the direction we are
    checking.
-   `if xmas.startswith(potential_xmas):` Check if the word we have formed so far
    is a prefix of the word `XMAS`.
    If it is, add the current coordinate to the match so far and continue
    checking the next letter in the direction we are checking.
-   `return []` If the word we have formed so far is not a prefix of `XMAS`, return an empty list.

Again, I know there's a lot of overhead in this function, sorry!

Now that we can check if the word `XMAS` can be formed starting from a given
coordinate in a given direction, we can use it to go through every coordinate
in the grid in every direction and keep track of the matches.

```python
def xmas_matches1(grid: list[list[str]]) -> int:
    matches = 0

    for y in range(len(grid)):
        for x in range(len(grid[y])):
            for direction in Directions:
                c = Coordinate()
                c.x = x
                c.y = y
                match = is_xmas_match(grid, c, [], direction)
                if match:
                    matches += 1

    return matches
```

Breaking down the function:

-   `matches = 0` Initialize a counter for the number of matches.
-   `for y in range(len(grid)):` Iterate over the rows of the grid.
-   `for x in range(len(grid[y])):` Iterate over the columns of the grid.
-   `for direction in Directions:` Iterate over the directions.
-   `c = Coordinate()` Create a coordinate object for the current position.
-   `match = is_xmas_match(grid, c, [], direction)` Check if the word `XMAS` can be formed starting from the current coordinate in the current direction.
-   `if match:` If a match is found, increment the counter.

Finally, we can call the `xmas_matches1` function with the grid we read from the input file to get the number of instances of `XMAS`.

```python
print(xmas_matches1(grid))
```

I've only included the relevant parts of the code here, but to see my full
solution, you can check out my [Advent of Code GitHub
repository](https://github.com/VBenny42/AoC/blob/main/2024/day04/solution.py).

## Part 2

For part 2, instead of finding the word `XMAS`, we have to find instances of two `MAS` words in the shape of an `X`.
For example,

```plaintext
M.S
.A.
M.S
```

Looking at the example input again with the irrelevant letters replaced with `.`.

```plaintext
.M.S......
..A..MSMS.
.M.S.MAA..
..A.ASMSM.
.M.S.M....
..........
S.S.S.S.S.
.A.A.A.A..
M.M.M.M.M.
..........
```

There are 9 instances of `X-MAS` appearing in the word search.
Notice that the instances can overlap, and the `X` can be formed in any direction.

### My Solution

To solve part 2, my solution is less complex than part 1.
Again, let's start with the pseudocode:

1.  Read the input file and store the word search in a 2D list. It's treated as a grid.
2.  An `X-MAS` instance can only be formed with an `A` in the middle, so
    iterate over all the coordinates in the grid, and if the letter is `A`, check
    if an `X-MAS` instance can be formed starting from that coordinate.
3.  Keep track of the number of instances of `X-MAS` found.

The file is read in the same way as part 1.

To check if the `A` at a coordinate can be part of an `X-MAS` instance, I created a function called `is_x_mas_match`.

```python
def is_x_mas_match(grid: list[list[str]], coord: Coordinate) -> bool:
    corners = {
        Directions.UP_LEFT,
        Directions.UP_RIGHT,
        Directions.DOWN_LEFT,
        Directions.DOWN_RIGHT,
    }
    adjacent = get_adjacent_letters(coord, grid)
    if (corners).intersection(adjacent.keys()) != corners:
        return False
    return (
        {adjacent[Directions.UP_LEFT][0], adjacent[Directions.DOWN_RIGHT][0]}
        == {"M", "S"}
    ) and (
        {adjacent[Directions.UP_RIGHT][0], adjacent[Directions.DOWN_LEFT][0]}
        == {"M", "S"}
    )
```

Breaking down the function:

-   `corners = {...}` The set of directions that are the corners of the `X` in an `X-MAS` instance. We only need to check these directions.
-   `adjacent = get_adjacent_letters(coord, grid)` Get the adjacent letters to the current coordinate (See! I told you it would be useful!).
-   `if (corners).intersection(adjacent.keys()) != corners:` Check if all the corners are present in the adjacent letters.
    If not, it cannot be an `X-MAS` instance, so return `False`.
-   `return (...) and (...)` For an `X-MAS` match, There must be an `M` and an `S` in _each_ of the diagonals.

Now that we can check if the `A` at a coordinate can be part of an `X-MAS`
instance, we can iterate over all the coordinates in the grid and keep track of
the number of instances of `X-MAS` found.

```python
def xmas_matches2(grid: list[list[str]]) -> int:
    matches = 0
    for y in range(len(grid)):
        for x in range(len(grid[y])):
            if grid[y][x] == "A":
                c = Coordinate()
                c.x = x
                c.y = y
                match = is_x_mas_match(grid, c)
                if match:
                    matches += 1
    return matches
```

The only difference between this function and the one for part 1 is that we only do more checks if the letter at the current coordinate is `A`.
Finally, we can call the `xmas_matches2` function with the grid we read from the input file to get the number of instances of `X-MAS`.

```python
print(xmas_matches2(grid))
```

---

That's it for day 4 of Advent of Code 2024! I hope you enjoyed reading my solution and let's see how the rest of the month goes!
