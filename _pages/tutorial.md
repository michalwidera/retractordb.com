---
layout: single
permalink: /tutorial/
title: "Tutorial"
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/IMG_0758.JPG
  caption: "Photo credit: Tomasz Widera"
excerpt: "RetractorDB is the open source time series database"
toc: true
toc_sticky: false
---
contents of this page is _work in progress_.
Page under construction.

Jump Start
==========
This is first and fast use case. Use tmux for switching tasks. There is a sequence of commands under Linux shell  that compiles example query, starts query processor and shows results.

```
sudo apt-get -y install gcc cmake libboost-all-dev make build-essential tmux
git clone https://github.com/michalwidera/retractordb.git
cd retractordb
cmake CMakeLists.txt; make
./build/xcompiler -q test/Regression/Data/query-lnx.txt
tmux
./build/xretractor
[ctrl+B %]
./build/xqry -d
./build/xqry -s str1
[ctrl+B "]
./build/xqry -s str2
```

Video Tutorials
===============

_Please note that on videos you will hear previous database name. Due trademark conflict this name has been changed to RetractorDB. New videos will be uploaded soon._

Here you can find some video tutorials about RetractorDB Time Series Database system. I suggest to start from Session Record. Math basis are hard to understand for start but at the end everyone who intends to use that system should touch this. Apologies for low quality of these tutorials but this is only a side job for me. I did my best to spread that idea. Hope it will be enough.

Session Record
--------------

This is record of my session with computer. This takes about 18 minutes of presentation how actually system works and how I handle with it. This presentation should go at first. Because there is no math jargon and we can see real data flowing and processing. I'm strongly suggest to start from this point. Note: you require some additional programs installed like: graphviz, feh, tmux. Use apt get install to get them.

{% include video id="I-nd5m7KSK4" provider="youtube" %}

Note: RetractorDB is current name of this database. This video will be recorded again.

Query Language
--------------

This presentation covers area of basic implemented query language. This is mandatory to see if we want to start coding and using this system.

{% include video id="opVlhm-K5Mw" provider="youtube" %}

Note: RetractorDB is current name of this database. This video will be recorded again.

Operator Mechanics
------------------

Operator mechanics is presentation that try to show how different operators from introduced algebra works without mathematical jargon. Is it still quite hard to understand but I did my best. Hope this effort will be helpful.

{% include video id="fRW0sg9EgVM" provider="youtube" %}

Note: RetractorDB is current name of this database. This video will be recorded again.

Math Basics
-----------

This presentation show math basics required to understand core behavior of algebraic operators used in this system. This is quite hard to adapt. I suggest to skip this and return to this data at the end of course.

{% include video id="McT_HoBElCM" provider="youtube" %}

Note: RetractorDB is current name of this database. This video will be recorded again.

Math-and-Model
==============

Following code initiate data.
Code is compatible with python3

```python
#!/usr/bin/python

"""Time Series Algebra Equations Implementations - Python 3.x
   2019 Michal Widera
"""

from fractions import Fraction
from math import floor, ceil
A = range(1, 24)
deltaA = Fraction(1)
B = list(map(chr, range(ord('a'), ord('z')+1)))
deltaB = Fraction(1, 2)
```

Summary operator
----------------

Here is formal mathematical representation of summary operation.

$$
c_{n}=\left\{
\begin{array}{cc}
a_{n}|b_{ \left\lfloor  \frac{n\Delta _{a}}{\Delta _{b}} \right\rfloor }  & \Delta
_{c}=\Delta _{a} \\
a_{ \left\lfloor  \frac {n\Delta _{b}}{\Delta _{a}} \right\rfloor }|b_{n} & \Delta
_{c}=\Delta _{b}
\end{array}
\right. ,\Delta _{c}=\min \left( \Delta _{a},\Delta _{b}\right)
$$

```python
def sum(A: list, deltaA: Fraction, B: list, deltaB: Fraction):

  result = []
  deltaC = min(deltaA, deltaB)

  for i in range(0, 20):
      if deltaC == deltaA:
          result.append(str(A[i])+B[int(i*deltaA/deltaB)]),
      else:
          result.append(str(A[int(i*deltaB/deltaA)])+B[i]),
  return result, deltaC
```

Example query with summary operator in RetractorDB Query Language:

```
SELECT *  STREAM Result FROM A + B
```

Difference operation
---------------------

Here is formal mathematical representation of difference operation.
Formal models precede physical implementation in python
The implementation of the operators of the introduced algebra is based on these equations.

$$
a_{n}=\left\{
\begin{array}{cc}
c_{n} & \Delta _{b}\geqslant \Delta _{a} \\
c_{\left\lceil \frac{n\Delta _{a}}{\Delta _{b}}\right\rceil } & \Delta
_{b}<\Delta _{a}
\end{array}
\right.
$$

```python
def diff(C: list, deltaA: Fraction, deltaB: Fraction):

  result = []
  deltaC = min(deltaA, deltaB)

  for i in range(0, 10):
      if deltaA > deltaB:
          result.append(C[int(ceil(i*deltaA/deltaB))])
      else:
          result.append(C[i])
  return result, deltaC
```

Example query whith difference operator in RetractorDB Query Language:

```
SELECT * STREAM Result FROM A - 0.5
```

Interlace operation
-------------------

$$
c_{n}=\left\{
\begin{array}{cc}
b_{n-\left\lfloor n z \right\rfloor } & \left\lfloor n z
\right\rfloor =\left\lfloor \left( n+1\right) z \right\rfloor \\
a_{\left\lfloor n z \right\rfloor } & \left\lfloor n z \right\rfloor
\neq \left\lfloor \left( n+1\right) z \right\rfloor%
\end{array}%
\right.
$$

$$
z =\frac{\Delta _{b}}{\Delta _{a}+\Delta _{b}},\Delta _{c}=%
\frac{\Delta _{a}\Delta _{b}}{\Delta _{a}+\Delta _{b}}
$$

```python
def hash(A: list, deltaA: Fraction, B: list, deltaB: Fraction):

  result = []
  delta = deltaB / (deltaA + deltaB)

  for i in range(0, 20):
      if floor(i*delta) == floor((i+1)*delta):
          result.append(B[i-int(floor((i+1)*delta))])
      else:
          result.append(A[int(floor(i*delta))])

  deltaC = (deltaA*deltaB)/(deltaA+deltaB)
  return result, deltaC
```

Example query with Hash (Interlace) operator in RetractorDB Query Language:

```
SELECT *  STREAM Result FROM A # B
```


Deinterlace
-----------

Main series:

$$
a_{n} = c_{n+ \left\lceil  \frac{(n+1)\Delta _{a}}{\Delta _{b}} \right\rceil }
$$

$$
\Delta _{a}=\frac{\Delta _{c}\Delta _{b}}{\left\vert \Delta _{c}-\Delta _{b}\right\vert }
$$

```python
def dehasheven(C: list, deltaC: Fraction, deltaA: Fraction):

  result = []
  deltaB = deltaA*deltaC / (deltaA - deltaC)

  for i in range(0, 6):
      result.append(C[i+int(ceil((i+1)*deltaA/deltaB))])
  return result, deltaB
```

Example query with Deinterlace (Even) in RetractorDB Query Language:

```
SELECT *  STREAM Result FROM A % 0.5
```

Resiude:

$$
b_{n} = c_{n+\left\lfloor \frac{n\Delta _{b}}{\Delta _{a}}\right\rfloor}
$$

$$
\Delta _{b}=\frac{\Delta _{c}\Delta _{a}}{\left\vert \Delta _{c}-\Delta_{a}\right\vert }
$$

```python
def dehashodd(C: list, deltaC: Fraction, deltaB: Fraction):

  result = []
  deltaA = deltaB*deltaC / (deltaB - deltaC)

  for i in range(0, 6):
      result.append(C[i+int(i*deltaB/deltaA)])
  return result, deltaA
```

Example query with Deinterlace (Odd) in RetractorDB Query Language:

```
SELECT *  STREAM Result FROM A & 0.5
```

Results
-------

Code that call above functions:

```python
def main():
    hash_result, delta_hash = hash(A, deltaA, B, deltaB)
    sum_result, delta_sum = sum(A, deltaA, B, deltaB)
    print("Sum:", sum(A, deltaA, B, deltaB))
    print("Hash:", hash(A, deltaA, B, deltaB))
    print("Diff:", diff(sum_result, deltaA, deltaB))
    print("dehasheven:", dehasheven(hash_result, delta_hash, deltaA))
    print("dehashodd:", dehashodd(hash_result, delta_hash, deltaB))

if __name__ == '__main__':
    main()
```

Result:

```
Sum: (['1a', '1b', '2c', '2d', '3e', '3f', '4g', '4h', '5i', '5j', '6k', '6l', '7m', '7n', '8o', '8p', '9q', '9r', '10s', '10t'], Fraction(1, 2))
Hash: (['a', 'b', 1, 'c', 'd', 2, 'e', 'f', 3, 'g', 'h', 4, 'i', 'j', 5, 'k', 'l', 6, 'm', 'n'], Fraction(1, 3))
Diff: (['1a', '2c', '3e', '4g', '5i', '6k', '7m', '8o', '9q', '10s'], Fraction(1, 2))
dehasheven: ([1, 2, 3, 4, 5, 6], Fraction(1, 2))
dehashodd: (['a', 'b', 'c', 'd', 'e', 'f'], Fraction(1, 1))
```

Papers
======

M.Widera. Deterministic method of data sequence processing. Annales Universitatis Mariae Curie-Skłodowska, Sectio AI, Informatica, Vol. 4 (2006), pages 314-331  [Link][det-method]

A. S. Fraenkel. The bracket function and complementary sets of integers. Canad. J.Math, 21:6–27, 1969. [Link][fraenkel]

S. Beatty. Problem 3173. Amer. Math. Monthly, 33:159, 1926.

[det-method]: https://journals.umcs.pl/ai/article/view/3066/2262
[fraenkel]: https://www.cambridge.org/core/journals/canadian-journal-of-mathematics/article/bracket-function-and-complementary-sets-of-integers/923DC20720CE446932EDBED7F348DF65
