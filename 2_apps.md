---
layout: default
title: Apps
permalink: /apps/
---
{% assign sorted_pages = site.apps | sort:site.apps.weight %}
{% for app in sorted_pages %}
<li class="app-list">
  <b><a href="{{app.playlink}}" target="_blank">{{app.title}}</a></b>
        
        {{ app.content }}
  
  <div align="center">
    <a href="{{app.playlink}}" target="_blank">
      <img src="{{app.image}}">
    </a>
  </div>
 </li>
{% endfor %}
