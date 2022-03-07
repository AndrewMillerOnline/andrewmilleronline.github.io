---
title: Projects
icon: fas fa-info-circle
order: 3
excerpt_separator: <!--more-->
---
Here are some of the projects I've worked on lately.
<!--more-->
<div class="d-flex flex-wrap">
    {% for project in site.categories.Projects %}
    <div class="col-6">
        <h2><a href="{{ project.url }}">{{ project.name }}</a></h2>
        <img width="350px" src="{{ project.image }}" alt="{{ project.image.alt }} | Project Preview Image">
        <div>
        {{ project.description }}
        </div>
        <div class="tools d-flex flex-wrap">
            Tools Used: {% for tool in project.tools %} <span class="tool">{{tool}}</span> {% endfor %}
        </div>
    </div>
    {% endfor %}
</div>