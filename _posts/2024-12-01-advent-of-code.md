---
title: "Advent of Code 2024 Day 1 – Historian Hysteria"
layout: post
date: 2024-12-01 20:54
image: /assets/images/favicon/apple-touch-icon.png
headerImage: false
tag:
    - python
    - advent-of-code
star: true
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 1 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 1 solution in Python."
seo_title: "Advent of Code 2024 Day 1 -- Historian Hysteria by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 1 solution in Python."
---

# What is Advent of Code?

For those that haven't heard of [Advent of
Code](https://adventofcode.com/2024/about), it's an advent calendar of small
Christmas-themed programming puzzles for a variety of skill sets and skill
levels that can be solved in any programming language you like.
As such, there are 25 puzzles in total, with a new one being unlocked each day in December
leading up to Christmas.
As the days go on, the puzzles get harder and harder,
so it's a great way to practice your problem-solving skills, learn new
programming concepts plus they're a great way to get into the Christmas spirit!

The puzzles normally involve some sort of input data that you have to process
in some way to get the answer.
Each puzzle has two parts, that are normally thematically related, with each
part having its own answer and each earning you a gold star which contributes
to your overall score.

I've done Advent of Code in 2022, but I was only able to get up to day 13 then
the problems were a bit too hard for me to solve.
This year, I'm hoping to solve all 25 problems (fingers crossed :) ).
I'm choosing to solve the problems in Python this year as to me, it's the
language that's easiest to actually start solving problems without boilerplate
initialization code.
Each puzzle has a story that goes along with it, but I won't be explaining the
story, as the
actual authors do a much better job of that than I ever could.
I will, however, be explaining the problem and my solution to it.

---

# Day 1: [Historian Hysteria](https://adventofcode.com/2024/day/1)

## Part 1

Today's puzzle is called "Historian Hysteria".
The input for today's puzzle is a file containing two columns of numbers, each
of them representing a list.
This is the example input that was given.

```plaintext
3   4
4   3
2   5
1   3
3   9
3   3
```

These lists are supposed to be the same, but we can see that they are off by some amount.
Our goal is to find how off they are by calculating how far apart the smallest
numbers in each list are from each other.

This is done with the following steps:

-   Get the smallest number from each list.
-   Subtract the smaller number from the larger number.
-   Add the result to a running total.
-   Repeat for each pair of numbers, until the ends of the lists are reached.
-   Return the running total.

For the example input, the smallest numbers from both lists are `1` and `3`, so
the difference is `3 - 1 = 2`; the running total is now `2`.
After that, the smallest numbers are `2` and `3`, so the difference is `3 - 2 =
1`; the running total is now `3`.
Continuing this process, the final running total is `11`.

This sample input is a bit small, so it's easy to do this by hand, but the actual
input is 1000 lines long in this case, so it's better to solve this programmatically.

### My Solution

First, I split the input file into two files, one for each list.
This was done using the `V-BLOCK` mode in NeoVim to select the columns and then copy them to new files.
I know this is might not be the most efficient way to do this, but this was the easiest way I could think of.

<div>
	<figcaption class="caption">After selecting the column, I save it into input1.txt, and then the other column into input2.txt.</figcaption>
	<script src="https://asciinema.org/a/9sk3YHQ2O9WVlIrqiq6NlDuX6.js" id="asciicast-9sk3YHQ2O9WVlIrqiq6NlDuX6" async="true"></script>
</div>

After I had the two files corresponding to the two lists, each list needed to be sorted, so then the pairs of numbers that I need to compare will each be on the same line.
I did this using the `sort` command in the terminal.

```bash
# Numerically sort the input1.txt file and save it into input1-sorted.txt
sort --numeric-sort input1.txt --output=input1-sorted.txt
sort --numeric-sort input2.txt --output=input2-sorted.txt
```

Now that I have the two lists sorted, I can start comparing the numbers and calculating the running total.

```python
with open("input1-sorted.txt", "rb") as i1:
    i1_lines = i1.readlines()
    i1_lines = map(int, i1_lines)
with open("input2-sorted.txt", "rb") as i2:
    i2_lines = i2.readlines()
    i2_lines = map(int, i2_lines)

    diff_sum = sum(abs(line1 - line2) for line1, line2 in zip(i1_lines, i2_lines))
    print(diff_sum)
```

Let's break down the code:

-   `with open("input1-sorted.txt", "rb") as i1:` Open the first sorted list for reading.
-   `i1_lines = i1.readlines()` Read all the lines into from the file.
-   `i1_lines = map(int, i1_lines)` Convert the each line to an integer.
-   `diff_sum = sum(abs(line1 - line2) for line1, line2 in zip(i1_lines, i2_lines))`
    -   `zip(i1_lines, i2_lines)` This will create a list of tuples, where each tuple contains the corresponding lines from the two files.
    -   `abs(line1 - line2)` Calculate the absolute difference between the two numbers.
    -   `sum(...)` Sum all the differences together.
-   `print(diff_sum)` Print the final result.

I've only included the relevant parts of the code here, but to see my full
solution, you can check out my [Advent of Code GitHub
repository](https://github.com/VBenny42/AoC/blob/main/2024/day01/solution.py).

## Part 2

For part 2, using the same input, we need to calculate a total similarity score
by adding up each number in the left list after multiplying it by the number of
times that number appears in the right list.

This was a but unclear to me at first, so let's go through the example.

```plaintext
3   4
4   3
2   5
1   3
3   9
3   3
```

-   The first number in the left list is `3`, and it appears `3` times in the right list, so the similarity score increases by `3 * 3 = 9`.
-   The second number in the left list is `4`, and it appears `1` time in the right list, so the similarity score increases by `4 * 1 = 4`.
-   The third number in the left list is `2`, and it appears `0` times in the right list, so the similarity score does not increase, since `2 * 0 = 0`.
-   The fourth number in the left list is `1`, and it appears `0` time in the right list, so the similarity score does not increase, since `1 * 0 = 0`.
-   The fifth number in the left list is `3`, and it appears `3` times in the right list, so the similarity score increases by `3 * 3 = 9`.
-   The sixth number in the left list is `3`, and it appears `3` times in the right list, so the similarity score increases by `3 * 3 = 9`.

So for the example input, the total similarity score is `9 + 4 + 0 + 0 + 9 + 9 = 31`.

### My Solution

So we need to keep track of:

-   How many times each number appears in the left list.
-   How many times each number from the left list appears in the right list.

These two things can be done using dictionaries in Python.
There's a special subclass of dictionary in Python called `Counter`
specifically to count the number of occurrences of items in an iterable, which
is perfect for this problem.

So given that we can count the number of occurrences of each number for both lists, we can then calculate the similarity score as follows:
For each unique number that appears in the left list, multiply it by the number
of times it appears in the right list, _as well as_ the number of times it
appears in the left list.
We need to multiply every unique number in the left list by the number of times
it appears in the left list, because we need to account for the fact that the
number of times a number appears in the left list is also a factor in the
similarity score.

```python
from collections import Counter

with open("input1-sorted.txt", "rb") as i1:
    lines = i1.readlines()
    lines = map(int, lines)
    i1_counter = Counter(lines)
with open("input2-sorted.txt", "rb") as i2:
    lines = i2.readlines()
    lines = map(int, lines)
    i2_counter = Counter(lines)

similarity_sum = 0
for key in i1_counter.keys():
    similarity_sum += (key * i1_counter[key]) * i2_counter.get(key, 0)
print(similarity_sum)
```

Let's break down the code:

-   `from collections import Counter` Import the `Counter` class from the `collections` module.
-   `i1_counter = Counter(lines)` Create a `Counter` object from the lines in the first file (,after converting them to integers, similar to the previous part).
-   Similarly for the second file.
-   `for key in i1_counter.keys():` Loop through each unique number in the first list, which would be the keys of the `Counter` object.
-   `similarity_sum += (key * i1_counter[key]) * i2_counter.get(key, 0)` Calculate the similarity score for the current number and add it to the running total.
    -   `(key * i1_counter[key])` Multiply the unique number by the number of times it appears in the first list.
    -   `i2_counter.get(key, 0)` Get the number of times the current number appears in the second list, or `0` if it doesn't appear at all.
-   `print(similarity_sum)` Print the final result.

Again, I've only included the relevant parts of the code here, check out my [repository](https://github.com/VBenny42/AoC/blob/main/2024/day01/solution.py) for the full solution.

---

That's it for day 1 of Advent of Code 2024! I hope you enjoyed reading my solution and let's see how the rest of the month goes!
