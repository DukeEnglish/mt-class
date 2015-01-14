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

Maybe you would also align *zu* to *rose*.

