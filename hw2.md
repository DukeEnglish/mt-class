---
img: voynich
img_link: http://en.wikipedia.org/wiki/Voynich_manuscript 
caption: The Voynich manuscript
title: Decoding | Homework 2
active_tab: hw2 
---

<div class="alert alert-danger">
  This assignment is over.
</div>

Decoding <span class="text-muted">| Homework 2</span>
=============================================================

Decoding is process of taking input in French:

<p class="text-center">
<em>honorables s&eacute;nateurs , que se est - il pass&eacute; ici , mardi dernier ?</em>
</p>

...And finding its best English translation under your  model:

<p class="text-center">
<em>honourable senators , what happened here last Tuesday ?</em>
</p>

To decode, we need a model of English sentences conditioned on the
French sentence. You did most of the work of creating
such a model in [Homework 1](hw1.html). In this assignment,
we will give you some French sentences and a probabilistic model consisting of
a phrase-based translation model $$p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e})$$
and an n-gram language model $$p_{\textrm{LM}}(\textbf{e})$$. __Your 
challenge is to find the most probable English translation under 
the model.__ We assume a noisy channel decomposition.

<p class="text-center">
$$\begin{align*} \textbf{e}^* & = \arg \max_{\textbf{e}} p(\textbf{e} \mid \textbf{f}) \\ & = \arg \max_{\textbf{e}} \frac{p_{\textrm{TM}}(\textbf{f} \mid \textbf{e}) \times p_{\textrm{LM}}(\textbf{e})}{p(\textbf{f})} \\ &= \arg \max_{\textbf{e}} p_{\textrm{TM}}(\textbf{f} \mid \textbf{e}) \times p_{\textrm{LM}}(\textbf{e}) \\ &= \arg \max_{\textbf{e}} \sum_{\textbf{a}} p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e}) \times p_{\textrm{LM}}(\textbf{e}) \end{align*}$$
</p>

Getting Started
---------------

If you have a clone of the repository from 
[homework 1](hw1.html), you can update it 
from your working directory:

    git pull origin master

Alternatively, get a fresh copy:

    git clone https://github.com/alopez/infr11062.git

Under the new `decode` directory, you now have simple decoder
and some data files. Take a look at the input sentences:

    cat data/input

Let's translate them!

    python decode > output

This creates the file `output` with translations of `data/input`.
Take a look:

    cat output

You can probably get the gist of the Canadian senator's complaint 
from this translation, but it isn't very readable. There are a 
couple of possible explanations for this:

1. We used a model of $$p(\textbf{e} \mid \textbf{f})$$ that gives high probability to bad translations. This is called *model error*.
1. Our model is ok but our decoder fails to find $$\arg \max_{\textbf{e}} p(\textbf{e} \mid \textbf{f})$$. This is called *search error*.

How can we tell which problem is the culprit? One way
would be to eliminate problem 2, which is a purely computational 
problem. Let's focus on this.

You can compute a value that is proportional to
$$p(\textbf{e} \mid \textbf{f})$$  using `compute-model-score`.

    python compute-model-score < output

This command sums over all possible ways that the model could have 
generated the English from the French, including translations
that permute the phrases. That is, it exactly computes:

<p class="text-center">
$$\sum_{\textbf{a}} p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e}) \times p_{\textrm{LM}}(\textbf{e})$$
</p>

This value is proportional to $$p(\textbf{e} \mid \textbf{f})$$ up to
the constant $$\frac{1}{p(\textbf{f})}$$. The sum is
intractable in general, but it turns out that we can solve these
particular sums in a few minutes. It is easier to do this 
than it is to find the optimal translation. 

I highly recommend that you look at `compute-model-score`.
You may get some hints about how to do the assignment! It contains
some useful utility functions, for example to add probabilities in
logarithmic space, and manipulate bitmaps.

The decoder generates the most probable translations 
that it can find, but it uses three common approximations that
might cause search error. 

First, it seeks the _Viterbi approximation_ to the most probable 
translation. Instead of computing the intractable sum over
all alignments for each sentence, we simply find the best 
single alignment and use its translation.

<p class="text-center">
$$\begin{align*} \textbf{e}^* &= \arg \max_{\textbf{e}} \sum_{\textbf{a}} p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e}) \times p_{\textrm{LM}}(\textbf{e}) \\ &\approx \arg \max_{\textbf{e}} \max_{\textbf{a}} p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e}) \times p_{\textrm{LM}}(\textbf{e}) \end{align*}$$
</p>

This approximation makes it possible to use dynamic programming
for search. This decoder uses a simple dynamic program that,
for each input word position $$i$$, keeps a _stack_ (actually a priority queue) of the $$s$$
most probable translations for the first $$i$$ words. The
main dynamic programming loop begins at 
[line 37](https://github.com/alopez/infr11062/blob/master/decoder/decode#L37).

Second, it translates French phrases into English without
changing their order. So, it only reorders words  if 
the reordering has been memorized as a phrase pair.
For example, in the first sentence, we see that
_<span class="text text-primary">mardi</span>
<span class="text text-danger">dernier</span>_
is correctly translated as
_<span class="text text-danger">last</span>
<span class="text text-primary">Tuesday</span>_.
If we consult `data/tm`, we will find that the model
has memorized the phrase
pair `mardi dernier ||| last Tuesday`. 

    grep "mardi dernier" data/tm

But in the second sentence, we see that 
_<span class="text text-danger">Comit&eacute; de</span> 
<span class="text text-primary">s&eacute;lection</span>_
is translated as 
_<span class="text text-danger">committee</span> 
<span class="text text-primary">selection</span>_,
rather than the more fluent
_<span class="text text-primary">selection</span>
<span class="text text-danger">committee</span>_. 
To show that this is a search problem rather than
a modeling problem, we can generate the latter output
by hand and check that the model really prefers it.

    head -2 data/input | tail -1 > example
    python decode -i example | python compute-model-score -i example
    echo a selection committee was achievement . | python compute-model-score -i example

The scores are reported as log-probabilities, and higher
scores (with lower absolute value) are better. We 
see that the model prefers
_<span class="text text-primary">selection</span>
<span class="text text-danger">committee</span>_, 
but the decoder does not consider this word order
since it has never memorized this phrase pair.

Finally, our decoder uses strict pruning. As it consumes the input
sentence from left to right, it keeps only the most probable 
output up to that point. You can vary the number of number
of outputs kept at each point in the translation using the
`-s` parameter. See how this affects the resulting model score
and the actual translations.

    python decode > out 
    python compute-model-score < out
    python decode -s 10000 > out10000
    python compute-model-score < out10000
    vimdiff out out10000
    
(Type `:q:q` to exit vimdiff).

**Question 1: Try several different values for the stack 
size parameter `-s`. How do changes in this value affect 
the resulting log-probabilities? How do they affect the 
resulting translations?** 


Baseline: Local Reordering
--------------------------

Your task is to __find the most probable English translation__.
Our model assumes that any segmentation of the French sentence into
phrases followed by a one-for-one substitution and permutation of
those phrases is a valid translation. We make the 
simplifying assumption that segmentation and ordering
probabilities are uniform across all sentences, hence constant.
This means that $$p(\textbf{e},\textbf{a} \mid \textbf{f})$$ is proportional to
the product of the n-gram probabilities in $$p_{\textrm{LM}}(\textbf{e})$$
and the phrase translation probabilities in $$p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e})$$. To 
avoid numerical underflow we work in logspace, seeking
$$\arg \max_{\textbf{e}} \max_{\textbf{a}} \log p_{\textrm{TM}}(\textbf{f},\textbf{a} \mid \textbf{e}) + \log p_{\textrm{LM}}(\textbf{e})$$. The
baseline decoder works with log probabilities, so you can
simply follow what it does. 

To earn 8 marks, you must answer questions 1 and 2 and 
implement a beam-search 
decoder like the one we have given you 
that is also capable of _swapping the translations of adjacent phrases_. That 
is, your decoder should be able to produce the correct translation
for the _<span class="text text-primary">selection</span>
<span class="text text-danger">committee</span>_ example. It
needn't to be able to permute sequences of three or more phrases.

**Question 2: Answer the questions you answered in Question 1 for
your new decoder.**

The Challenge
-------------

Implementing a decoder that swaps adjacent phrases will earn you 8 marks out of
10. But swapping adjacent phrases will not get you anywhere close to the most
probable translation according to this model. To get 
full credit, you __must__ additionally experiment with another decoding algorithm.
Any permutation of phrases is a valid translation, so you might attempt to
search over all or some part of this larger space. This search is
NP-Hard, so it will not be easy. You 
can trade efficiency for search effectiveness
by implementing histogram pruning or threshold pruning, or by using 
reordering limits as described in the textbook (Chapter 6). Or, you might
consider implementing other approaches to solving this combinatorial
optimization problem:

* [Implement a greedy decoder](http://www.iro.umontreal.ca/~felipe/bib2webV0.81/cv/papers/paper-tmi-2007.pdf).
* [Use chart parsing to search over many permutations in polynomial time](http://aclweb.org/anthology/C/C04/C04-1030.pdf).
* [Use a traveling salesman problem (TSP) solver](http://aclweb.org/anthology-new/P/P09/P09-1038.pdf).
* [Use finite-state algorithms](http://mi.eng.cam.ac.uk/~wjb31/ppubs/ttmjnle.pdf).
* [Use Lagrangian relaxation](http://aclweb.org/anthology//D/D13/D13-1022.pdf).
* [Use integer linear programming](http://aclweb.org/anthology-new/N/N09/N09-2002.pdf).
* [Use A* search](http://aclweb.org/anthology-new/W/W01/W01-1408.pdf).

These methods all attempt to approximate or solve the Viterbi approximation to decoding.
You can also try to approximate $$p(\textbf{e} \mid \textbf{f})$$ directly.

* [Use variational algorithms](http://aclweb.org/anthology//P/P09/P09-1067.pdf).
* [Use Markov chain Monte Carlo algorithms](http://aclweb.org/anthology//W/W09/W09-1114.pdf).

But the sky's the limit! There are many ways to decode.
You can try anything you want as long as you follow the ground rules:

Ground Rules
------------

* You may work in independently or pairs, under these 
  conditions: 
    1. You must let me know about the collaboration in advance
       by emailing me and copying your collaborator on the email.
    1. You agree that everyone in the group will receive the same grade on the assignment. 
    1. You cannot undo a collaboration once you've informed me.
       I encourage collaboration since explaining things to someone else
       often helps you understand them better yourself. But I will not adjudicate Rashomon-style 
       stories about who did or did not contribute.
* You must turn in four things using the `submit mt 2 <files>` command on dice:
    1. A file containing your answers to Questions 1 and 2.
    1. A clear, mathematical description of your algorithm and its motivation
       written in scientific style. This needn't be long, but it should be
       clear enough that one of your fellow students could re-implement it.
    1. Your translations of the input sentences.
    1. Your code. 
       You are free to extend the code we provide or roll your own in whatever
       langugage you like, but the code should be self-contained, 
       self-documenting, and easy to use. 

Your written work may be submitted in PDF, text, or markdown (like 
[this website](https://github.com/alopez/mt-class)). I will not accept
any other format. If you write the file in a different format, please
convert it to one of the above prior to submission.


### Acknowledgements

This assignment was developed in collaboration with
[Chris Callison-Burch](http://www.cis.upenn.edu/~ccb/),
[Chris Dyer](http://www.cs.cmu.edu/~cdyer), and
[Matt Post](http://cs.jhu.edu/~post/).
