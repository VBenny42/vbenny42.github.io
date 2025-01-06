---
title: "Advent of Code 2024 - Golang Time"
layout: post
date: 2025-01-05 18:01
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - go
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Porting Vinesh Benny's Advent of Code 2024 solutions to Golang.

excerpt: "Porting Vinesh Benny's Advent of Code 2024 solutions to Golang."
seo_title: "Advent of Code 2024 - Golang Time by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 solutions in Golang."
---

Once Christmas arrived, I had all my 50 stars for Advent of Code 2024! I had a
lot of fun solving the puzzles and I learned a lot of new things. After
Christmas, people started posting their overall execution times for all the
puzzles in different languages. Someone posted their times for all the puzzles
in Golang and I was surprised to see that they got all running in less than 1
second! When I saw that, I timed my python solutions, and in general my
solutions altogether were at about 16 seconds.

![Python time](/assets/images/2025-01-05/python_time.png)

I recently read
[Go Programming in Easy Steps](https://ineasysteps.com/products-page/go-programming-in-easy-steps/),
and learnt the basic syntax and most if not all of the features that Go has
built in. I thought it would be a good idea to port my solutions to Go and get
some practice with the language. Along the way, I learned a lot of the Go
idiosyncrasies and how to use the standard library to solve problems. A lot of
the content I've read online about Go talks about how there are very simple
constructs in Go that have no higher level abstractions, and that allowed
efficient code to be written. I found that to be true in my experience as well.

# Main takeaways from porting Python to Go

- Go doesn't have list comprehensions or any other comprehensions for that
  matter. This wasn't a big deal, but it did make the code a bit more verbose.
  Also forced me to think about sizes of slices and maps when creating them.
- Go doesn't have a built-in `set` data structure. I had to use a map with
  `struct{}` as the value to simulate a set. Again, not an issue, just more LOC.
- Goroutines are _really_ easy to use. Most of my code was fast enough without
  needing to use goroutines, but I did use them in a few places where I didn't
  need context from the previous iteration.
- Related to the above, channels are also really easy to use. I used them in
  conjunction with goroutines, letting me not worry about mutexes and locks.
- Go's tooling is really good. Testing can be done without any complex setup,
  and the `go test` command is really easy to use. Profiling is also built in,
  and I used it to determine where I should focus my optimization efforts.
- Handling errors in Go forces you to think about what to do with them. They are
  more of a first-class citizen in Go, and you get to explicitly know where
  errors can occur.
- Go doesn't really do sum types. I had to use empty interfaces to simulate
  them, which was a bit of a pain. I'm sure there are better ways to do this,
  but I didn't find them in my limited time with Go.

# Results

After porting and optimizing my solutions, I was able to get all the puzzles to
run in under a second too!

![Golang time](/assets/images/2025-01-05/go_time.png)

I only used two external libraries: one for generating combinations and one for
finding maximal cliques in a graph. I used `networkx` in Python for the latter
anyway, so I didn't feel bad about using a library for that. Combinations were
built in with `itertools` in Python, so I used a library for that in Go too.
Here are the libraries I used:

- [github.com/Tom-Johnston/mamba/graph](https://pkg.go.dev/github.com/Tom-Johnston/mamba/graph)
- [gonum.org/v1/gonum/stat/combin](https://pkg.go.dev/gonum.org/v1/gonum/stat/combin)

I also missed `functools.cache` from Python. I implemented a simple cache with a
global map in Go, but the cache decorator in Python is much more elegant.

A lot of my code follows the same logic as the Python code, just with more lines
due to the lack of comprehensions. I'm sure there are more optimizations that
can be done, but I'm happy with the results I got.

I've uploaded all my solutions to my
[Advent of Code repo](https://github.com/VBenny42/AoC/tree/main/2024/golang).
Now I'm looking forward to Advent of Code 2025, and maybe I'll try to solve
previous years' puzzles in Go too!

If anyone wants to try out my solutions, feel free to do so. The input files
need to be downloaded from the Advent of Code website and placed in the `inputs`
directory. I have a
[shell script](https://github.com/VBenny42/AoC/blob/main/2024/getDayInput.sh)
can download the inputs on the repo as well, you need to get your session cookie
from the website and put it in the same directory as the script.
