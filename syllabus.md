---
img: do_not_want
img_link: http://winterson.com/2005/06/episode-iii-backstroke-of-west.html
title: MT | Syllabus
active_tab: syllabus
---

<div class="alert alert-danger">
  This page contains information for the 2016 version of the course.
  For 2017, the content of the course is likely to be similar, but
  the form of the course, and particularly the assessment, will change.
</div>

Slides will be posted after each lecture. Slides for lectures that have not
yet occurred are subject to change. I consider slides to be
visual aids for the lecture. The purpose of the lecture is to guide you
towards intuitions about the material, but to really understand the material,
you must read and interact with it in depth, which you cannot do from the 
slides. I urge you to read the source 
material (especially the textbook) and not to rely extensively on the slides,
for exam preparation.

<table class="table table-striped"> 
  <tbody>
    <tr>
      <th>Date</th>
      <th>Topic</th>
      <th>Readings (<span class="glyphicon glyphicon-star"></span> = optional)</th>
    </tr>
    {% for lecture in site.data.syllabus.past %}
    <tr>
      <td>{{ lecture.date | date: "%b %d" }}</td>
      <td>
        {% if lecture.slides %}<a href="{{ lecture.slides }}">{{ lecture.title }}</a>
        {% else %}{{ lecture.title }}{% endif %}
      {% if lecture.links %}
        {% for link in lecture.links %}
          <p><a href="{{ link.url }}">{{ link.text }}</a></p>
        {% endfor %}
      {% endif %}
  {% if lecture.language %}
	<br/><a href="lin10.html">Language in 10</a>: <a href="{{ lecture.language_slides }}">{{ lecture.language }}</a>
        {% endif %}
      </td>
      <td>
        {% if lecture.reading %}
          <ul class="fa-ul">
          {% for reading in lecture.reading %}
            <li>
            {% if reading.optional %}<span class="glyphicon glyphicon-star"></span>
            {% else %}{% endif %}
            {% if reading.author %}{{ reading.author }},{% endif %}
            {% if reading.url %}
            <a href="{{ reading.url }}">{{ reading.title }}</a>
            {% else %}
            {{ reading.title }} 
            {% endif %}
            </li>
          {% endfor %}
          </ul>
        {% endif %}
      </td>
    </tr>
    {% endfor %}

  </tbody>
</table>

