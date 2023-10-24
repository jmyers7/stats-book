---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.15.0
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

(random-vectors)=
# Random vectors; joint, marginal and conditional distributions; independence

(motivation)=
## Motivation

Suppose that you are studying the local housing market. You decide to track the variable $X$, which is the size of a house in square feet, as well as the variable $Y$, which is the selling price of a house in thousands of (US) dollars. You collect data on $n=2000$ houses (the local market encompasses a _large_ metropolitan area), giving you observed random samples

$$
x_1,x_2,\ldots,x_n \quad \text{and} \quad y_1,y_2,\ldots,y_n.
$$

Thus, for each $i=1,2,\ldots,n$, the pair $(x_i,y_i)$ represents the size and selling price of the $i$-th house. You then plot the empirical distributions of the two datasets, obtaining:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center
:   image:
:       width: 100%

import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import multivariate_normal

# hide annoying warnings for tight_layout()
import warnings
warnings.filterwarnings("ignore")

# set custom style for plots
plt.style.use('../aux-files/custom_style_light.mplstyle')

# make sure this comes last in the imports!
# change the output resolution of figures
import matplotlib as mpl 
mpl.rcParams['figure.dpi'] = 600

# end import section

np.random.seed(42)

# data will be simulated from a multivariate gaussian.
# define the parameters
rho = 0.85
sigma_1 = 200
sigma_2 = 20
mu = np.array([2000, 175])
sigma = np.array([
    [sigma_1 ** 2, rho * sigma_1 * sigma_2],
    [rho * sigma_1 * sigma_2, sigma_2 ** 2]
])

# generate the data
data = multivariate_normal(mean=mu, cov=sigma).rvs(size=2000)
X = np.round(data[:, 0])
Y = np.round(data[:, 1])

# build the plots
_, (ax1, ax2) = plt.subplots(ncols=2, figsize=(10, 4))
sns.histplot(x=X, ax=ax1, kde=True, stat='density')
sns.histplot(x=Y, ax=ax2, kde=True, stat='density')
ax1.set_xlabel(r'$x=$size (ft$^2$)')
ax2.set_xlabel(r'$y=$price (\$1,000)')
ax1.set_ylabel('density')
ax2.set_ylabel('')
plt.tight_layout()
```

These are histograms of the two datasets, with KDEs superimposed (we learned about these in the [previous chapter](theory-to-practice)).

Whatever information we might glean from these histograms, the information is about the two random variables $X$ and $Y$ _in isolation_ from each other. But because we expect that the size of a house and its selling price might be (strongly) related, it might be more informative to study $X$ and $Y$ _in tandem_ with each other. One way to do this is to plot the points $(x_i,y_i)$ in the $xy$-plane, obtaining a visualization called a _scatter plot_:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center
:   image:
:       width: 80%

sns.scatterplot(x=X, y=Y)
plt.xlabel(r'$x=$size (ft$^2$)')
plt.ylabel(r'$y=$price (\$1,000)')
plt.tight_layout()
```

This plot confirms exactly what we expected! Since the pattern of points generally follows a positively-sloped line, we may conclude that as the size $x$ of a house increases, so too does its selling price $y$.

To reiterate:

> We would not have been able to discover this relation between $X$ and $Y$ had we studied these two variables in isolation from each other. This suggests that, given _any_ pair of random variables $X$ and $Y$, we might obtain valuable information if we study them _together_ as a single object.

+++

























## $2$-dimensional random vectors

So, what do you get when you want to combine two random variables $X$ and $Y$ into a single object? Here's the answer:

```{prf:definition}
Let $S$ be a probability space. A _$2$-dimensional random vector_ is a function

$$
\mathbf{X} : S \to \mathbb{R}^2.
$$

Thus, we may write $\mathbf{X}(s) = (X_1(s), X_2(s))$ for each sample point $s\in S$. When we do so, the functions $X_1$ and $X_2$ are ordinary random variables that are called the _components_ of the random vector $\mathbf{X}$.
```

So, a $2$-dimensional random vector is nothing but a pair of random variables. That's it. We will often write a random vector simply as a pair $(X_1,X_2)$, where $X_1$ and $X_2$ are the component random variables. In the example in the previous section, we have the random vector $(X,Y)$ consisting of the size of a house and its selling price.

What's the big deal? Simply putting two random variables together into a random vector doesn't seem like it would lead to anything useful. But do you remember that every random variable $X$ induces a probability distribution on $\mathbb{R}$? (If not, look [here](prob-measure-rv).) As I will now show you, every random vector does the same, but this time the induced probability distribution is on the plane $\mathbb{R}^2$. This is where things get interesting!

```{prf:definition}
Let $(X,Y):S \to \mathbb{R}^2$ be a $2$-dimensional random vector on a probability space $S$ with probability measure $P$. We define the _probability measure_ of $(X,Y)$, denoted $P_{(X,Y)}$ or $P_{XY}$, via the formula

$$
P_{XY}(C) = P \left( \{s\in S \mid (X(s),Y(s))\in C\} \right),
$$ (hard-eqn)

for all events $C\subset \mathbb{R}^2$. The probability measure $P_{XY}$ is also called the _joint distribution_ or the _bivariate distribution_ of $X$ and $Y$.
```

Holy smokes, the equation {eq}`hard-eqn` looks confusing! However, just as we saw back in {numref}`prob-measure-rv` when we were talking about the probability measure $P_X$ induced by a single random variable, there is alternate notation for $P_{XY}$ that may clarify the definition. Indeed, we often write the much more suggestive

$$
P((X,Y)\in C)
$$ (wrong2-eqn)

in place of $P_{XY}(C)$. If $C$ happens to be a set of the form $A\times B$ where $A,B\subset \mathbb{R}$, then we will write

$$
P(X\in A, \ Y\in B) \quad \text{or} \quad P(X\in A \text{ and } Y\in B)
$$ (wrong-eqn)

in place of $P_{XY}(A\times B)$. (The comma is pronounced "and.") The expressions in {eq}`wrong2-eqn` and {eq}`wrong-eqn` are technically abuses of notation, but they are _so_ much more descriptive and intuitive that we feel OK forgiving the abuse.

```{admonition} Problem Prompt
Do problem 1 on the worksheet.
```

Before continuing, it might be worth briefly reviewing the definitions of _discrete_ and _bivariate continuous probability distributions_ (see {numref}`discrete-prob` for the first, and {numref}`bivar-cont-prob` for the second). The first types of distributions were defined in terms of the existence of probability mass functions, while the second were defined in terms of the existence of probability density functions. We use these definitions in:

```{prf:definition}
Let $(X,Y)$ be a $2$-dimensional random vector.

* We shall say $(X,Y)$ is _discrete_, or that $X$ and $Y$ are _jointly discrete_, if the joint probability distribution $P_{XY}$ is discrete. In other words, we require that there exists a _joint probability mass function_ $p(x,y)$ such that

    $$
    P((X,Y)\in C) = \sum_{(x,y)\in C} p(x,y)
    $$

    for all events $C \subset \mathbb{R}^2$.

* We shall say $(X,Y)$ is _continuous_, or that $X$ and $Y$ are _jointly continuous_, if the joint probability distribution $P_{XY}$ is continuous. In other words, we require that there exists a _joint probability density function_ $f(x,y)$ such that

    $$
    P\left( (X,Y)\in C \right) = \iint_C f(x,y) \ \text{d}x \text{d}y
    $$
  
    for all events $C\subset \mathbb{R}^2$.
```

So, the component random variables $X$ and $Y$ of a random vector $(X,Y)$ may be discrete or continuous, and the random vector $(X,Y)$ _itself_ may be discrete or continuous. What are the relations between these properties? Answer:

```{prf:theorem} 
Let $(X,Y)$ be a $2$-dimensional random vector.

1. The random vector $(X,Y)$ is discrete if and only if both $X$ and $Y$ are discrete.

2. If $(X,Y)$ is continuous, then $X$ and $Y$ are both continuous. However, it does _not_ nececessarily follow that if both $X$ and $Y$ are continuous, then so too is $(X,Y)$.
```

Part of this theorem will be proved below in {prf:ref}`marginal-thm`.

```{margin}
This clever counterexample is borrowed from {cite}`DeGrootSchervish2014`.
```
The reason that individual continuity of $X$ and $Y$ does _not_ imply continuity of $(X,Y)$ can be explained by observing that _if_ $(X,Y)$ were continuous, then the probability that $(X,Y)$ takes values on any given line in $\mathbb{R}^2$ is $0$. Indeed, this is because probabilities of bivariate continuous distributions are volumes under density surfaces, and there is _no_ volume above a line in the plane. But if $X$ is continuous and $X=Y$, then the probability that $(X,Y)$ takes values on the line $x=y$ is exactly $1$. Thus, the random vector $(X,Y)$ cannot be continuous!

```{admonition} Problem Prompt
Do problems 2-4 on the worksheet.
```

+++






























## Bivariate distribution functions

We now generalize the cumulative distribution functions from {numref}`dist-func-rv` to $2$-dimensional random vectors.

```{prf:definition}
Let $(X,Y)$ be a $2$-dimensional random vector. The _distribution function_ of $(X,Y)$ is the function $F:\mathbb{R}^2 \to \mathbb{R}$ defined by

$$
F(x,y) = P(X\leq x, Y\leq y).
$$

In particular:

1. If $(X,Y)$ is discrete with probability mass function $p(x,y)$, then

    $$
    F(x,y) = \sum_{x^\star\leq x, \ y^\star \leq y} p(x^\star, y^\star).
    $$

2. If $(X,Y)$ is continuous with probability density function $f(x,y)$, then

    $$
    F(x,y) = \int_{-\infty}^y \int_{-\infty}^x f(x^\star, y^\star) \ \text{d}x^\star \text{d} y^\star.
    $$
```

```{admonition} Problem Prompt
Do problem 5 on the worksheet.
```

+++



































## Marginal distributions

If $(X,Y)$ is a $2$-dimensinal random vector, then we have the joint distribution $P_{XY}$ on the plane $\mathbb{R}^2$, as well as the individual distributions $P_X$ and $P_Y$ induced by $X$ and $Y$ on the real line $\mathbb{R}$. When the random vector $(X,Y)$ is in the mix, the latter two distributions have special names:

```{prf:definition}
Let $(X,Y)$ be a $2$-dimensional random vector. Then the distributions $P_X$ and $P_Y$ are called the _marginal distributions_ of $(X,Y)$.
```

So, just to emphasize:

> Marginal distributions are nothing new---the only thing that is new is the terminology.

It turns out that it is rather easy to obtain the marginal distributions from the joint one. This is the content of:

```{prf:theorem}
:label: marg-thm
Let $(X,Y)$ be a $2$-dimensional random vector with distribution $P_{XY}$. Then the marginal distributions $P_X$ and $P_Y$ may be obtained via the formulas

$$
P_X(A) = P_{XY}(A\times \mathbb{R}) \quad \text{and} \quad P_Y(B) = P_{XY}(\mathbb{R} \times B)
$$

for all events $A,B\subset \mathbb{R}$. In particular:

1. If $(X,Y)$ is discrete with probability mass function $p(x,y)$, then

    $$
    P(X\in A) = \sum_{x\in A, \ y\in \mathbb{R}}p(x,y) \quad \text{and} \quad P(Y\in B) = \sum_{y\in B, \ x\in \mathbb{R}} p(x,y).
    $$

2. If $(X,Y)$ is continuous with probability density function $f(x,y)$, then

    $$
    P(X\in A) = \int_A \int_{-\infty}^\infty f(x,y) \ \text{d}x \text{d}y
    $$
  
    and

    $$
    P(Y\in B) = \int_B \int_{-\infty}^\infty f(x,y) \ \text{d}y \text{d}x.
    $$
```

An immediate corollary of this result is a description of the marginal probability mass and density functions. We will use this result often:

```{prf:theorem}
:label: marginal-thm

Let $(X,Y)$ be a $2$-dimensional random vector.

1. If $(X,Y)$ is discrete with probability mass function $p(x,y)$, then both $X$ and $Y$ are discrete with probability mass functions given by

    $$
    p_X(x) = \sum_{y\in \mathbb{R}} p(x,y) \quad \text{and} \quad p_Y(y) = \sum_{x\in \mathbb{R}}p(x,y).
    $$
  
2. If $(X,Y)$ is continuous with probability density function $f(x,y)$, then both $X$ and $Y$ are continuous with probability density functions given by 

    $$
    f_X(x) = \int_{-\infty}^\infty f(x,y) \ \text{d}y \quad \text{and} \quad f_Y(y) = \int_{-\infty}^\infty f(x,y) \ \text{d} x.
    $$ (marg-cont-eqn)

```

Here's how I remember these formulas:

```{tip}
1. To obtain the marginal density $f_X(x)$ from the joint density $f(x,y)$, we "integrate out" the dependence of $f(x,y)$ on $y$. Likewise for obtaining $f_Y(y)$ from $f(x,y)$.

2. To obtain the marginal mass $p_X(x)$ from the joint mass $p(x,y)$, we "sum out" the dependence of $p(x,y)$ on $y$. Likewise for obtaining $p_Y(y)$ from $p(x,y)$.
```

In the continuous case, you should visualize the formulas {eq}`marg-cont-eqn` as integrations over cross-sections of the density surfaces. For example, the following picture is a visualization of the formula for the marginal density function $f_Y(y)$ evaluated at $y=5$:

```{image} ../img/margDensity.svg
:width: 70%
:align: center
```
&nbsp;

In the picture, we imagine slicing the graph of the density $f(x,y)$ with the vertical plane $y=5$. (We must imagine this plane extends off infinitely far in each direction.) Where the plane and the density surface intersect, a curve will be traced out on the plane. The area beneath _this_ cross-sectional curve is exactly the value

$$
f_Y(5) = \int_{-\infty}^\infty f(x,5) \ \text{d}x.
$$

This is the visualization for continuous distributions---what might the analogous visualization look like for discrete distributions? Can you sketch the figure on your own?

```{admonition} Problem Prompt
Do problems 6 and 7 on the worksheet.
```


















## Bivariate empirical distributions

Almost everything that we learned in the [previous chapter](theory-to-practice) on data and empirical distributions may be applied in the bivariate setting. Essentially, you just need to change all random variables to random vectors, and put either a "joint" or "bivariate" in front of everything. :)

To illustrate, let's return to our housing data explored in the [first section](motivation). There, we had _two_ observed random samples

$$
x_1,x_2,\ldots,x_n \quad \text{and} \quad y_1,y_2,\ldots,y_n
$$

of the sizes (in square feet) and the selling prices (in thousands of dollars) of a subset of $n=500$ houses. In the precise language of the [previous chapter](theory-to-practice), we would say that these datasets are observations from the IID random samples

$$
X_1,X_2,\ldots,X_n \quad \text{and} \quad Y_1,Y_2,\ldots,Y_n.
$$

Make sure you remember the difference between a random sample and an _observed_ random sample!

However, the $i$-th size $x_i$ and the $i$-th price $y_i$ naturally go together, since they are both referring to the $i$-th house. Therefore, we might instead consider our two observed random samples as a _single_ observed random sample

$$
(x_1,y_1), (x_2,y_2),\ldots,(x_n,y_n).
$$

But what is this new observed random sample an observation of? Answer:

```{prf:definition}
:label: random-sample-vec-defn

Let $(X_1,Y_1), (X_2,Y_2),\ldots,(X_n,Y_n)$ be a sequence of $2$-dimensional random vectors, all defined on the same probability space.

* The random vectors are called a _bivariate random sample_ if they are _independent_ and _identically distributed_ (IID).

Provided that the sequence is a bivariate random sample, an _observed bivariate random sample_, or a _bivariate dataset_, is a sequence of pairs of real numbers

$$
(x_1,y_1), (x_2,y_2),\ldots,(x_n,y_n)
$$

where $(x_i,y_i)$ is an observation of $(X_i,Y_i)$.
```

To say that $(x_i,y_i)$ is an observed value of the random vector $(X_i,Y_i)$ simply means that it is in the range of the random vector (as a function with codomain $\mathbb{R}^2$). As I remarked in the [previous chapter](theory-to-practice), we haven't officially defined _independence_ yet---that will come in {prf:ref}`independence-defn` below.

```{margin}
An interesting question is raised as to whether the individual independence of the pair of observed univariate random samples is equivalent to independence of the observed bivariate random sample. We will address this in the next chapter.
```
Thus, it might be more natural to say that our housing data constitutes an observed bivariate random sample, rather than just two individual observed univariate random samples. These types of random samples---along with their higher-dimensional cousins called _multivariate random samples_---are quite common in prediction tasks where we aim to predict the $y_i$'s based on the $x_i$'s. For example, this is the entire gist of _supervised machine learning_.

Adapting the definition of empirical distributions of univariate datasets from the [previous chapter](theory-to-practice) is also easy:

```{prf:definition}
Let $(x_1,y_1),(x_2,y_2),\ldots,(x_n,y_n)$ be an observed bivariate random sample, i.e., a bivariate dataset. The _empirical distribution_ of the dataset is the discrete probability measure on $\mathbb{R}^2$ with joint probability mass function

$$
p(x,y) = \frac{\text{number of data points $(x_i,y_i)$ that match $(x,y)$}}{n}.
$$
```

If we assume that our bivariate dataset consists of observations drawn from an IID random sample

$$
(X_1,Y_1),(X_2,Y_2),\ldots,(X_n,Y_n),
$$

then the empirical distribution of the dataset is supposed to serve as an approximation for the _true_ joint distribution of the $(X_i,Y_i)$'s.

We saw in the [first section](motivation) that we may visualize bivariate empirical distributions using scatter plots. Here's a variation on a scatter plot, which places the marginal empirical distributions in the (where else?) margins of the plot!

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center
:   image:
:       width: 70%

sns.jointplot(x=X, y=Y, marginal_kws={'kde' : True})
plt.xlabel(r'$x=$size (ft$^2$)')
plt.ylabel(r'$y=$price (\$1,000)')
plt.tight_layout()
```

To be a bit more precise, along the top of the figure we see a histogram (with KDE) of the empirical distribution of the $x_i$'s, and along the side we see a histogram (with KDE) of the empirical distribution of the $y_i$'s.

This type of figure makes it very clear how marginal distributions are obtained from joint distributions. For example, take a look at:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center  
:   image:
:       width: 70%

import pandas as pd

# combine the data into a DataFrame
joint_data = pd.DataFrame({'X' : X, 'Y' : Y})
slice_data = joint_data[(178 <= joint_data['Y']) & (joint_data['Y'] <= 180)]

# instantiate a JointGrid object
g = sns.JointGrid()

# generate the plots
sns.scatterplot(data=joint_data, x='X', y='Y', ax=g.ax_joint, alpha=0.05,)
sns.scatterplot(data=slice_data, x='X', y='Y', ax=g.ax_joint)
ax1 = sns.histplot(data=joint_data, x='X', ax=g.ax_marg_x, ec='b', color='w')
ax2 = sns.histplot(data=joint_data, y='Y', ax=g.ax_marg_y, ec='b')

# change one of the bars to magenta
for bar in ax2.patches:
    bar.set_facecolor('w')
ax2.patches[16].set_facecolor('#FD46FC')

g.set_axis_labels(xlabel=r'$x=$size (ft$^2$)', ylabel=r'$y=$price (\$1,000)')
plt.tight_layout()
```

Here, I have highlighted one of the histogram bars in the empirical distribution of the $y_i$'s. The height of this bar is obtained by counting all the data points in the corresponding highlighted horizontal slice. This is a direct translation of the formula

$$
p_Y(y) = \sum_{x\in \mathbb{R}} p_{XY}(x,y)
$$

for obtaining a marginal PMF $p_Y(y)$ from a joint PMF $p_{XY}(x,y)$ by summing over all $x$'s with a fixed $y$-value.

What about bivariate versions of KDEs and histograms? Answer:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center
:   image:
:       width: 100%

_, (ax1, ax2) = plt.subplots(ncols=2,
                             figsize=(9, 4),
                             sharey=True,
                             sharex=True)

sns.kdeplot(x=X, y=Y, ax=ax1)
sns.histplot(x=X, y=Y,
             cbar=True,
             ax=ax2,
             cbar_kws={'label': 'frequency'})

ax1.set_title('bivariate kde')
ax2.set_title('bivariate histogram')
ax1.set_xlabel(r'$x=$size (ft$^2$)')
ax2.set_xlabel(r'$x=$size (ft$^2$)')
ax1.set_ylabel(r'$y=$price (\$1,000)')
plt.tight_layout()
```

Even though the density surface for a bivariate empirical distribution is _not_ a continuous surface, if it _were_, you can imagine that the curves in the KDE on the left are its contours. In other words, these are the curves over which the density surface has constant height. It appears that the density surface has either a global minimum or global maximum near $(2000,175)$, but we can't tell which from the KDE alone because the contours are not labelled.

On the right-hand side of the figure above, we have a bivariate version of a histogram. While a histogram for a univariate dataset is obtained by subdividing the line $\mathbb{R}$ into bins, for a bivariate dataset the plane $\mathbb{R}^2$ is subdivided into rectangle bins. Then, over each of these rectangular bins we would place a $3$-dimensional "bar" whose height is equal (or proportional) to the number of data points that fall in the bin; thus, a histogram for bivariate data should really live in three dimensions ($xyz$-space). However, the histogram above shows only the bins in the plane $\mathbb{R}^2$, and it displays the heights of the "bars" by color, with darker shades of blue indicating a larger number of data points are contained in the bin. It is evident from this diagram that the global extreme point identified in the KDE is, in fact, a global maximum.
















## Conditional distributions

Given two events $A$ and $B$ in a probability space, we learned [previously](cond-prob) that the _conditional probability of $A$ given $B$_ is defined via the formula

$$
P(A|B) = \frac{P(A\cap B)}{P(B)},
$$

provided that $P(B) \neq 0$. The probability on the left is the probability that $A$ occurs, _given_ that you already know the event $B$ has occurred. One may view $P(-|B)$ (the "$-$" means "blank") as a probability measure with sample space $B$ and where all events are of the form $A\cap B$. It is worth repeating this, in slightly simpler language:

> Passing from plain probabilities to conditional probabilities has the effect of shrinking the sample space to the event that you are "conditioning on."

Let's see how this might work with the probability measures induced by random variables.

To get a feel for what we're going for, let's return to our housing data

$$
(x_1,y_1),(x_2,y_2),\ldots,(x_n,y_n)
$$

and its bivariate empirical distribution that we studied in the previous section. Suppose that we are interested in studying the (empirical) distribution of sizes $x$ of houses with fixed sale price $y=180$. If we set $B = \{180\}$, then this means we want to shrink the range of the $y$'s down from all of $\mathbb{R}$ to the simple event $B$. The slice of data points with $y=180$ are highlighted in the following scatter plot:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center  
:   image:
:       width: 80%

# combine the data into a DataFrame
joint_data = pd.DataFrame({'X' : X, 'Y' : Y})
slice_data = joint_data[joint_data['Y'] == 180]

sns.scatterplot(x=X, y=Y, alpha=0.05)
sns.scatterplot(data=slice_data, x='X', y='Y')
plt.xlabel(r'$x=$size (ft$^2$)')
plt.ylabel(r'$y=$price (\$1,000)')
plt.tight_layout()
```

Then, after cutting down the range of $y$'s to lie in $B=\{180\}$, we wonder what the distribution over the sizes $x$ looks like. Answer:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center  
:   image:
:       width: 70%

# instantiate a JointGrid object
g = sns.JointGrid()

# generate the plots
scatter = sns.scatterplot(data=joint_data, x='X', y='Y', ax=g.ax_joint, alpha=0.05,)
sns.scatterplot(data=slice_data, x='X', y='Y', ax=g.ax_joint)
sns.histplot(data=slice_data, x='X', ax=g.ax_marg_x, ec='b', color='#FD46FC', kde=True)

g.ax_marg_y.remove()
g.set_axis_labels(xlabel=r'$x=$size (ft$^2$)', ylabel=r'$y=$price (\$1,000)')
plt.tight_layout()
```

```{margin}
If you haven't caught on by now, the word _empirical_ usually always means "observed" or "based on a dataset."

```
The histogram along the top of the figure shows the empirical distribution of the $x$'s belonging to data points $(x,y)$ with $y=180$. If we remember that our original random variables in the [first section](motivation) were $X$ and $Y$, then this empirical distribution is an approximation to the _conditional distribution of $X$ given $Y=180$._ (The precise definition is below.) So, the histogram along the top of the scatter plot displays an _empirical_ conditional distribution.


```{margin}
At the most general level, defining _conditional distributions_ is surprisingly very difficult. Have a look at [this](https://en.wikipedia.org/wiki/Conditional_probability_distribution#Measure-theoretic_formulation) page if you don't believe me.
```
Alright. We're ready for the definitions. At this level, it turns out that the easiest way to define conditional distributions is via mass and density functions:

```{prf:definition}
Let $X$ and $Y$ be random variables.

* Suppose $(X,Y)$ is discrete, so that both $X$ and $Y$ are discrete as well. The _conditional probability mass function of $X$ given $Y$_ is the function

    $$
    p_{X|Y}(x|y) = \frac{p_{XY}(x,y)}{p_Y(y)},
    $$

    defined for all those $y$ such that $p_Y(y)\neq 0$.

* Suppose $(X,Y)$ is continuous, so that both $X$ and $Y$ are continuous as well. The _conditional probability density function of $X$ given $Y$_ is the function

    $$
    f_{X|Y}(x|y) = \frac{f_{XY}(x,y)}{f_Y(y)},
    $$

    defined for all those $y$ such that $f_Y(y)\neq 0$.

```

Let's get some practice:

```{admonition} Problem Prompt
Do problems 8 and 9 on the worksheet.
```

Just _calling_ $p_{X|Y}(x|y)$ a probability mass function does not _make_ it so, and similarly for $f_{X|Y}(x|y)$. So, in what sense do these define probability measures?

```{prf:theorem}
Let $X$ and $Y$ be random variables.

* In the case that $(X,Y)$ is discrete, for fixed $y$ with $p_Y(y)\neq 0$, the function $p_{X|Y}(x|y)$ is a probability mass function in the variable $x$. In particular, we have

    $$
    P(X\in A|Y=y) = \sum_{x\in A} p_{X|Y}(x|y),
    $$ (cond-dist-eqn)

    for all events $A\subset \mathbb{R}$.

* In the case that $(X,Y)$ is continuous, for fixed $y$ with $f_Y(y)\neq 0$, the function $f_{X|Y}(x|y)$ is a probability density function in the variable $x$.
```

Let me show you how the discrete case works, leaving you do adapt the arguments to the continuous case on your own. First, note that $p_{X|Y}(x|y)$ is nonnegative, as all PMF's must be. Thus, we need only check that it sums to $1$ over all $x$:

$$
\sum_{x\in \mathbb{R}}p_{X|Y}(x|y) = \frac{1}{p_Y(y)} \sum_{x\in \mathbb{R}}p_{XY}(x,y) = \frac{P(\mathbb{R} \times \{y\})}{p_Y(y)}  = \frac{p_Y(y)}{p_Y(y)} = 1,
$$

where I used {prf:ref}`marg-thm` in the second equality. The same type of argument will prove {eq}`cond-dist-eqn`, which I will also let you do on your own.

```{warning}
Notice the absence of an analogous equation to {eq}`cond-dist-eqn` for conditional density functions! This is because the left-hand side of {eq}`cond-dist-eqn` is equal to the ratio

$$
\frac{P(X\in A, Y=y)}{P(Y=y)}.
$$

But _both_ the numerator and denominator of this fraction are $0$ in the case that $Y$ is continuous! _So what probability does $f_{X|Y}(x|y)$ compute?_

Answering this question precisely is hard---this is part of what I was alluding to in my margin note above. But here's the rough idea: Suppose that $\epsilon$ is a small, positive real number. Then the conditional probability

$$
P(X\in A | y \leq Y \leq y+\epsilon) = \frac{P(X\in A, y\leq Y \leq y+\epsilon)}{P(y\leq Y \leq y+\epsilon)}
$$

at least has a _chance_ to be well-defined, since the denominator

$$
P(y\leq Y \leq y+\epsilon) = \int_y^{y+\epsilon} f_Y(y^\star) \ \text{d}y^\star
$$

can be nonzero. But we also have

$$
P(X\in A, y\leq Y \leq y+\epsilon) = \int_y^{y+\epsilon} \int_A f_{XY}(x, y^\star) \ \text{d}x \text{d}y^\star,
$$

and so substituting these last two expressions into the conditional probability gives

\begin{align*}
P(X\in A | y \leq Y \leq y+\epsilon) &= \frac{\int_y^{y+\epsilon} \int_A f_{XY}(x, y^\star) \ \text{d}x \text{d}y^\star}{\int_y^{y+\epsilon} f_Y(y^\star) \ \text{d}y^\star} \\
&\approx \frac{\epsilon\int_A f_{XY}(x, y) \ \text{d}x}{\epsilon f_Y(y) \ } \\
&= \int_A \frac{f_{XY}(x,y)}{f_Y(y)} \ \text{d} x \\
&= \int_A f_{X|Y}(x|y) \ \text{d} x.
\end{align*}

The interpretation is this: For fixed $y$, integrating the conditional density $f_{X|Y}(x|y)$ over $x\in A$ yields the probability that $X\in A$, _given_ that $Y$ is in an "infinitesimal" neighorhood of $y$. (This "infinitesimal" neighborhood is represented by $[y,y+\epsilon]$, when $\epsilon$ is really small.)

```

In spite of this warning, we shall _still_ imagine that the conditional density $f_{X|Y}(x|y)$ is the density of the conditional probability $P(X\in A | Y=y)$, though technically the latter is undefined according to the standard definition of conditional probability. You will see this in:

```{admonition} Problem Prompt

Do problems 10 and 11 on the worksheet.
```














## The Law of Total Probability and Bayes' Theorem for random variables

Back in {numref}`total-prob-bayes`, we studied the Law of Total Probability and Bayes' Theorem for arbitrary probability measures. In this section, we adapt these results to the probability measures induced by random variables.

```{prf:theorem} The Law of Total Probability (for random variables)

Let $X$ and $Y$ be random variables.

* If $X$ and $Y$ are jointly discrete, then

    $$
    p_X(x) = \sum_{y\in \mathbb{R}} p_{X|Y}(x|y) p_Y(y)
    $$ (disc-law-eqn)

    for each $x\in \mathbb{R}$.

* If $X$ and $Y$ are jointly continuous, then

    $$
    f_X(x) = \int_{\mathbb{R}} f_{X|Y}(x|y) f_Y(y) \ \text{d}y.
    $$ (cont-law-eqn)

    for each $x\in \mathbb{R}$.
```

Let me show you how to prove {eq}`cont-law-eqn`; I will leave the other equality {eq}`disc-law-eqn` for you to do on your own. We want to prove that the right-hand side of {eq}`cont-law-eqn` is the PDF of $X$. So, we integrate it over $[a,b]$ and hope that the probability $P(a\leq X \leq b)$ pops out. Here are the computations:

\begin{align*}
\int_a^b \left[\int_{-\infty}^\infty f_{X|Y}(x|y) f_Y(y) \ \text{d}y \right] \ \text{d} x &= \int_a^b \left[\int_{-\infty}^\infty f_{XY}(x,y) \ \text{d}y \right] \ \text{d} x \\
&= \int_a^b f_X(x) \ \text{d} x \\
&= P(a\leq X \leq b).
\end{align*}

In moving from the first line to the second, you'll notice that I "integrated out" the dependence on $x$ of the joint PDF, obtaining the marginal PDF of $X$ as a result. Since indeed the probability $P(a\leq X \leq b)$ was obtained by integrating the right-hand side of {eq}`cont-law-eqn`, this proves that it is the PDF of $X$.

```{prf:theorem} Bayes' Theorem (for random variables)

Let $X$ and $Y$ be random variables.

* If $X$ and $Y$ are jointly discrete, then

    $$
    p_{X|Y}(x|y) = \frac{p_{Y|X}(y|x) p_X(x)}{p_Y(y)}.
    $$ (bayes-disc-eqn)

* If $X$ and $Y$ are jointly continuous, then

    $$
    f_{X|Y}(x|y) = \frac{f_{Y|X}(y|x) f_X(x)}{f_Y(y)}.
    $$ (bayes-cont-eqn)
```

The proofs of these two equations follow immediately from the definitions of conditional mass and density functions. In applications, you will often see Bayes' Theorem combined with the Law of Total Probability, the latter allowing one to compute the denominators in {eq}`bayes-disc-eqn` and {eq}`bayes-cont-eqn`. For example, in the continuous case, we have

$$
f_{X|Y}(x|y)  = \frac{f_{Y|X}(y|x) f_X(x)}{\int_{\mathbb{R}} f_{Y|X}(y|x^\star) f_X(x^\star) \ \text{d}x^\star}
$$

for all $x,y$. The advantage gained by writing the denominator like this is that one _only_ needs information about the conditional density $f_{Y|X}(y|x)$ and the marginal density $f_X(x)$ in order to compute the other conditional density $f_{X|Y}(x|y)$.


```{note}

Notice that the random vector $(X,Y)$ was required to be either discrete or continuous in both the Law of Total Probability and Bayes' Theorem. Actually, there are versions of many of the definitions and results in this chapter for "mixed" joint distributions in which one of $X$ or $Y$ is continuous and the other is discrete (as long as an analog of a mass or density function exists). I won't state these more general results here because it would be incredibly awkward and tedious to cover all possible cases using our limited theory and language. This is a situation where the machinery of measure theory is needed.
```

In any case, you'll see an example of one of these "mixed" distributions in the following Problem Prompt in which I introduce you to the canonical example in Bayesian statistics. (See also {numref}`untrustworthy` below.)


```{admonition} Problem Prompt

Do problem 12 on the worksheet.
```











## Random vectors in arbitrary dimensions

Up till now in this chapter, we have studied pairs of random variables $X$ and $Y$, or what is the same thing, $2$-dimensional random vectors $(X,Y)$. But there's an obvious generalization of these considerations to higher dimensions:

```{prf:definition}

Let $S$ be a probability space and $n\geq 1$ an integer. An _$n$-dimensional random vector_ is a function

$$
\mathbf{X}: S \to \mathbb{R}^n.
$$

Thus, we may write

$$
\mathbf{X}(s) = (X_1(s),X_2(s),\ldots,X_n(s))
$$

for each sample point $s\in S$. When we do so, the functions $X_1,X_2,\ldots,X_n$ are ordinary random variables that are called the _components_ of the random vector $\mathbf{X}$.
```

So, an $n$-dimensional random vector is nothing but a sequence of random variables $X_1,X_2,\ldots,X_n$. These vectors are thus _immensely important_, since they are connected with random samples and datasets as defined back in {prf:ref}`random-sample-defn`.

Random vectors in dimensions $>2$ induce joint probability distributions, just like their $2$-dimensional relatives:

```{prf:definition}
Let $(X_1,X_2,\ldots,X_n):S \to \mathbb{R}^n$ be an $n$-dimensional random vector on a probability space $S$ with probability measure $P$. We define the _probability measure_ of the random vector, denoted $P_{(X_1,X_2,\ldots,X_n)}$ or $P_{X_1X_2\cdots X_n}$, via the formula

$$
P_{X_1X_2\cdots X_n}(C) = P \left( \{s\in S \mid (X_1(s),X_2(s),\ldots,X_n(s))\in C\} \right),
$$ (hard2-eqn)

for all events $C\subset \mathbb{R}^n$. The probability measure $P_{X_1X_2\cdots X_n}$ is also called the _joint distribution_ of the component random variables $X_1,X_2,\ldots,X_n$.
```

The equation {eq}`hard2-eqn` is the _precise_ definition of the joint distribution for _any_ event $C$ in $\mathbb{R}^n$. But if $C$ happens to be an event of the form

$$
C = \{ (x_1,x_2,\ldots,x_n) : x_1\in A_1, x_2\in A_2,\ldots, x_n \in A_n\}
$$

for some events $A_1,A_2,\ldots,A_n\subset \mathbb{R}$, then we shall _always_ write

$$
P(X_1\in A_1, X_2\in A_2,\ldots, X_n \in A_n)
$$ (clock-eqn)

in place of $P_{X_1X_2\cdots X_n}(C)$. Again, this expression {eq}`clock-eqn` is technically an abuse of notation, but it is tolerated because it is so much more descriptive and intuitive.


```{margin}

If you want to see precise statements of definitions and theorems in dimensions $>2$, have a look at Section 3.7 in {cite}`DeGrootSchervish2014`.
```
Almost all the definitions and results that we considered above for $2$-dimensional random vectors have obvious generalizations to higher-dimensional random vectors. This includes higher-dimensional marginal and conditional distributions, as well as Laws of Total Probability and Bayes' Theorems. Provided that you understand the $2$-dimensional situation well, I am confident that the higher-dimensional case should pose no problem. Therefore, we will content ourselves with working through a few example problems, in place of an exhaustive account of all the definitions and theorems.

```{admonition} Problem Prompt

Do problems 13 and 14 on the worksheet.
```













(independence)=
## Independence

Because of its central role in the definitions of random samples and datasets (see {prf:ref}`random-sample-defn` and {prf:ref}`random-sample-vec-defn`), _independence_ is one of the most important concepts in all probability and statistics. In a somewhat different order compared to previous chapters, I will begin our discussion with the abstract definitions before moving on to an extended example from Bayesian statistics which highlights how independence is used in practice and brings together many of the concepts we have studied in past chapters.


We already studied a form of _independence_ back in {numref}`independence-first`, where we saw that two events $A$ and $B$ in a probability space are _independent_ if

$$
P(A\cap B) = P(A) P(B).
$$ (ind-fan-eqn)

As long as $P(B)\neq 0$ (if not, then both sides of this last equation are $0$), independence is the same as

$$
P(A|B) = P(A).
$$ (cond-ind-eqn)

This latter equation is telling us that $A$ and $B$ are independent provided that the conditional probability of $A$, given $B$, is just the plain probability of $A$. In other words, if $A$ and $B$ are independent, then whether $B$ has occurred has no impact on the probability of $A$ occurring.

Our mission in this chapter is to adapt these definitions to the probability measures induced by random variables. The key step is to replace the left-hand side of {eq}`ind-fan-eqn` with a joint probability distribution. We make this replacement in the next defintion, while also generalizing to an arbitrary number of random variables:

```{prf:definition}
:label: independence-defn

Let $X_1,\ldots,X_n$ be random variables, all defined on the same probability space. Then these random variables are said to be _independent_ if

$$
P(X_1\in A_1,\ldots,X_n \in A_n) = P(X_1\in A_1) \cdots P(X_n\in A_n)
$$

for all events $A_1,\ldots,A_n\subset \mathbb{R}$. If the variables are not independent, they are called _dependent_.
```

Notice that no conditions are placed on the random variables $X_1,\ldots,X_n$ in this definition, like assuming they are discrete or continuous. However, provided that mass or density functions exist, then convenient criteria for independence may be obtained in terms of these functions:

```{prf:theorem} Mass/Density Criteria for Independence
Let $X_1,\ldots,X_n$ be random variables.

* Suppose that the random variables are jointly discrete. Then they are independent if and only if

  $$
  p_{X_1\cdots X_n}(x_1,\ldots,x_n) = p_{X_1}(x_1) \cdots p_{X_n}(x_n)
  $$

  for all $x_1,\ldots,x_n \in \mathbb{R}$.

* Suppose that the random variables are jointly continuous. Then they are independent if and only if

  $$
  f_{X_1\cdots X_n}(x_1,\ldots,x_n) = f_{X_1}(x_1) \cdots f_{X_n}(x_n)
  $$ (cont-factor-eqn)

  for all $x_1,\ldots,x_n \in \mathbb{R}$.
```

Let's outline a quick proof of {eq}`cont-factor-eqn` in the case that there are only two random variables $X$ and $Y$. In any case, we have

\begin{align*}
P(X\in A) P(Y\in B) &= \int_Af_X(x) \ \text{d} x \int_B f_Y(y) \ \text{d}y \\
&= \int_B \int_A f_X(x) f_Y(y) \ \text{d}x\text{d} y \\
&= \iint_{A\times B} f_X(x) f_Y(y) \ \text{d} x \text{d}y
\end{align*}

for all events $A,B\subset \mathbb{R}$. Therefore, if $X$ and $Y$ are independent, then this shows the joint probability of $A\times B$ is the integral over the product $f_X(x)f_Y(y)$, which is enough to show that the joint density is equal to this product. Conversely, if the joint density factors, then these computations also show that $X$ and $Y$ are independent.

It turns out that there is also a characterization of independence in terms of factorizations of joint cumulative distribution functions. This characterization is actually taken as the _definition_ of independence in some references (e.g., {cite}`Wackerly2014`).


```{admonition} Problem Prompt

Do problems 15-18 on the worksheet.
```

Before moving on to the last section of this chapter, we state a "random variable" version of the equation {eq}`cond-ind-eqn` describing independent events in terms of conditional probabilities.

```{prf:theorem} Conditional Criteria for Independence

Let $X$ and $Y$ be two random variables.

* Suppose $X$ and $Y$ are jointly discrete. Then they are independent if and only if

    $$
    p_{X|Y}(x|y)  = p_X(x)
    $$

    for all values of $y$ such that $p_Y(y)>0$ and all $x\in \mathbb{R}$.

* Suppose $X$ and $Y$ are jointly continuous. Then they are independent if and only if

    $$
    f_{X|Y}(x|y)  = f_X(x)
    $$

    for all values of $y$ such that $f_Y(y)>0$ and all $x\in \mathbb{R}$.
```








(untrustworthy)=
## Case study: an untrustworthy friend

My goal in this section is to step through an extended example that illustrates how independence is often used in computations involving real data. The scenario is taken from Bayesian statistics, and is similar to problem 12 on the worksheet. The point is not for you to learn the techniques and philosophy of Bayesian statistics in depth---that would require an entire course of its own. Instead, view this section as an enjoyable excursion into more advanced techniques that you might choose to study later.

The situation we find ourselves in is this:

```{margin}

I am using the letter $\theta$ for the probability rather than $p$ because the latter is overloaded in this scenario, as it also represents probability mass functions. Blame the statistical community for using $p$ for PMFs.
```

```{admonition} The Canonical Bayesian Coin-Flipping Scenario

Our friend suggests that we play a game, betting money on whether a coin flip lands heads or tails. If the coin lands heads, our friend wins; if the coin lands tails, we win.

However, our friend has proven to be untrustworthy in the past. We suspect that the coin might be unfair, with a probability of $\theta = 0.75$ of landing heads. So, before we play the game, we collect data and flip the coin ten times and count the number of heads. Depending on our results, how might we alter our prior estimate of $\theta = 0.75$ for the probability of landing heads?
```

You might initially wonder why we need any statistical theory at all. Indeed, we might base our assessment of whether the coin favors us or our friend entirely on the proportion of heads that we see in our ten flips: If we see six or more heads, then we might believe $\theta>0.5$ and that the coin favors our friend, while if we obtain four or less, then $\theta < 0.5$ and the coin favors us.

But the crucial point is that our friend has an untrustworthy track record. Since we believe at the outset that $\theta$ is somewhere around $0.75$, seeing four heads is not enough to "offset" this large value and tug it down below $0.5$. So, the situation is a bit more complex than it might appear at first glance.

The goal is to find a _systematic_ and _quantitative_ method for updating our assessment of the likely values of $\theta$ based on the data. The keys to the computations will be Bayes' Theorem (as you might have guessed) and _independence_.

Let's begin with our prior estimate $\theta = 0.75$ for the probability of landing heads. We might go a step further than this single (point) estimate and actually cook up an _entire_ probability distribution for what we believe are the most likely values of $\theta$. Perhaps we suppose that $\theta$ is an observation of a $\mathcal{B}eta(6,2)$ random variable:

```{margin}

Notice that I am abusing the notation somewhat by writing $\theta \sim \mathcal{B}eta(6,2)$. This is formally incorrect, since $\theta$ is supposed to be an _observed value_ of the random variable, not the random variable itself.
```

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center
:   image:
:       width: 70%

from scipy.stats import beta

Theta = beta(a=6, b=2)
theta = np.linspace(0, 1, 150)

plt.plot(theta, Theta.pdf(theta))
plt.xlabel(r'$\theta$')
plt.ylabel('probability density')
plt.suptitle(r'$\theta \sim \mathcal{B}eta(6,2)$')
plt.tight_layout()
```

```{margin}

Of course, there are infinitely many distributions on the interval $(0,1)$ with an expected value of $0.75$, so our choice of a $\mathcal{B}eta(6,2)$ distribution is by no means the only one. Choosing these "prior distributions" can be quite difficult in general, and there are many considerations that go into making a good choice. Our choice of a beta distribution is primarily motivated according to [these](https://en.wikipedia.org/wiki/Conjugate_prior) reasons.
```

Notice that this distribution favors values of $\theta$ toward $1$, which is consistent with our friend's untrustworthy track record. Also, if $\theta$ really does come from a $\mathcal{B}eta(6,2)$ random variable, then as we learned back in {prf:ref}`exp-beta-thm`, its expected value is indeed $6/(6+2) = 0.75$. Suppose we write

$$
f(\theta) \propto \theta^5 (1-\theta)
$$

for the density function of $\theta$ drawn from the $\mathcal{B}eta(6,2)$ distribution.

Now, suppose that we flip the coin ten times. It is natural to realize the resulting dataset as an observation from an identically distributed random sample

$$
X_1,X_2,\ldots,X_{10}
$$

with $X_i \sim \mathcal{B}er(\theta)$ for each $i$ and a value $x_i=1$ indicating that a head was obtained on the $i$-th flip. It is also natural to assume that the flips are independent of each other (conditioned on $\theta$). Therefore, the joint mass function of the random sample (conditioned on $\theta$) factors as

$$
p(x_1,x_2,\ldots,x_{10}| \theta) = \prod_{i=1}^{10} p(x_i|\theta) = \prod_{i=1}^{10}\theta^{x_i}(1-\theta)^{1-x_i} = \theta^x (1-\theta)^{10-x},
$$ (factor-likelihood-eqn)

where $x = x_1+x_2+\cdots +x_{10}$ is the total number of heads in the dataset.

```{margin}

Recall from the worksheet in the previous chapter that the new "updated" distribution is often called the _posterior distribution_.
```

We now use Bayes' Theorem to update our prior $\mathcal{B}eta(6,2)$ distribution for $\theta$ to a new distribution with density $f(\theta|x_1,x_2,\ldots,x_{10})$. Assuming that the values $x_1,x_2,\ldots,x_{10}$ are fixed, observed values, here are the computations:

\begin{align*}
f(\theta| x_1,\ldots,x_{10}) &= \frac{p(x_1,\ldots,x_{10}|\theta)f(\theta)}{p(x_1,\ldots,x_{10})} \\
&\propto p(x_1,\ldots,x_{10}|\theta)f(\theta)\\
&= \theta^x(1-\theta)^{10-x} \theta^5 (1-\theta) \\
&= \theta^{(x+6)-1} ( 1-\theta)^{(12-x)-1}.
\end{align*}

Thus, the "updated" distribution must be of the form $\mathcal{B}eta(x+6,12-x)$, where $x$ is the number of heads in the dataset with $0\leq x \leq 10$. Let's plot these eleven distributions:

```{code-cell} ipython3
:tags: [hide-input]
:mystnb:
:   figure:
:       align: center
:   image:
:       width: 70%

from cycler import cycler

color = [
    '#486afb',
    '#4579ff',
    '#4788ff',
    '#4f95ff',
    '#5ba2ff',
    '#6aafff',
    '#7bbbff',
    '#8ec7ff',
    '#a2d2ff',
    '#b6ddff',
    '#cce8ff'
]

default_cycler = cycler(color=color)
plt.rc('axes', prop_cycle=default_cycler)

for x in range(10, -1, -1):
    Theta_posterior = beta(a=x + 6, b=12 - x)
    plt.plot(theta, Theta_posterior.pdf(theta), label=rf'$x={x}$')

plt.xlabel(r'$\theta$')
plt.ylabel('probability density')
plt.legend()
plt.tight_layout()
```

Here's what you should notice: For larger values of $x$, the distributions are shifted toward higher values of $\theta$, which reflects the fact that many heads suggests $\theta$ is close to $1$. In the other direction, smaller values of $x$ shift the distributions toward $0$.

In particular, for $x=3$ we obtain an "updated" distribution of $\mathcal{B}eta(9,9)$, which has the symmetric density curve in the figure. The expected value for this distribution is exactly $0.5$, so we would need to see 3 heads in our dataset to conclude that the coin has a good chance of being fair (at least according to the "posterior mean"). But if we see four or more heads ($x\geq 4$), then our computations still support the conclusion that our friend is untrustworthy since the distributions in this range have expected values $>0.5$. In the other direction, if we see two or fewer heads ($x\leq 2$), then we might believe that the coin favors us.

```{note}

It is worth addressing this point again: Why do we need to see _three_ heads to conclude that the coin might be fair, and not _five_ heads?

Remember, we believe that our friend is untrustworthy. If we believed that $\theta=0.75$, then seeing five heads is not enough evidence to convince us that $\theta$ is, in reality, near $0.5$. We would need to see _even fewer_ heads to overcome (or offset) the prior estimate of $\theta=0.75$.
```

Actually, my description here is slightly at odds with a strict interpretation of Bayesian philosophy, since a true Bayesian would never assume that the parameter $\theta$ has a fixed valued, only that it has a prior probability distribution. My description has shades of [hypothesis testing](hyp-test).

Notice that independence of the coin flips was _absolutely crucial_ for the computations to go through. Without assuming independence, we would not have been able to factor the joint mass function (conditioned on $\theta$) as in {eq}`factor-likelihood-eqn`, which would have made our application of Bayes' Theorem much more difficult. This joint mass function is actually known as the _likelihood function_ $\mathcal{L}(\theta)$ of the parameter $\theta$, and we will see these same computations appear again when we study [_maximum likelihood estimation_](point-est) (_MLE_).