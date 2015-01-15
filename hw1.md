---
img: rosetta
img_link: http://en.wikipedia.org/wiki/Rosetta_Stone 
caption: Rosetta stone (image by flickr user calotype46)
title: Alignment | Homework 1
active_tab: homework
---

Alignment <span class="text-muted">| Homework 1</span>
=============================================================

Word alignment is a fundamental problem in machine translation. We start with
a large _parallel text_ of translated sentences. For example, our
parallel text might contain the following translation:

    Das Parlament erhebt sich zu einer Schweigeminute .  
    The House rose and observed a minute ' s silence .

Your task is to align the words of the sentences. Given the sentence pair
above, you would ideally produce an alignment containing the following
word pairs:

| German            | | English                |
|-------------------|-|------------------------|
| 0. Das            | | 0. The                 |
| 1. Parlament      | | 1. House               |
| 2. erhebt         | | 2. rose                |
| 3. sich           | | 2. rose                |
| 5. einer          | | 5. a                   |
| 6. Schweigeminute | | 6. minute              |
| 6. Schweigeminute | | 7. 's                  |
| 6. Schweigeminute | | 8. silence             |
| 7. .              | | 9. .                   |

\\
Some words, like _observed_, are not aligned, while other words, like _rose_, 
are multiply aligned. In many examples, the aligned words will not even be 
in the same order. This makes word alignment challenging.

Bilingual speakers do not always completely agree about the correct 
alignment of a translation. In this example, some would also align _zu_ 
to _rose_. However, in most cases alignment is uncontroversial. 

Getting started
---------------

You must have `git` and `python2.7`, both available on dice. To get the code
and data, run:

    git clone https://github.com/alopez/infr11062.git

In the new `infr11062/aligner` directory you will find a python program 
called `align.py`, which contains a complete but very simple alignment 
algorithm. Intuitively, English and German words that frequently appear 
together are likely to be translations of each other. So for every word, 
our aligner computes the set of sentences that the word appears in. Then 
for every pair of English and German words, it computes the similarity of 
their respective sets, and it aligns those pairs with similarity higher than 
a threshold. The aligner computes sentence similarity with 
[Dice’s coefficient](http://en.wikipedia.org/wiki/Dice's_coefficient/). 
Given the co-occurence count
$$C(e,f)$$ of words $$e$$ and $$f$$ in the parallel corpus, Dice's
coefficient for each pair of words $$e_i, f_j$$ is:

<p>$$ \delta(i,j) = \frac{2 \times C(e_i, f_j)}{C(e_i) +C(f_j)} $$</p>

The aligner will align any word pair $$e_i, f_j$$ with a
coefficient $$\delta(i,j)$$ over 0.5. Run it on 200 sentences:

    python align.py -n 200 > dice200.out

For each sentence, it generates pairs of numbers on a line, one per 
alignment link. For our example alignment above, this would be:

    0-0 1-1 2-2 3-2 5-5 6-6 6-7 6-8 7-9

We want to know how good these alignments are. To do this, we will compare 
our automatic alignment with human alignments of the first 150 sentences. 
Our measures will include _precision_, the percentage of guessed alignments 
that are correct; _recall_, the percentage of human alignments that are 
guessed; and [_alignment error rate_](http://aclweb.org/anthology/P/P00/P00-1056.pdf)
(AER), which combines both measures. For precision and recall, 
higher is better, while for AER, lower is better. To compute them, run:

    python score-alignments.py < dice200.out

The alignments are terrible! Perhaps this is because  we used very 
little data. Try using ten times as much data:

    python align.py -n 2000 | python score-alignments.py -n 0

__Question 1. 
What happens when you use different amounts of data? 
What happens when you experiment with the threshold, controlled by the `-t` option? How are precision, recall, and AER affected? 
Study the grids produced by the aligner under different settings and those produced by the human annotators. What patterns do you observe? How do they differ?__

Baseline: IBM Model 1
---------------------

Your challenge is to improve the alignment error rate as much as possible. 
It shouldn’t be hard: while our intuition about the coocurrence of words 
makes our aligner better than chance, it is still very crude. Among other 
problems, you may have noticed that some words tend to spuriously align to 
many other words in the translation. Perhaps you could add some heuristics 
to the Dice aligner to solve this. We will use probability, which provides 
us with powerful tools to reason about data. Specifically, we will use IBM 
Model 1, which you must implement, examine, and extend.
IBM Model 1 is a *word-to-word* or *lexical* translation model. 
It has only one set of parameters: a
conditional probability $$t(f \mid e)$$ for every foreign word 
$$f$$ and English word $$e$$ that cooccur in at least one sentence. 

Pick a translation pair from the data $${\cal D}$$. Let the German
sentence be $$\mathbf{f} = (f_1, \ldots, f_I)$$ and the English
sentence $$\mathbf{e} = (e_1, \ldots, e_J)$$. Now we make a big 
assumption: that this pair of sentences exists
because someone translated the English into German following a simple
procedure. First, the translator chose the length of the German sentence based
on the length of the English sentence. Second, for each position in
the German sentence, the translator chose a word $$e$$ at random from
the English sentence, and then translated it to foreign word $$f$$
according to the translation probability $$t(f \mid e)$$.

This means that we can represent an alignment $$\textbf{a}$$
of the foreign words by: $$\textbf{a} = (a_1, \ldots, a_I)$$. There
is one random variable $$a_i$$ for each foreign word $$f_i$$, and
each $$a_i$$ must take value in the range $$1,...,J$$, 
indicating the chosen English word.[^1] Hence we obtain the following
model:

[^1]: In some versions of this story, $$a_i$$ is allowed to be $$0$$, which means the translator plucked it from thin air. We call this *null* alignment.

<p>$$ \Pr(\mathbf{f}, \textbf{a} \mid \mathbf{e}) = \ell(I \mid J) \prod_{i=1}^I
c(a_i \mid J) \cdot t(f_i \mid e_{a_i})$$</p>

Here, parameter $$\ell(I \mid J)$$ is the probability of producing
a German sentence of length $$I$$ from an English sentence of length
$$J$$, and $$c(a_i=j\mid J)$$ is the probability of choosing the $$j$$th
English word. We have assumed that English words are chosen at random 
from a uniform distribution, so $$c(a_i=j \mid J) = \frac{1}{J}$$.
Take for example a three-word German sentence ($$I = 3$$) aligned
to a three-word English sentence ($$J = 3$$).  If the alignment
$$\textbf{a} = (1,3,2)$$, that is, $$a_1 = 1$$, $$a_2 = 3$$, and
$$a_3 = 2$$ we can write the probability of this sentence pair as:

<p>$$\Pr(\mathbf{f}, \textbf{a} \mid \mathbf{e}) = \ell(3\mid 3) 
\cdot \left(\frac{1}{3}\right)^3 t(f_1 \mid e_1)
\cdot t(f_2 \mid e_3) \cdot t(f_3 \mid e_2)$$</p>

Let's focus on the parameters $$t(f_3 \mid e_2)$$. Where do we get 
them? We have some data that we assume was generated by our model, and the
probability of this data under the model is a function of the parameters
(called the _likelihood_). So a reasonable solution is to choose 
parameters that maximize this function -- this is the _maximum likelihood_
estimate. We want:

<p>$$ \arg\max_{t} \sum_s \log \Pr(\mathbf{f}^{(s)},\mathbf{a}^{(s)}\mid
\mathbf{e}^{(s)}, t) $$</p>

You will notice a small problem with this scheme: we weren't given the
alignments, which are needed to compute the maximum likelihood estimate.
The alignment $$\mathbf{a}$$ is a _latent variable_. But we can still
maximize the data likelihood -- we just maximize the probability of the
_observed_ data by summing over all possible values of the latent variables.

<p>$$
\begin{eqnarray*}
\Pr(\mathbf{f} \mid \mathbf{e}, t) & = & \sum_{\textbf{a}} \Pr(\mathbf{f}, \textbf{a} \mid \mathbf{e}, t) \\
& = & \sum_{a_1=1}^J \cdots \sum_{a_I=1}^J  \ell(I|J) \prod_{i=1}^I c(a_i|J) \cdot t(f_i \mid e_{a_i}) \\
&& \textrm{(this computes all possible alignments)} \\
& = & C \prod_{i=1}^I \sum_{j=1}^J t(f_i \mid e_j) \\
&& \textrm{(since the } \ell \textrm{ and } c \textrm{ terms don't depend on the alignment, we lump them into a constant } C)
\end{eqnarray*}
$$</p>

To perform this optimization we will use the _Expectation-Maximization_
(EM) algorithm. A full explanation of this algorithm is lenghy, 
but I expect you to understand what it does and how it works at least at
the level described in your textbook and in 
[this note](http://edin.ac/1DH1C78). In particular,
you should be able to derive EM update equations for simple models, though
I won't ask you to prove anything about it.

In order to estimate the parameters $$t(\cdot \mid \cdot)$$
we start with an initial estimate $$t_0$$ and modify it iteratively
to get $$t_1, t_2, \ldots$$. The parameter updates are derived
for each German word $$f_i$$ and English word $$e_j$$ as follows:

<p>$$t_k(f_i \mid e_j) = \sum_{s=1}^N \sum_{(f_i, e_j) \in (\textbf{f}^{(s)}, \textbf{e}^{(s)})} \frac{ \mathbb{E}_{t_{k-1}}(f_i, e_j, \textbf{f}^{(s)}, \textbf{e}^{(s)}) }{\mathbb{E}_{t_{k-1}} (e_j, \textbf{f}^{(s)}, \textbf{e}^{(s)}) }$$</p>

These counts are *expected counts* over all possible alignments,
and each alignment has a probability computed using $$t_{k-1}$$.

Once we have trained an alignment model, we want to find
the most probable  alignment for a given translation pair.

<p>$$ 
\hat{\textbf{a}} = \arg\max_{\textbf{a}} \Pr(\textbf{a} \mid \textbf{e}, \textbf{f})
$$</p>

The best alignment to a target sentence in our simple baseline model
is obtained by finding the best alignment for each word in
the source sentence independently of the other words. For each
foreign word $$f_i$$ in the source sentence the best alignment is
given by:

<p>$$ 
\hat{a_i} = \arg\max_{a_i} t(f_i \mid e_{a_i}) 
$$</p>

If your Model 1 implementation is correct, it will give you
much better alignments than the Dice aligner. They are still far
from perfect, though. 

I strongly recommend that you work through the derivation of the full
algorithm from first principles, described in much more detail in 
[this note](http://edin.ac/1DH1C78). This is to practice writing
down a model, deriving simple estimators from the model, and implementing
those estimators as code. On the exam I am very likely to ask you to
derive a new model. I will not ask you to recite the specific
form of Model 1.

You are required to implement IBM Model 1. This requires
a very modest amount of code: my implementation is one line shorter than the 
Dice aligner! You are not required to use the python baseline (though that
will be easiest). Use whatever language you prefer.

__Question 2: The theory behind expectation maximization guarantees that
the data likelihood increases after each iteration. Plot the data likelihood
(you will need to do this in logspace to avoid numerical underflow) for
some data over several iterations. How does it change? Does it appear to
converge? Now compare this with the alignment accuracy of the model after
each iteration. How do data likelihood and accuracy relate?__

__Question 3: Choose two English words from the data: a very frequent word,
and an infrequent word. Now look at the learned translation distributions
for these words. What do you observe? How do the shapes of these distributions
affect the resulting alignments?__

__Question 4: Choose two English words from the data that are morphological
variants of each other (e.g. _house_ and _houses_). How similar or dissimilar
are they? Think about how IBM Model 1 treats these words. How does this
affect the resulting distributions? Does this seem reasonable to you?__

__Question 5: Observe the human alignments of the data. Do the English
words appear to be chosen at random? Or, do there seem to be any underlying
regularities in the data? How does this affect the accuracy of the
results you get with IBM Model 1?__


The Challenge
-------------

Developing a Model 1 aligner and answering questions 1 through 5 
will earn you 8 marks out of 10. But if you answered the questions in full
you will have noticed some serious problems with Model 1. By thinking
carefully about these problems, you should be able to develop better 
probabilistic models of alignment. How well can you do?[^3]
To get full credit you **must** experiment with at least one 
well-motivated extension of Model 1 and report the results. Your report
should concisely state the problem you tried to solve, how you modeled it, and
the results you obtained on the data. You are not required to do better than 
Model 1, only to hypothesize and experiment in good faith. Failed experiments
are a necessary component of good problem-solving.

[^3]: The best AER on this task is at least .125 according to this [comparison of different alignment models](http://aclweb.org/anthology/P/P04/P04-1023.pdf).

Here are some ideas:

* Implement [a model that prefers to align words close to the diagonal](http://aclweb.org/anthology/N/N13/N13-1073.pdf).
* Implement an [HMM alignment model](http://aclweb.org/anthology-new/C/C96/C96-2141.pdf).
* Implement [a morphologically-aware alignment model](http://aclweb.org/anthology/N/N13/N13-1140.pdf).
* [Use *maximum a posteriori* inference under a Bayesian prior](http://aclweb.org/anthology/P/P11/P11-2032.pdf).
* Train a German-English model and an English-German model and [combine their predictions](http://aclweb.org/anthology-new/N/N06/N06-1014.pdf).
* Train a [supervised discriminative alignment model](http://aclweb.org/anthology-new/P/P06/P06-1009.pdf) on the annotated development set.
* Train an [unsupervised discriminative alignment model](http://aclweb.org/anthology-new/P/P11/P11-1042.pdf).
* Seek out additional [inspiration](http://scholar.google.com/scholar?q=word+alignment).

But the sky's the limit! You are welcome to design your own solution
as long as you follow the ground rules.

Ground Rules
------------

* You may work in independently or pairs, under these 
  conditions: 
    1. You must let me know about the collaboration in advance.
    1. You agree that everyone in the group will receive the same grade on the assignment. 
    1. You cannot undo a collaboration once you've informed me.
       I encourage collaboration, but I will not adjudicate Rashomon-style 
       stories about who did or did not contribute.
* You must turn in four things using the `submit mt 1 <files>` command on dice:
    1. A file containing your answers to Questions 1 through 5.
    1. A clear, mathematical description of your algorithm and its motivation
       written in scientific style. This needn't be long, but it should be
       clear enough that one of your fellow students could re-implement it.
    1. An alignment of the first 300 sentences of the data using your final model.
    1. Your code. 
       You are free to extend the code we provide or roll your own in whatever
       langugage you like, but the code should be self-contained, 
       self-documenting, and easy to use. 

Your written work may be submitted in PDF, text, or markdown (like 
[this website](https://github.com/alopez/mt-class). I will not accept
any other format. If you write the file in a different format, please
convert it to one of the above prior to submission.

### Acknowledgements

[John DeNero](http://www.denero.org/),
[Chris Dyer](http://www.cs.cmu.edu/~cdyer),
[Philipp Koehn](http://homepages.inf.ed.ac.uk/pkoehn/),
[Matt Post](http://cs.jhu.edu/~post/), and
[Anoop Sarkar](http://www.cs.sfu.ca/~anoop/)
contributed to the development of this 
assignment.


### Footnotes 
