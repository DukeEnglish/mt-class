---
title: MT HW3
active_tab: hw3 
---

<div class="alert alert-info">
This assignment is due at 4pm on Thursday, 13 April. Late submissions
will receive a mark of zero in accordance with the
<a href="http://web.inf.ed.ac.uk/infweb/student-services/ito/admin/coursework-projects/late-coursework-extension-requests">school-wide late policy</a>.
Submissions received at 4:01pm on the due date will be considered late.
If you require an extension, contact the ITO. I am not empowered to
grant extensions and will ignore any request to do so. Circumstances
under which the ITO will grant an extension are clearly described in
the late policy.

Before you start, please read this page carefully. Submissions that do not
follow the Ground Rules will receive a mark of zero.
</div>

ᐊᒥᓲᕗᑦ ᐱᔭᕆᐊᑐᔪᑦ ᓴᖅᑭᓂᐊᖅᑐᑦ ᓯᕗᓂᒃᓴᒥ <small>| Coursework 3</small>
==========================

From its beginnings, one of the attractions of statistical machine 
translation has been the ability to create a new system 
in a very brief period of time, 
[perhaps as little as a day](https://www.clsp.jhu.edu/vfsrv/ws99/final/Stat_Machine_Translation.pdf).
All that is required is a parallel text. This is now reality: since its 
[initial launch almost eleven years ago](https://research.googleblog.com/2006/04/statistical-machine-translation-live.html)
Google Translate has grown to support 10,506 language pairs, meaning that 
on average, new language pairs are added at a frequency of once every nine
hours.

This capability is useful for more than users of free translation services. In some situations,
it is useful to assemble parallel data and create machine translation 
systems---even rudimentary ones---in a very short period of time. For
example, following the Haiti earthquake in 2010, a large team of volunteers
quickly assembled a new machine translation system between 
[Haitian Creole and English](http://www.mt-archive.info/EAMT-2010-Lewis.pdf),
in order to assist with relief efforts.[^japan]

[^japan]: NLP systems were also quickly assembled [in response to the 2011 Japanese tsunami](https://www.aclweb.org/anthology/I11-1108).

Although [it is not possible to predict earthquakes](http://www.world-earthquakes.com/index.php?option=frc) or
most other natural disasters,
it is possible to prepare for them. MT developers had already
practiced for the Haiti earthquake through participation in 
[surprise language exercises](http://ece.umd.edu/~oard/pdf/talip03.pdf), 
in which teams were asked to develop MT systems from scratch in very brief
periods of time, using whatever resources they could assemble. For example,
in 2003, a research team [quickly assembled an MT system for Cebuano](http://aclweb.org/anthology/N03-2026),
an Austronesian language spoken by around 25 million people in the Philippines.[^desperate]
One important lesson from creating MT for new language pairs is that 
they often highlight specific difficulties of MT models, often of the sort
that are not highlighted by working solely on high-resource Indo-European
language pairs.

[^desperate]: The Cebuano paper title is a play on a [film title](https://en.wikipedia.org/wiki/Desperately_Seeking_Susan).

In [coursework 2](hw2.html) you learned about the basic components of
state-of-the-art neural machine translation. But there are many things that
we don't know about how neural machine translation will behave on new
languages. In this coursework, your task is to contribute
to scientific knowledge about neural machine translation by exploring its
application to **one** of two languages for which there are no published 
NMT results:

1. **Hungarian** is a [Uralic](https://en.wikipedia.org/wiki/Uralic_languages)
   language spoken by 13 million people, and an official language of the
   European Union. It has several features that make it quite different
   from most Indo-European languages, including 
   [vowel harmony](https://en.wikipedia.org/wiki/Vowel_harmony), 
   [topic prominence](https://en.wikipedia.org/wiki/Topic-prominent_language), 
   and [agglutinative morphology](https://en.wikipedia.org/wiki/Agglutinative_language). 
   Our data consists of film subtitles, which are somewhat noisier than 
   news or parliamentary texts. Google Translate handles Hungarian, 
   but as of this writing, it still uses phrase-based MT.[^gt] 
   [Other recent neural MT experiments with European languages](https://arxiv.org/pdf/1610.01108.pdf)
   have not included Hungarian, though [one is planned](http://statmt.org/wmt17/biomedical-translation-task.html)
   for later this year.

1. **Inuktitut** is an [Inuit](https://en.wikipedia.org/wiki/Inuit_languages)
   language native to northern Canada. It is spoken by about 35,000 people;
   most are native speakers and many are monolingual. Inuktitut is [polysynthetic](https://en.wikipedia.org/wiki/Polysynthetic_language). 
   Our data comes from the Legislative Assembly of Nunavut, which publishes 
   its Hansard in both English and Inuktitut. Despite the availability of
   this high-quality parallel corpus, Inuktitut is not included in Google
   Translate. 

[^gt]: Here's how you can tell what engine underlies a particular
       language pair on Google Translate: choose a language pair, and type 
       a sentence into the input window. Now hover your mouse over the 
       output. Part or all of the input will be highlighted, and if you
       click on it, you'll be able to choose an alternate translation. 
       If only [a few words](https://translate.google.com/#en/hu/When%20I%20see%20a%20sentence%20written%20in%20Hungarian%2C%20I%20think%20to%20myself%3A%20this%20is%20really%20written%20in%20English%2C%20but%20it%20has%20been%20coded%20in%20some%20strange%20symbols.%20I%20will%20now%20proceed%20to%20decode.)
       are highlighted at a time, you're looking at the
       output of phrase-based translation. If the 
       [entire sentence](https://translate.google.com/#en/es/When%20I%20see%20a%20sentence%20written%20in%20Spanish%2C%20I%20think%20to%20myself%3A%20this%20is%20really%20written%20in%20English%2C%20but%20it%20has%20been%20coded%20in%20some%20strange%20symbols.%20I%20will%20now%20proceed%20to%20decode.) 
       is highlighted, you're looking at neural MT. By now, you should know
       enough about how these systems work to guess why this feature is 
       currently only available for phrase-based MT.

We will give you a baseline NMT system for both languages. **Your task is to choose
a language, analyze the output of the baseline system, identify some way it might be improved 
and a method to improve it, and then implement and carefully experiment with your method.** The method you choose
does not need to be novel---it could be something we talked about [in class](syllabus.html).
Since there is no prior art for these languages, we will still learn something
interesting from a comparison of existing methods, and indeed there are
existing methods that are likely to be quite helpful for these languages. 
However, you are also welcome to develop novel approaches. Your sole deliverable
for this coursework will be a report explaining your analysis, approach, and results.

Code, data, and other resources
-------------------------------

First, get code and data that we have provided.

    git clone https://github.com/INFR11133/hw2.git

The layout of the code and data is identical to what you received 
for [coursework 2](hw2.html). This time, we've provided you with a complete
encoder-decoder model with our own implementation of attention.  We've also provided you
with some pretrained models on the two language pairs, so you can experiment 
with them to see how they behave out of the box. 

There is one small thing you must do before you can use the Inuktitut model, which was
slightly too big for github. In the directory `in_en_model_50000`, run this command:

    cat seq2seq_50000sen_3-3layers_200units_baseline_aajuq_SOFT_ATTN_PART1.model seq2seq_50000sen_3-3layers_200units_baseline_aajuq_SOFT_ATTN_PART2.model > seq2seq_50000sen_3-3layers_200units_baseline_aajuq_SOFT_ATTN.model
    rm *PART*

Our new implementation includes a script `prepare_seq2seq.py` with some 
utility functions that may be useful if your experiments require changing
the way the input data is preprocessed.

You are not required to
use our implementation: you may use your own implementation or a third-party 
implementation, as you prefer. However, if you do so, you **must** retrain the baseline
models using your chosen codebase. We want you to compare alternative 
**models or algorithms**, not implementations, which can vary in many 
ways---often undocumented ways. If you were to compare the baseline model trained with our
code and another model trained with different code, we would not know if 
differences in behaviour were due to differences in the model, or differences
in the implementation.

Although you are free to use third-party software, 
**we will not offer support for this**. If you use third-party tools, it's
up to you to figure out how they work, though some developers
may offer support via email. These days, new neural network frameworks 
and NMT systems based on these frameworks pop up [all the time](https://github.com/jonsafari/nmt-list). A 
non-exhaustive list of those that seem to be popular includes:

* [NeMaTus](https://github.com/rsennrich/nematus), based on Theano. 
* [Kyunghyun Cho's code](http://www.kyunghyuncho.me/home/code), also based on Theano.
* [LamTram](https://github.com/neubig/lamtram), based on DyNet. 
* [OpenNMT](http://opennmt.net/), based on Torch/ PyTorch.

You may find other toolkits
that are more suited to your project or your programming language or framework of choice.
However, as noted above, a comparison of two different implementations (e.g. NeMaTus
vs. LamTram) would not be particularly interesting.

In addition to other MT software tools, you may find it useful to use 
other NLP tools. For example, the [website](http://www.inuktitutcomputing.ca/index.php)
from which we obtained the Inuktitut data includes other resources, including a
[morphological analyzer](http://www.inuktitutcomputing.ca/Uqailaut/info.php). 

You are free to use other data, including monolingual data. But just about the only experiment
we don't want to see is one in which you simply add more parallel data to the baseline
system and compare. We already know that will help. If you use more parallel data to train, you 
should include it in your baseline and compare it with a new method trained on the same
data.

If you aren't sure where to start, you may want to first read about the languages
in the [World Atlas of Language Structures](http://wals.info/). You may also find
it helpful to look at the [Language index](http://www.mt-archive.info/srch/languages.htm)
of the [MT archive](http://www.mt-archive.info/), which organizes machine translation
research papers by language. Even though there are no pointers to NMT for these languages,
it may be possible to adapt techniques that have been used in other MT approaches.

Finally: although the design, implementation, analysis, and writing in your report
must be solely the work of you (and optionally a partner;
see below), **you are encouraged** to share knowledge and information on 
[the piazza forum](https://piazza.com/class/irvzfyo9ahs6mi), up to an including 
things that you learn about languages, software, and data during the course of 
completing the assignment. This isn't a competition, and you will learn more by
asking and answering questions. Your report should correctly acknowledge any
help you received, and clearly identify what is and is not your own work. 


Report guidance
---------------

**What to work on.** Most topics that draw on knowledge of language and
modeling language with neural networks are fair game. For example, in class
we talked about large-vocabulary models and their interaction with morphology,
different approaches to attention, using monolingual data, and syntactic 
models. The associated readings for these lectures on 
[the syllabus](syllabus.html) are a good starting point for learning about
current approaches. However, neural machine translation is a very active
research area and you may find other inspiration from 
[Google scholar](https://scholar.google.co.uk/scholar?q=neural+machine+translation).
As noted above, we prefer that you don't focus experiments on the effect
of adding more data (we know that will make the numbers go up, but what do
we learn from it?) and comparison of implementations (this tells us who
is a better engineer, not which ideas are most useful for these languages).

Whatever you do, 
your report should justify your choice of approach using evidence in the form
of empirical observations about the baseline system, knowledge of language 
typology, and/ or conjectures about how these properties of the data and the language may interact with a
basic encoder-decoder model with attention.

**Metrics and improvements.** 
You are *required* to clearly define objective measures that test whether you have
made improvements. Think very carefully about this; the way in which you 
measure success should depend on your goals. For example,
if you decide to implement [character-based MT](https://arxiv.org/abs/1610.03017), you
cannot compare it to the baseline using perplexity, since the set of predicted output
events at each time step is different (you should of course measure perplexity on
training as a check that your model is learning). Likewise, if you were to try to 
improve handling of subject-verb agreement, it does not make sense to evaluate the
result using BLEU, a document-level metric that is sensitive to every n-gram in
the output. [Try something else](https://arxiv.org/pdf/1612.04629.pdf).
On the other hand, if you decide to implement [minimum risk training](https://arxiv.org/abs/1512.02433),
you should probably measure BLEU, TER and/ or METEOR, since the purpose
of minimum risk training is to directly minimize an arbitrary error measure (or
equivalently, to maximize a gain). Regardless of what you
measure, you should think carefully about the limitations of measurement, and also
include qualitative analysis (using examples) where appropriate.

Once you have decided what to measure,
you should make a serious attempt to improve that measure. But it is
perfectly ok if the method you implement does not produce improvements in the end. 
You are also *not* required to improve the system on specific measures (e.g. BLEU).
What matters is that you carefully analyze the results, good or bad, and tell us
what you learned about neural machine translation for one of these languages.
It is far more important to have a carefully designed hypothesis and experimental
design than it is to have positive results.
[Research is the process of going up alleys to see if they are blind.](https://en.wikiquote.org/wiki/Marston_Bates)

**Choice of language.** You are only required to work with one of the two languages, and the
choice of language is up to you. It is fine to work with both, but you 
should do so only if it makes sense in the context of your project; your
marks will depend on the quality and depth of your report, not which 
language you use.

**Sanity checking.** It's often difficult to know 
whether poor results of a machine learning system are due to model error, search
error, or implementation error. To get good at it, you should get in the habit
of asking and answering many small questions. Are there are lot of out-of-vocabulary
words? Why? Is the training method working? (Check to see that training loss is 
decreasing). Can this model class learn some specific phenomena? (Generate
some synthetic data that contains the pattern and see if the model can learn it; if it doesn't learn
from ideal data, then it definitely won't learn on real data!) Will this implementation work
on a big data set? (Try it on a small one first before scaling up; it will save you
a lot of time finding the worst bugs.) Especially when you're venturing into new
methods, you may want to explain some of these sanity checks in your report. If
they convince you that your implementation is correct, they might convince us, too.

**Marking**. Your report will be evaluated according to the University's
[common marking scheme](http://www.ed.ac.uk/student-administration/exams/regulations/common-marking-scheme).
Just as in the real world, there are no right or wrong answers to this
coursework. It asks you look at a problem and make progress on it, to synthesize
what you have learned and to clearly explain your reasoning at every step along
the way, [as is expected in a level 11 course](http://scqf.org.uk/wp-content/uploads/2014/03/SCQF-Revised-Level-Descriptors-Aug-2012-FINAL-web-version1.pdf).
Concretely, your report should clearly convey:

* **[20 marks] A research question.** As discussed above, this question should be motivated by 
  an analysis of what you find in the baseline systems, and should be supported with
  examples from the data, an explanation of language typology, and/ or discussion of model
  architecture and its interaction with properties of the data or language. 

* **[20 marks] A description of measures and methods.** Clearly explain what you
  intend to measure and how it relates your hypotheis. Clearly explain how your 
  approach works. I recommend using the first two levels in [Marr's level of
  analysis](https://en.wikipedia.org/wiki/David_Marr_(neuroscientist)#Levels_of_analysis):
  a mathematical/ algorithmic description is preferable to code or a 
  textual description of code. Keep in mind a complete description of your 
  methods should include parameters of network architectures (e.g. embedding
  dimensions) and learning algorithms (e.g. initialization, learning rates,
  number of epochs or stopping criterion).

* **[40 marks] Results and analysis.** What are your results? What do you
  conclude from them? What further things might you try based on what you've
  learned? How do your results relate to other work in the field that you learned
  about in class or read about independently?

* **[20 marks] Presentation and clarity.** Your presentation should be clear and concise,
   providing enough information to enable work to be reproduced, informative 
   discussion and conclusions. Imagine that you give your report to a classmate. Could
   they understand what you did? Would they be able to reimplement your approach?
   Would they believe your results? Would they find your interpretation of the results
   convincing?

You should expect to write a concise scientific report. If you aren't
sure what that looks like, look at some good training data (papers from the [syllabus](file:///Users/alopez/teaching/mt2017/website/mt-class/_site/syllabus.html)) or [read some advice](https://www.microsoft.com/en-us/research/academic-program/write-great-research-paper/).
Note that (unlike with a dissertation) you don't need to extensively describe
the general problem of MT or why you're working on it: obviously, that has
been decided for you.

Ground Rules
------------
* **You must use the [overleaf template](https://www.overleaf.com/read/mvkfhzzjyywq)
  for your report**. You are permitted up to four (4) pages for the main text of your
  report and an unlimited number of pages for references and appendices, including
  plots, figures, diagrams, and long examples. Your appendix should not include any
  information (e.g. equations) that is essential to understand how you approached
  your problem or implemented your solution, but supporting figures, especially if
  they contain concrete evidence illustrating some point, are very welcome.

* **You are encouraged to work in pairs.** Previous courseworks have
  assessed your individual work, so in this coursework we are able to relax
  this restriction. I encourage you to seek partners with complementary
  skills to your own. But choose your partner wisely: if you submit
  as a pair, you agree to receive the same mark for your work. I refuse
  to adjudicate [Rashomon](https://en.wikipedia.org/wiki/Rashomon)-style
  stories about who did or did not contribute. If you would like a partner
  but have no one in mind, you can search for a partner on 
  [piazza](https://piazza.com/class/irvzfyo9ahs6mi#).
  
  Once you have agreed to work with a partner, you **must** 
  [email me with "5636915" in the subject line](mailto:alopez@inf.ed.ac.uk?subject=5636915),
  copying your partner. Please include both UUNs in the body of the email. I 
  will then know that a submission received from either UUN should be marked
  on behalf of both you and your partner. **You may not change partners
  after you have emailed me.** This means that your email consistutes an
  agreement to the parameters outlined above. It
  will also help us predict marking load, which may mean you get your 
  results faster. **I will ignore emails received after the deadline.**

* You must submit these files **and only these files**. 
    1. `report.pdf`: A file containing your writeup.    
    1. `translations.txt`: The output of your final system on the test set. 

* Your name(s) __must not appear in any of the submitted files__. If your name
  appears in either file you will (both) receive a mark of zero.

To submit your files on dice, run:

    submit mt 3 report.pdf translations.txt

### Credits

This assignment was developed by
[Ida Szubert](https://www.inf.ed.ac.uk/people/staff/Katarzyna_Szubert.html)
and [Sameer Bansal](https://0xsameer.github.io/),
with helpful comments from 
[Nikolay Bogoychev](http://homepages.inf.ed.ac.uk/s1031254/),
[Anny Currey](http://homepages.inf.ed.ac.uk/s1639783/), 
[Soňa Galovičová](http://homepages.inf.ed.ac.uk/s1206522/),
[Jonathan Mallinson](http://homepages.inf.ed.ac.uk/s1564225/),
and
[Joana Ribeiro](https://www.inf.ed.ac.uk/people/students/Joana_Ribeiro.html).
[Adam Lopez](https://alopez.github.io/)
added all of the errors.

### Footnotes
