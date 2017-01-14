---
img: do_not_want
img_link: http://winterson.com/2005/06/episode-iii-backstroke-of-west.html
title: MT | Syllabus
active_tab: 2016syllabus
---

<div class="alert alert-danger">
  This is the 2016 syllabus. The 
  <a href="syllabus.html">2017 syllabus</a> will be substantially different
  in places, but you're welcome to refer to previous material here.
</div>

These slides are visual aids for the corresponding lectures, meant to guide 
you towards intuitons about the material. To really understand the material,
you must read and interact with it in depth, which you cannot do by reading
slides. I urge you to read the source material (especially the textbook) and
not to rely extensively on the slides.

<table class="table table-striped"> 
  <tbody>
    <tr>
      <th>Date</th>
      <th>Topic</th>
      <th>Readings (<span class="glyphicon glyphicon-star"></span> = optional)</th>
    </tr>
    {% for lecture in site.data.2016syllabus.past %}
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

