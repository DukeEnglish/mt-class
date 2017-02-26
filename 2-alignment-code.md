---
layout: common
title: From Maths to Code
---

Word Alignment Models from Maths to Code
========================================

Several of the questions I've gotten have been about computing marginals for
EM and converting it to code. Below are some raw notes; please let me know 
if anything is unclear.

But first a note on scope: this note doesn't go through the general 
derivation or justification of the EM algorithm itself. In fact, I will not 
ask you about this, since this is an applied course on machine translation. 
But it's very useful, so if you want the full details, you may want to read 
[this note](http://www.seanborman.com/publications/EM_algorithm.pdf). On the
other hand, I **do** want you to understand what it means that the algorithm
optimizes parameters of a model: that's a general idea that we will return
to over and over again.

(Some of you are confused about notation. Unfortunately, there is no such 
thing as a unified notation that everyone uses, so you'll need to rely on 
every document you read to define terms. I will try to do that consistently,
but if I leave out this detail, or write something that is otherwise 
inconsistent, just remind me. Also: it takes some practice to get 
comfortable reading this notation. I will only promise that with consistent 
practice, it gets easier. In fact this is true of every topic in this 
course, and everything else under the sun, too.)

Let $$e = e_1 ... e_J$$ be an English sentence of $$J$$ words and 
$$f = f_1 ... f_I$$ be a French sentence of $$I$$ words. Define 
the conditional probability $$p(f\mid e)$$ in terms of alignment 
$$a = a_1 ... a_I$$, where the domain of $$a_i$$ is $$\{1,...,J\}$$ for 
each $$i\in \{1,...,I\}$$. By this definition the domain of $$a$$ is 
$$\{1,...,J\}^I$$. This notation means: the set of all vectors of 
$$I$$ elements where each element is in the set $$\{1,...,J\}$$. 
(More generally, for an arbitrary set $$\mathcal{X}$$, $$\mathcal{X}^m$$ 
is the set of all vectors of $$m$$ elements, where each element is in 
$$\mathcal{X}$$).

Define IBM Model 1 as: 

$$p(f,a|e) = p(I|J) \prod_{i=1}^I p(a_i|J) \times p(f_i|e_{a_i})$$

To run EM we must compute expectations. This means we want to answer the 
following question for every setting of $$i$$ and $$j$$: what is the value 
of $$p(a_i=j\mid f,e)$$? By the inference rules that we derived from the 
axioms of probability, this is just:

$$p(a_i=j|f,e) = \frac{p(a_i=j,f|e)}{p(f|e)}$$

Now we have two new questions: What the value of $$p(f\mid{}e)$$? And what 
is the value of $$p(a_i=j,f\mid{}e)$$? Let's deal with the first question.

To obtain $$p(f\mid{}e)$$, marginalize over $$a$$. Since we defined the 
range of $$a$$ as $$\{1,...,J^I$$ above we can write it this way:

$$p(f|e) = \sum_{a\in [1,...,J]^I} p(I|J) \prod_{i=1}^I p(a_i|J) \times p(f_i|e_{a_i})$$

How would you implement this? The expression above literally says that you 
should enumerate all possible alignments, compute the probability of each, 
and sum the results. It's instructive to see what this looks like on an 
actual example. In fact, whenever you're faced with a problem like this, 
working out a small example on paper is always a good idea; it often makes
clear what you do and don't understand.

Suppose that you have a French sentence *le maison bleu* and English sentence 
*the blue house*. Then you could write a nested loop (this is python-ish
pseudocode, not actual python):

    sum = 0
    for a[1] from 1 to 3:
        for a[2] from 1 to 3:
            for a[3] from 1 to 3: 
                p(f,a[1],a[2],a[3]|e) = 1
                for i from 1 to I:
                    p(f,a[1],a[2],a[3]|e) *= p(a[i]|J) * p(f[i]|e[a[i]])
                sum += p(f,a[1],a[2],a[3]|e)
    sum *= p(I|J) # if you must; but this is constant, so doesn't matter

In general, that would require $$\mathcal{O}(J^II)$$ time, though 
as you can see from inspection of the nested loops. There are $$3^3=27$$ 
alignments just for this example. Let's look at the structure of the 
calculation. To make it concise, we'll use the following substitutions for 
this example:

$$p(a_1=1) \times p(f_1|e_1) = A$$

$$p(a_1=2) \times p(f_1|e_2) = B$$

$$p(a_1=3) \times p(f_1|e_3) = C$$

$$p(a_2=1) \times p(f_2|e_1) = D$$

$$p(a_2=2) \times p(f_2|e_2) = E$$

$$p(a_2=3) \times p(f_2|e_3) = F$$

$$p(a_3=1) \times p(f_3|e_1) = G$$

$$p(a_3=2) \times p(f_3|e_2) = H$$

$$p(a_3=3) \times p(f_3|e_3) = K$$

If it helps to visualize, we can associate each of these values with the cells of an alignment grid:

$$\begin{array}{rccc}

& \textrm{le} & \textrm{maison} & \textrm{bleu}\\

\textrm{the} & A & D & G\\

\textrm{bleu} & B & E & H \\

\textrm{house} & C & F & K\\

\end{array}
$$

According to the above definition, every allowable alignment requires a 
choice of exactly one cell in each column of this grid, and its probability 
is obtained by multiplying $$p(I\mid{}J)$$ by the product of the values in those 
cells. Hence aligning *le* to *the*, *maison* to house, and *bleu* to *blue*
is equivalent to the alignment vector $$a = \langle 1, 3, 2\rangle$$ and 
its probability is $$p(I\mid{}J) \times AFH$$. To get the probability 
$$p(f\mid{}e)$$ we compute the sum using the loop above. Literally:

$$p(f|e) = p(I|J) \times \left(\begin{array}{c} ADG + ADH + ADK + AEG + AEH + AEK + AFG + AFH + AFK + \\

BDG + BDH + BDK + BEG + BEH + BEK + BFG + BFH + BFK + \\

CDG + CDH + CDK + CEG + CEH + CEK + CFG + CFH + CFK ~~\end{array}\right)
$$

Again, this is what the mathematical expression implies when we code it directly. But the obvious pattern in this computation allows us to simplify:

$$\begin{align} p(f|e) & = p(I|J) \times (A+B+C) \times \left(\begin{array}{c} DG + DH + DK +\\

EG + EH + EK + \\

FG + FH + FK~~\end{array}\right)\\

& = p(I|J) \times (A+B+C) \times (D+E+F) \times (G+H+K)\end{align}
$$

It is exactly this observation that we exploit when we use the distributive property to write:

$$p(f|e) = p(I|J) \prod_{i=1}^I \sum_{j=1}^J p(a_i=j|J) \times p(f_i|e_j)$$

We can convert this directly into code by making the sums and products into loops. If we do so, we get an efficient algorithm that runs in $$\mathcal{O}(IJ)$$ time:

    sum = p(I|J)
    for i from 1 to I:
        inner_sum = 0
        for j from 1 to J:
            inner_sum += p(a[i]=j|J) * p(f[i]|e[j])$$
        sum *= inner_sum
 
We can use similar reasoning to get $$p(a_i=j,f\mid{}e)$$. In the above 
example, for instance, $$p(a_1=1,f\mid{}e) = p(I\mid{}J) \times A \times (D+E+F) \times (G+H+K)$$. 
This might make it clearer why most of the terms in the numerator and 
denominator cancel when we want to compute posterior probabilities for the 
EM algorithm, e.g.:

$$p(a_1=1|f,e) = \frac{p(a_1=1,f|e)}{p(f|e)} = \frac{p(I|J) \times A \times (D+E+F) \times (G+H+K)}{p(I|J) \times (A+B+C) \times (D+E+F) \times (G+H+K)} = \frac{A}{A+B+C}$$

Importantly, this is a property of the particular independence assumptions 
in our model. That this is so deliberate; the inventors of this 
model baked them in on purpose to get efficient inference! If we use 
different assumptions (e.g. the first-order condition of a hidden Markov 
model, or the one-to-one restriction that we discussed in class), then the 
products do not factor (or decompose) so conveniently. But we get a more 
expressive model that might be (and in fact usually is) more accurate. 
These are typical tradeoffs in data modeling.

To get a taste of this, let's look more closely at an important extension 
of IBM Model 1, the HMM. It makes a single change the alignment 
distribution, in order to encode the intuition that the alignments of 
adjacent words in $$f$$ should influence each other:

$$p(f,a|e) = p(I|J) \prod_{i=1}^{I} p(a_i|a_{i-1},J) \times p(f_i|e_{a_i})$$

Again we must answer the question: what is the value of $$p(a_i=j,f\mid{}e)$$? 
(We also need to know the value of $$p(f|e)$$, but will address that later). 
For an HMM, the key to answering it is to look at how the assignment 
$$a_i=j$$ interacts with the data:

1. With words $$f_1,...,f_{i-1}$$ only through the term $$p(a_i\mid{}a_{i-1},e)$$.
1. With word $$f_i$$ through the term $$p(f_i\mid{}a_i,e)$$.
1. With words $$f_{i+1},...,f_{I}$$ only through the term $$p(a_{i+1}\mid{}a_i,e)$$.

Each of these interactions gives us a term:

1. $$\sum_{j'} p(a_{i-1}=j',f_1,...,f_{i-1}\mid{}e) \times p(a_i=j\mid{}a_{i-1}=j',e)$$. Notice that this is just $$p(a_i=j,f_1,...,f_{i-1}\mid{}e)$$.
1. $$p(f_i\mid{}a_i=j,e)$$.
1. $$\sum_{j'} p(a_{i+1}=j',f_{i+1},...,f_I\mid{}e) \times p(a_{i+1}=j'\mid{}a_i=j,e)$$. Notice that this is just $$p(f_{i+1},...,f_I\mid{}a_i=j,e)$$.

This is really just an application of the chain rule, which becomes more 
obvious when we multiply them together:

$$p(a_i=j,f_1,...,f_{i-1}\mid{}e) \times p(f_i\mid{}a_i=j,e) \times p(f_{i+1},...,f_I\mid{}a_i=j,e) = p(a_i=j,f\mid{}e)$$

As a notational convenience, let $$\alpha(i,j) = p(a_i=j,f_1,...,f_{i-1}\mid{}e)$$. 
In words, $$\alpha(i,j)$$ is just the probability that $$a_i=j$$, 
given $$e$$ and the first $$i-1$$ words of $$f$$. How do we compute this 
quantity? It is not a simple term: $$\alpha(i,j) = \sum_{j'} p(a_{i-1}=j',f_1,...,f_{i-1}\mid{}e) \times p(a_i=j\mid{}a_{i-1}=j',e)$$. 
In particular, $$p(a_{i-1}=j',f_1,...,f_{i-1}\mid{}e)$$ looks like a complicated 
joint probability. But, we can apply the same reasoning that we used above 
to separate it into two terms:

$$\sum_{j''} p(a_{i-2}=j'',f_1,...,f_{i-2}\mid{}e) \times p(a_{i-1}=j'\mid{}a_{i-2}=j'',e)$$. Notice that this is just $$p(a_{i-1}=j',f_1,...,f_{i-2}\mid{}e)$$.

$$p(f_{i-1}\mid{}a_{i-1}=j',e)$$

But notice that the first term is just $$\alpha(i-1,j')$$. This gives us 
the recursive decomposition $$\alpha(i,j) = \sum_{j'} \alpha(i-1,j') \times p(f_{i-1}\mid{}a_{i-1}=j',e) \times p(a_i=j\mid{}a_{i-1}=j',e)$$. 
If you've seen HMMs before you will probably recognize this as the forward 
probability of being in state $$j$$ at position $$i$$. As you might guess, 
there's an analogous backward probability $$\beta(i,j)$$ representing 
$$p(f_{i+1},...,f_I|a_i=j,e)$$. I leave its derivation as an exercise. 
Restating our computation:

$$p(a_i=j,f\mid{}e) = \alpha(i,j) \times p(f_i\mid{}a_i=j,e) \times \beta(i,j)$$.

The forward-backward algorithm computes these recursive quantities 
efficiently by dynamic programming. It computes $$\alpha(i,j)$$ for all 
$$j$$ for increasing values of $$i$$, and $$\beta(i,j)$$ for all $$j$$ for 
decreasing values of $$i$$. It takes $$IJ^2$$ time. (Can you see why?) 

One more thing worth noticing here: if you replace the sums in each 
recursion with $$\max$$, you'll compute an expression meaning "What is the 
highest probability path that aligns $$e_j$$ to $$f_i$$"? This one-line 
change to the computation is sometimes called the Viterbi algorithm in the 
HMM world. It turns out that both of these algorithms are just the 
sum-product and max-product instances of a more general technique called 
message passing, which is widely used in probabilistic inference and 
applies to arbitrary structured models. The basic idea:

1. Identify the variable assignment whose expectation (or maximizing 
   assignment) you want to calculate.
1. Identify all terms of the model that affect or are affected by the 
   variable assignment. Call these factors.
1. Compute the joint probability of the variable assignment with all other 
   variables in each factor, summing out latent variables and recursively 
   applying the algorithm as needed. Each joint probability is a message to 
   the variable concisely summarizing what some part of the model thinks 
   its expectation should be.

In the HMM, $$\alpha(i,j)$$, $$p(f_i\mid{}a_i=j,e)$$, and $$\beta(i,j)$$ 
are the messages to variable $$a_i$$.

One final thing: Earlier we mentioned that we need to compute $$p(f\mid{}e)$$ 
do get expectations. We do this using inference rules derived from the 
axioms of probability: we can choose any value of $$i$$ between 1 and $$I$$ 
to get $$p(f\mid{}e) = \sum_{j=1}^J p(a_i=j,f\mid{}e)$$.
