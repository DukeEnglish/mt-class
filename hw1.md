---
img: rosetta
img_link: http://en.wikipedia.org/wiki/Rosetta_Stone 
caption: Rosetta stone (image by flickr user calotype46)
title: Alignment | Homework 1
active_tab: homework
---

<div class="alert alert-danger">
  This page is under revision. 
  Wait until this message disappears before starting the assignment.
</div>

Alignment <span class="text-muted">| Homework 1</span>
=============================================================

Word alignment is a fundamental problem in machine translation. We start with
a large _parallel text_ of translated sentences. For example, our
parallel text might contain the following translation:

    Das Parlament erhebt sich zu einer Schweigeminute .  
    The House rose and observed a minute ' s silence .

Your task is to align the words of the sentences. Given the sentence pair
above, you would ideally produce an alignment like this:

| German            | | English                |
|-------------------|-|------------------------|
| 0. Das            | | 0. The                 |
| 1. Parlament      | | 1. House               |
| 2-3. erhebt sich  | | 2. rose                |
| 4. zu             | |                        |
|                   | | 3. and                 |
|                   | | 4. observed            |
| 5. einer          | | 5. a                   |
| 6. Schweigeminute | | 6-8. minute 's silence |
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

    python align.py -n 200 > dice.out

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

    python score-alignments.py < dice.out

Maybe these alignments are terrible since we used very little data. Try 
using ten times as much data:

    python align.py -n 2000 | python score-alignments.py -n 0

__Question 1. What happens when you use different amounts of data? 
What happens when you experiment with the threshold, controlled by the 
`-t` option? How are precision, recall, and AER affected? 
Study the grids produced by the aligner under different settings 
and those produced by the human annotators. What do you observe? 
How do they differ?__

The challenge
-------------

Your challenge is to improve the alignment error rate as much as possible. 
It shouldn’t be hard: while our intuition about the coocurrence of words 
makes our aligner better than chance, it is still very crude. Among other 
problems, you may have noticed that some words tend to spuriously align to 
many other words in the translation. Perhaps you could add some heuristics 
to the Dice aligner to solve this. We will use probability, which provides 
us with powerful tools to reason about data. Specifically, we will use IBM 
Model 1, which you must implement, examine, and extend.


