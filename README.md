Machine Translation Course Material
===================================

This directory contains a complete set of course material for a senior 
undergraduate or graduate-level course in machine translation.

All of this material is made available under a 
[Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/)
You are free to reuse it any way you like if you acknowledge that you got it from us: 
* [Adam Lopez](http://homepages.inf.ed.ac.uk/alopez), University of Edinburgh
* [Matt Post](http://www.cs.jhu.edu/~post), Johns Hopkins University
* [Chris Callison-Burch](http://www.cis.upenn.edu/~ccb), University of Pennsylvania
* [Chris Dyer](http://www.cs.cmu.edu/~cdyer), Carnegie Mellon University

The current fork of these files is used to teach the 
[machine translation course](http://www.inf.ed.ac.uk/teaching/courses/mt/)
at the [University of Edinburgh](http://www.ed.ac.uk/home) by Adam Lopez.

Homework Assignments
--------------------

The code and data for the homework assignments are maintained by Adam Lopez
and distributed under an MIT License. Feel free to reuse them! 
[DREAMT: Decoding, Reranking, Evaluation, and Alignment for Machine Translation
](https://github.com/alopez/dreamt)

Web-Based Stack Decoder
-----------------------

A web-based live demo of the classic stack decoding algorithm used in word-based
statistical machine translation was also used in the class. The code is maintained
by Matt Post, and you can get it [here](https://github.com/mjpost/word-decoder/).

Building the course page for Edinburgh Informatics
--------------------------------------------------

The School of Informatics mandates a uniform style for all of the pages hosted
on its top domain. This is acheived through ssi directives included in source
files. Informatics also manages updates to the website through cvs. To comply
with these requirements, while still writing content in markdown for jekyll
and managing everything using git, I use the following process:

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

