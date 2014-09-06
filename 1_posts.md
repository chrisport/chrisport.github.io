---
layout: default
title: Posts
permalink: /posts/
weight : 2
---
 <ul class="post-list">
        {% for post in site.posts %}
        <li class="well">
        
            <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
            <h2>
                <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
            </h2>
            <a href="{{post.author-url}}"><span class="post-meta glyphicon glyphicon-user"> {{post.author}}  </span></a>
        </li>
        {% endfor %}
    </ul>