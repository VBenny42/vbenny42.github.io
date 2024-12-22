---
title: "Advent of Code 2024 Day 22 – Monkey Market"
layout: post
date: 2024-12-22 12:42
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 22 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 22 solution in Python."
seo_title: "Advent of Code 2024 Day 22 -- Monkey Market by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 22 solution in Python."
---

# Day 22: [Monkey Market](https://adventofcode.com/2024/day/22)

## Part 1 {#Part1}

For today's puzzle, we're at a market where monkeys are buying hiding spots for
their bananas. We have some hiding spots to sell, and we want to maximise the
amount of bananas we can get for selling them. The monkeys' buying patterns are
actually _pseudo-random_, and luckily for us, we have a list of their seed
values (our input).

Each buyer's secret number changes into a new number based on the following
algorithm:

- Calculate the result of multiplying the secret number by `64`. Then, mix this
  result into the secret number. Finally, prune the secret number.
- Calculate the result of dividing the secret number by `32`. Round the result
  down to the nearest integer. Then, mix this result into the secret number.
  Finally, prune the secret number.
- Calculate the result of multiplying the secret number by `2048`. Then, mix
  this result into the secret number. Finally, prune the secret number.

To mix a value into the secret number, calculate the bitwise XOR of the given
value and the secret number. Then, the secret number becomes the result of that
operation. (If the secret number is `42` and you were to mix `15` into the
secret number, the secret number would become `37`.) To prune the secret number,
calculate the value of the secret number modulo `16777216`. Then, the secret
number becomes the result of that operation. (If the secret number is
`100000000` and you were to prune the secret number, the secret number would
become `16113920`.)

After this process is completed, the buyer has the next number in their
sequence, and can choose to continue generating numbers.

If a buyer started off with the seed value `123`, the first ten numbers in their
sequence would be:

```
15887950
16495136
527345
704524
1553684
12683156
11100544
12249484
7753432
5908254
```

In a single day, buyers can generate `2000` new secret numbers. We want to find
the sum of the `2000`th secret number for each buyer.

### My Solution {#Part1Solution}

This is pretty simple. I need to implement the algorithm described above and run
it for each secret number `2000` times.

```python
def find_secret_number(n: int) -> int:
    mod = 16777216
    n = ((n * 64) ^ n) % mod
    n = ((n // 32) ^ n) % mod
    n = ((n * 2048) ^ n) % mod
    return n
```

Running this function `2000` times for each buyer and summing:

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        secret_numbers = map(int, f.read().split())

    sum_secret_numbers = 0
    for secret_number in secret_numbers:
        for _ in range(2000):
            secret_number = find_secret_number(secret_number)
        sum_secret_numbers += secret_number

    print(f"ANSWER1: { sum_secret_numbers }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day22/solution.py).

## Part 2 {#Part2}

It turns out that only the ones digit of the secret number is the price that the
buyers are willing to pay. So if a buyer starts with `123`, their first ten
prices would be:

```
3 (from 123)
0 (from 15887950)
6 (from 16495136)
5 (etc.)
4
4
6
4
4
2
```

We have our own monkey that can talk to the other monkeys to conduct business,
but this monkey only decides to sell when it sees a specific sequence of _four
consecutive changes_ in price, immediately selling at that point.

Again if a buyer starts with `123`, their first ten secret numbers, prices and
changes are:

```
     123: 3 
15887950: 0 (-3)
16495136: 6 (6)
  527345: 5 (-1)
  704524: 4 (-1)
 1553684: 4 (0)
12683156: 6 (2)
11100544: 4 (-2)
12249484: 4 (0)
 7753432: 2 (-2)
```

The first price has no associated change as there's nothing to compare it with.

Within these first ten prices, the highest price to sell would be when the price
is `6`. When the first `6` occurs, there's no way to instruct the monkey to sell
then, as only _two_ consecutive changes have occurred. The second six occurs
after the changes `-1,-1,0,2`, so we can instruct the monkey to sell once it
sees that sequence.

Each buyer only buys one hiding spot, so once the monkey sells to one buyer,
they move on to the next buyer. If the monkey never sees the sequence occurring
for a specific buyer, it will just _never_ sell to that buyer, and move on to
the next one.

Also, we can only give the monkey a _single_ sequence to look out for when
selling to each buyer.

Since we want to sell as many bananas as possible, we need to find the sequence
that causes the monkey to sell the most bananas overall.

If the initial secrets of each buyer is:

```
1
2
3
2024
```

The sequence that gets us the most bananas is `-2,1,-1,3`. Using this sequence,
the monkey makes the following sales:

- For the buyer with an initial secret number of `1`, changes `-2,1,-1,3` first
  occur when the price is `7`.
- For the buyer with initial secret `2`, changes `-2,1,-1,3` first occur when
  the price is `7`.
- For the buyer with initial secret `3`, the change sequence `-2,1,-1,3` _does
  not occur_ in the first `2000` changes.
- For the buyer starting with `2024`, changes `-2,1,-1,3` first occur when the
  price is `9`.

Then, the total bananas we get will be `7 + 7 + 9 = 23`.

Our task is to figure out the most bananas that can be sold overall.

### My Solution {#Part2Solution}

I can first build up the prices and changes for each secret number over the
`2000` generations. Then I can look at the `4`-length sequences of the changes,
and map each sequence to its associated price. Once I have this map for every
sequence for the secret number, I can combine all the maps for every secret
number to get the sequence that has the max price overall.

Building the prices and changes:

```python
def get_prices_and_changes(secret_number: int) -> tuple[list[int], list[int]]:
    last = secret_number
    prices = []
    changes = []
    for _ in range(2000):
        secret_number = find_secret_number(secret_number)
        prices.append(secret_number % 10)
        changes.append(secret_number % 10 - last % 10)
        last = secret_number
    return prices, changes
```

Building the map of sequences to prices:

```python
def get_banana_sequences(
    prices: list[int], changes: list[int]
) -> dict[tuple[int, ...], int]:
    sequences = {}
    for i in range(3, len(changes)):
        seq = tuple(changes[i - 3 : i + 1])
        if sum(seq) > 0 and seq not in sequences:
            sequences[seq] = prices[i]
    return sequences
```

I only add `seq` to `sequences` when the sequence is a net positive. If it's not
a net positive, the associated price at the end can never be a maximum overall.
Also I only add the price for the _first_ time `seq` occurs, as the monkey sells
as soon as it sees the sequence, so anytime the sequence occurs after that
doesn't matter.

Now I can combine the sequences for each buyer's secret number:

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        secret_numbers = map(int, f.read().split())

    banana_sequences = Counter()
    for secret_number in secret_numbers:
        prices, changes = get_prices_and_changes(secret_number)
        sequences = get_banana_sequences(prices, changes)
        banana_sequences.update(sequences)

    max_sequence = max(banana_sequences.values())
    print(f"ANSWER2: { max_sequence }")
```

A `Counter` is used for `banana_sequences`, as everytime it gets updated with
new sequences from a secret number, it will add the sequences values to what is
already in the dictionary. e.g.

```python
banana_sequences = Counter({(1, 2, 3, 4): 4})
banana_sequences.update({(1, 2, 3, 4): 2,  (5, 6, 7, 8): 9})
>>> banana_sequences
Counter({(5, 6, 7, 8): 9, (1, 2, 3, 4): 6})
```

Then, once all the possible sequences have been added, I can just find the max
value, which is what we want.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day22/solution.py)
for the full solution.

---

That's it for day 22 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
