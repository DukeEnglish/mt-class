---
img: do_not_want
img_link: http://winterson.com/2005/06/episode-iii-backstroke-of-west.html
title: MT | Syllabus
active_tab: syllabus
---

Lecture materials will be posted after each lecture.

The course is being updated, but 
[Last year's materials](2016syllabus.html) are available
for reference.

<table class="table table-striped"> 
  <thead>
    <tr>
      <th>Week</th>
      <th>Day</th>
      <th>Plan</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <th>Week</th>
      <th>Day</th>
      <th>Plan</th>
    </tr>
  </tfoot>
  <tbody>
    {% for week in site.data.syllabus %}
    <tr>
      <td rowspan="{{ week.plan.size }}">{{ week.name }}</td>
      {% for day in week.plan %}
        {% unless forloop.first == true %}
          </tr>
          <tr>
        {% endunless %}
        <td>{{ day.date | date: "%b %d" }}</td>
        <td>
          {% for header in day.header %}
            {% if header.type == "lecture" %}<span class="label label-primary">Lecture</span>{% endif %}
            {% if header.type == "lab" %}<span class="label label-success">Lab</span>{% endif %}
            {% if header.type == "coursework" %}<span class="label label-danger">Coursework</span>{% endif %}
            {{ header.title }}
            <br/>
          {% endfor %}
          {% if day.items %}
            <ul>
            {% for item in day.items %}
            <li>
              {% if item.url %}<a href="{{ item.url }}">{% endif %}
                {{ item.description }}
              {% if item.url %}</a>{% endif %}
            </li> 
            {% endfor %}
            </ul>
          {% endif %}
        </td>
      {% endfor %}
    </tr>
    {% endfor %}
  </tbody>
</table>

