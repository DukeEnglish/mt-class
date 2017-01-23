---
layout: common
title: Probability and language models 
---

$$n$$-gram language models
--------------------------

If you aren't familiar with the basics of probability, please go read
Sharon Goldwater's [excellent tutorial](http://homepages.inf.ed.ac.uk/sgwater/math_tutorials.html)
right now. I'll wait.

Our first goal is to answer the question: how do we define the probability
of a sentence? Once we've answered this question, we'll graduate to the topic
of the course: how do we define the probability of a *pair* of sentences?

We first need to define a sample space.
Let $$\Sigma$$ be a finite set of words, and let $$\Sigma^*$$ be the set of 
strings over these words.[^1] Our sample space is $$\Sigma^*$$.
If $$w$$ is a string in $$\Sigma^*$$, let $$w_i$$ be its $$i$$th token and 
$$|w|$$ its length. Hence every $$w_i$$ is a word in $$\Sigma$$ and
every sample is a sequence of such words, $$w = w_1...w_{|w|}$$. 

We will also need to define some events.
Let $$W$$ be a random variable with domain $$\Sigma^*$$. It simply returns 
the sampled string, so that $$W(w) = w$$. The probabilitiy distribution of
interest is $$P(W)$$, but since the domain of $$W$$ is infinite, this
will be difficult to work with; we need to break it into smaller, finite
pieces. To do this, we will also define a random
variable for each position of a string drawn from the sample space. 
Since strings can be arbitrarily long, we still need an infinite number of 
these variables, but at least the domain of each variable is finite. 
Let $$W_i$$ be the random variable corresponding to the $$i$$th element of 
the string. The complete set of random variables will be $$\{W,W_0,W_1,...,W_\infty\}$$.
For every random variable $$W_i$$, let its set of possible values be
$$\Sigma \cup \{\langle START \rangle, \langle STOP\rangle\}$$, where 
$$\langle START \rangle$$ and $$\langle STOP\rangle$$ are special symbols
not in $$\Sigma$$. If we draw a string $$w$$ from the sample space, we define
our random variables as follows:

- $$W_0(w) = \langle START\rangle$$. 
- $$W_i(w) = w_i$$ if $$0<i$$ and $$i \leq$$ $$\mid{}w\mid$$.
- $$W_i(w) = \langle STOP\rangle$$ if $$i>$$ $$\mid{}w\mid$$.

**Example.** Suppose that $$w$$ is the string ''Sam loves Casey''. Then:

- $$W = (Sam, loves, Casey)$$.
- $$W_0 = \langle START\rangle$$.
- $$W_1 = Sam$$.
- $$W_2 = loves$$.
- $$W_3 = Casey$$.
- $$W_i = \langle STOP\rangle$$ for all $$i>3$$.

Under this definition, it is easy to see that for every string $$w\in\Sigma^*$$,
there is a unique assignment to $$W$$ and a unique *joint* assignment to 
the set of variables $$\{W_0,...,W_\infty\}$$. Hence for 
any sample $$w$$, $$P(W_0(w),...,W_\infty(w))=P(W=w)$$.
That means we can  model the distribution $$P(W)$$ using 
the joint distribution $$P(W_0,...,W_\infty)$$.
By the chain rule, we can rewrite this in terms of an infinite
set of conditional distributions:

$$P(W_0,...,W_\infty) = \prod_{i=0}^\infty P(W_i|W_0,...,W_{i-1})$$

**Important.** Remember that when we apply the chain rule, we can generate 
the variables in any order; the only requirement is that each new variable 
is simply conditioned on all previously generated variables. In other words,
since the chain rule is true for any permutation of the variables, this
is a modeling choice that we're making (our first). It says that each 
word is generated conditioned on
all words to its left, which seems natural for English speakers, since for 
English (and many other languages), it corresponds to the order in which 
the words are spoken or written. But not for all languages!

Notice that by the definition of the $$W_i$$'s, $$P(W_0=\langle START\rangle)$$
must be 1, and the conditional 
probability $$P(W_i=\langle STOP\rangle |W_0,...,W_{i-1})$$ must be 1 if $$W_{i-1}=\langle STOP\rangle$$.
This means that we can define the probability of a sample $$w$$ as:

$$P(W=w) = \prod_{i=1}^{|w|+1} P(W_i=w_i|W_0=w_0,...,W_{i-1}=w_{i-1})$$

...which is quite a bit nicer, since it only involves a finite number of 
distributions. But this isn't yet an implementable model; How do we  represent
the distribution $$P(w_i|w_0,...,w_{i-1})$$? There are various ways to represent distributions
over discrete objects. We'll talk about a few different strategies in this
course, but we'll start with a simple one. Let's take the distribution
$$P(W_1|W_0=\langle START\rangle)$$. Since the domain of $$W_1$$ is finite,
we can *directly* represent the distribution as a set of parameters, one
for each element of $$\Sigma \cup \{\langle STOP \rangle\}$$. In other words,
we can just write down a function with finite domain,
$$g_1:\Sigma \cup \{\langle STOP \rangle\}\rightarrow [0,1]$$, subject to the
constraint that $$\sum_{W_1 \in \Sigma \cup \{\langle STOP \rangle\}} g_1(W_1) = 1$$.
This can be implemented as a table mapping each word to a probability, 
in which case every $$g_1(u)$$ is a *parameter* of a model we can implement.

What about $$P(W_2|W_0=\langle START\rangle,W_1)$$? In this case, the 
probability is now a function of both $$W_1$$ and $$W_2$$. We could again
implement this concretely as a function with finite domain, 
$$g_2:(\Sigma \cup \{\langle STOP \rangle\})^2 \rightarrow [0,1]$$, subject to the
constraint that $$\sum_{W_2 \in \Sigma \cup \{\langle STOP \rangle\}} g_2(W_1,W_2) = 1$$
for all values of $$W_1\in\Sigma$$.

We could repeat this process for increasing numbers of words, but as you have
no doubt noticed, the domain of $$g_i$$ grows exponentially in $$i$$, and 
in the end we will have an infinite number of parameters. To keep 
things under control, we make our second major modeling choice, which is
to assume that the variable $$W_i$$ is *conditionally independent* of all
words except the $$n-1$$ variables $$W_{i-n+1},...,W_{i-1}$$. In other words,
we *approximate* the conditional distributions for each $$W_i$$:

$$P(W_i|W_0,...,W_{i-1}) \approx P(W_i|W_{i-n+1},...,W_{i-1})$$

This assumption is clearly too strong, since for any value of $$n$$, there 
are many cases where a word depends on a previous word that is more than
$$n-1$$ words in the past. Pronouns are a good example, but it's easy to
think of other ones:

* Captain Ahab, after harboring his grudge for many years, is now seeking the White _____.
* Donald Trump, after harboring his grudge for many years, is now seeking the White _____.

Under this assumption, each function $$g_i$$ has finite domain, but there are
still infinitely many of them. This is easy to fix..
If for every value of $$i$$, we now say that the 
parameter determining $$P(W_i|W_{i-n+1},...,W_{i-1})$$ is *shared* across
all values of $$i$$ (that is, that we don't have a separate $$\Sigma^n$$-sized
parameter table for every velue of $$i$$, but only a single function $$g$$ shared 
between all of them), then we can say that:

$$P(W_i|W_{i-n+1},...,W_{i-1}) = g(W_{i-n+1},...,W_i)$$

Now we have a model that you can actually implement. Given 
table $$g$$, you can now answer the question "what is the probability
of this sentence?" for any sentence defined over $$\Sigma$$. We'll talk
about how to estimate the parameters (thus populating $$g$$) in subsequent
notes.

(As you have by now guessed, for values of $$n>2$$, we can handle the first 
$$n-1$$ words of the sentence uniformly by defining random variables 
$$W_{-1},...,W_{-n+2}$$ that always take the value $$\langle START \rangle$$.)

[^1]: Do you have any objections to the idea of modeling language in
      terms of words? What about a *finite set* of words? I hope you thought
      of some objections, but hold those thoughts for now. We'll come back
      to this point later.
