# Math notation in LaTeX

Reference of some basic math notations in LaTeX.

## Inline vs display notation

Inline notation uses only one `$` and display uses two `$$`.

For example, for a fraction: $\dfrac{x}{y}$ vs. $$\dfrac{x}{y}$$

```latex
$\dfrac{x}{y}$

$$\dfrac{x}{y}$$
```

## Exponent notation

- $2^0 = 1$
- $2^1 = 2$
- $2^4 = 2*2*2*2$
- $2^{-4} = \frac1{2^4}$

Translates into markdown:

```latex
- $2^0 = 1$
- $2^1 = 2$
- $2^4 = 2*2*2*2$
- $2^{-4} = \frac1{2^4}$
```

## Summation notation

Mathematical notation for adding a series of numbers.

The summary of a list of values:

$a_1 + a_2 + ... + a_n$ is equal to $\sum\limits_{i=1}^\mathbb{n}a_i$

```latex
$\sum\limits_{i=1}^\mathbb{n}a_i$
```

## Natural exponential and logarithm

Natural exponent and logarithm are two of the most important functions in math and its applications.

$y=log(x)$

$y = exp(x)$ or $y = e^x$

## Matrices

This 3 x 3 matrix:

$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
$$

is written with this notation:

```latex
$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
$$
```

## Miscellaneus notations

### Absolute value

Absolute value is the distance away from zero, regardless of sign.

$ abs(-5) = | -5 |$

```latex
$ abs(-5) = | -5 |$
```

### For all values

For all x in $X$ has a representation of: $\forall x \in X$

```latex
 $\forall x \in X$
```

## Links

- [LaTeX/Mathematics](https://en.wikibooks.org/wiki/LaTeX/Mathematics)
