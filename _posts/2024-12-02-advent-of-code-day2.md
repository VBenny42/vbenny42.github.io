---
title: "Advent of Code 2024 Day 2 – Red-Nosed Reports"
layout: post
date: 2024-12-02 19:41
image: /assets/images/favicon/apple-touch-icon.png
headerImage: false
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 2 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 2 solution in Python."
seo_title: "Advent of Code 2024 Day 2 -- Historian Hysteria by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 2 solution in Python."
---

# Day 2: [Red-Nosed Reports](https://adventofcode.com/2024/day/2)

## Part 1

Today's puzzle is called "Red-Nosed Reports". The input for today's puzzle is a
file where each line is a sequence of numbers, and the size of the sequence can
be different for every line in the file. This is the example input that was
given.

```plaintext
7 6 4 2 1
1 2 7 8 9
9 7 6 2 1
1 3 2 4 5
8 6 4 4 1
1 3 6 7 9
```

Each line corresponds to a _report_ with the numbers being the _levels_ for that
report. In the example, every report has `5` levels, but in the actual input,
the number of levels can be different for each report.

We need to figure out the number of reports that are _safe_. For a report to be
considered _safe_:

- The levels in the report must either be _all in increasing order_ or _all in
  decreasing order_.
- Any adjacent pair of levels must only differ by _at least one_ and _at most
  three_.

We can determine which reports from the example are safe/unsafe using the above
rules:

- `7 6 4 2 1`: _Safe_ because all the levels are decreasing by `1` or `2`.
- `1 2 7 8 9`: _Unsafe_ because `2 7` is an increase of `5`.
- `9 7 6 2 1`: _Unsafe_ because `6 2` is a decrease of `4`.
- `1 3 2 4 5`: _Unsafe_ because `1 3` is increasing but `3 2` is decreasing.
- `8 6 4 4 1`: _Unsafe_ because `4 4` is neither an increase or a decrease.
- `1 3 6 7 9`: _Safe_ because the levels are all increasing by `1`, `2`, or `3`.

### My Solution

I solved as follows:

- Iterate through each report from the input.
- Validate each report using the rules mentioned above.
- If a report is safe, increment the count of safe reports.
- Return the count of safe reports.

My code has some more details than yesterday's solution, so there' a bit more to
explain.

```python
with open("input.txt", "rb") as f:
    lines = f.readlines()

converted_lines = ((map(int, line.split())) for line in lines)

sum_of_valid_reports_part_1 = reduce(
    lambda sum, report: sum + (1 if validate_report_1(report) else 0),
    converted_lines,
    0,
)
print(f"LOG: {sum_of_valid_reports_part_1 = }")
```

Breaking down the code:

- `with open("input.txt", "rb") as f`: Open the input file in read mode.
- `lines = f.readlines()` Read all the lines into memory from the file.
- `converted_lines = ((map(int, line.split())) for line in lines)` Convert each
  line to a list of integers.
- `sum_of_valid_reports_part_1 = reduce(` Use the `reduce` function to iterate
  through all the reports and sum the safe reports. The `reduce` function takes
  three arguments:
  - `lambda sum, report: sum + (1 if validate_report_1(report) else 0)`: The
    lambda function that increments the sum by `1` only if the report is valid.
  - `converted_lines` The iterable to iterate through.
  - `0` The initial value of the sum.

Now let's look at the `validate_report_1` function which validates each report.

```python
def validate_report_1(report: Iterable[int]) -> bool:
    direction = ""

    for prev, curr in pairwise(report):
        if direction == "":
            if prev > curr:
                direction = "decreasing"
            else:
                direction = "increasing"
        # Adjacent levels can only differ by at least one and at most three
        if (abs(prev - curr) < 1) or (abs(prev - curr) > 3):
            return False
        if direction == "increasing" and prev > curr:
            return False
        if direction == "decreasing" and prev < curr:
            return False
    return True
```

Breaking down the code:

- `direction = ""` Set the direction of the report to an empty string, which
  will be updated to either "increasing" or "decreasing".
- `for prev, curr in pairwise(report):` Iterate through each pair of levels in
  the report. The `pairwise` function is a function from the `itertools` package
  that yields pairs of adjacent elements from an iterable.
- `if direction == "":` If the direction is not set, determine the direction of
  the report based on the first two levels. It only needs to be determined once,
  as all the levels in the report must be in the same direction to be a safe
  report.
- `if (abs(prev - curr) < 1) or (abs(prev - curr) > 3)` Check if the difference
  between the levels is less than `1` or more than `3`. If it is, the report is
  invalid, and early return `False`.
- `if direction == "increasing" and prev > curr:` If the direction is set to be
  increasing, but the levels are not increasing, then the second rule is not
  being followed. So early return `False`.
- Similarly, if the direction is set to be decreasing, but the levels are not
  decreasing, then early return `False`.
- If `False` has not been returned after all the pairs have been checked, then
  the report must be valid, so return `True`.

Now that we have the `validate_report_1` function, we can use it to validate
each report in the input and count the number of safe reports, which is the
answer to part 1.

I've only included the relevant parts of the code here, but to see my full
solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day02/solution.py).

## Part 2

For part 2, we can slightly relax the rules for a report to be considered safe.

If removing a single level from the report that would make the report _unsafe_
according to the rules of part 1, then the report is still considered _safe_.

Looking at the example input again:

```plaintext
7 6 4 2 1
1 2 7 8 9
9 7 6 2 1
1 3 2 4 5
8 6 4 4 1
1 3 6 7 9
```

Determining which reports are safe/unsafe according to the new rules:

- `7 6 4 2 1`: _Safe_ because report was already safe in part 1.
- `1 2 7 8 9`: _Unsafe_ because regardless of level moved, still considered
  unsafe by part 1 rules.
- `9 7 6 2 1`: _Unsafe_, same as above.
- `1 3 2 4 5`: _Safe_, removing `3` makes the report increasing in a steady
  manner.
- `8 6 4 4 1`: _Safe_, removing `4` makes the report decreasing in a steady
  manner.
- `1 3 6 7 9`: _Safe_ because report was already safe in part 1.

### My Solution

My solution for part 2 is less work than part 1, as we can reuse the
`validate_report_1` function from part 1.

Given that we still need to validate each report as per the rules of part 1, we
reuse the `validate_report_1` function, but for each report, we try removing
each level and check if the report is still safe.

```python
with open("input.txt", "rb") as f:
    lines = f.readlines()

# Need to be able to index the levels for part
# so we must convert the map object to a list
converted_lines = ((list(map(int, line.split())

sum_of_valid_reports_part_2 = reduce(
lambda sum, report: sum + (1 if validate_report_2(report) else 0),
	converted_lines,
	0,
)
print(f"LOG: {sum_of_valid_reports_part_2 = }")
```

The only change in the code from part 1 is that we also convert the `map` object
to a `list` so that we can index the levels in the report, which is required for
the part 2 validator function.

```python
def validate_report_2(report: list[int]) -> bool:
    possible_reports = (report[:i] + report[i + 1 :] for i in range(len(report)))
    # If any of the reports with one level removed is valid,
    # then the original report is considered safe
    return any(
        validate_report_1(possible_report) for possible_report in possible_reports
    )
```

Breaking down the code:

- `possible_reports = (report[:i] + report[i + 1 :] for i in range(len(report)))`:
  Generate all possible reports with one level removed from the original report.
- `return any(`: Check if any of the possible reports are valid according to the
  rules of part 1. `any` is a built-in function that returns `True` if any of
  the elements in the iterable are `True`.
  - `validate_report_1(possible_report) for possible_report in possible_reports`:
    Validate each possible report using the rules of part 1. This generates an
    iterator of `True` or `False` values for each possible report, where `True`
    means the report is valid according to the rules of part 1.
  - If any of the possible reports are valid, then the original report is
    considered safe, and the function returns `True`.

---

That's it for day 2 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
