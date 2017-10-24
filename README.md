Machine Translation Course Material
===================================

This directory contains a complete set of course material for a senior 
undergraduate or graduate-level course in machine translation. It was last
updated in spring 2017, and I have no plans to update it again. It will 
probably be completely out of date within a few years. With that caveat
in mind:

All of this material is made available under a 
[Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/)
You are free to reuse it any way you like if you acknowledge that you got it from us: 
* [Adam Lopez](http://homepages.inf.ed.ac.uk/alopez), University of Edinburgh
* [Matt Post](http://www.cs.jhu.edu/~post), Johns Hopkins University
* [Chris Callison-Burch](http://www.cis.upenn.edu/~ccb), University of Pennsylvania
* [Chris Dyer](http://www.cs.cmu.edu/~cdyer), Deepmind / Carnegie Mellon University 
* [Anoop Sarkar](http://www.cs.sfu.ca/~anoop/), Simon Fraser University

The current fork of these files was used to teach the 
[machine translation course](https://alopez.github.io/mt-class/)
at [Johns Hopkins University](http://www.jhu.edu) in 2012 and 2014 and
at the [University of Edinburgh](http://www.ed.ac.uk/home) between 2015 and
2017 by Adam Lopez. The materials evolved substantially over that time, and
reflect the 2017 offering of the course.

Keynote source for slides
-------------------------

The keynote source files are under `assets/keynote`.

Homework Assignments and Labs
-----------------------------

There are two sets of homework assignments: one for phrase-based statistical
machine translation, and a newer one for neural MT. 

The code and data for the older homework assignments are maintained by Adam 
Lopez and distributed under an MIT License. Feel free to reuse them! 
[DREAMT: Decoding, Reranking, Evaluation, and Alignment for Machine Translation
](https://github.com/alopez/dreamt)

The code and data for the newer neural MT homework assignments was developed
by Ida Szubert and Sameer Bansal, and appears in the 
[course repository](https://github.com/INFR11133). Text for the three 
assignments can be found in this repository.

Web-Based Stack Decoder
-----------------------

A web-based live demo of the classic stack decoding algorithm used in word-based
statistical machine translation was also used in the class. The code is maintained
by Matt Post, and you can get it [here](https://github.com/mjpost/word-decoder/).

Building the course page for Edinburgh Informatics
--------------------------------------------------

The School of Informatics manages updates to the website through cvs. To 
comply with this unspeakable requirement, while still staying sane by 
writing content in markdown for jekyll and managing everything using git, I 
use the following process:

    # Set $CVSROOT
    export CVSROOT=:pserver:<username>@www.inf.ed.ac.uk:/cvsroot

    # Check out both cvs and git versions of the repo
    cvs login
    cvs checkout web/teaching/courses/mt
    git cvsimport -d $CVSROOT -C inf-mt -r cvs -k web/teaching/courses/mt

    # Configure the git clone of the cvs repo
    cd inf-mt
    git config cvsimport.module web/teaching/courses/mt
    git config cvsimport.r cvs
    git config cvsimport.d $CVSROOT
    
    # Now, I can make changes as needed in this repository. If I want
    # to preview the results locally, I just run "jekyll build". When
    # I'm ready to commit the changes to the informatics server, I build
    # using a special config file that overwrites the git copy of the
    # the class directory, and then commit it from there using the git
    # bridge to cvs, like so:
    cd mt-class
    jekyll build --config _config4inf.yml
    cd ../inf-mt
    git commit -am <commit message>
    git cvsexportcommit -w ../web/teaching/courses/mt -u -p -c HEAD^ HEAD

Much of the process was adapted from [this stackoverflow question](http://stackoverflow.com/questions/584522/how-to-export-revision-history-from-mercurial-or-git-to-cvs/#586225).

