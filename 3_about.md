---
layout: headless_page
title: About
permalink: /about/
weight : 1
---
{% for member in site.data.members %}
<div class="footer-col-wrapper">
    <div class="footer-col  footer-col-1">
            <b><p class="text">{{ member.name }}</p></b>
                    <p class="text">{{ member.description }}</p>
                    
                    
</div>
    <div class="footer-col  footer-col-2">
  <ul class="social-media-list">
          {% if member.github_username %}
          <li>
              <a href="https://github.com/{{ member.github_username }}">
                  <i class="fa fa-github fa-lg"></i>
                  <span class="username">{{ member.github_username }}</span>
              </a>
          </li>
          {% endif %}
          {% if member.twitter_username %}
          <li>
              <a href="https://twitter.com/{{ member.twitter_username }}">
                  <i class="fa fa-twitter fa-lg"></i>
                  <span class="username">{{ member.twitter_username }}</span>
              </a>
          </li>
          {% endif %}
       
      </ul>
      </div>
       <div class="footer-col  footer-col-3">
        <ul class="social-media-list">
              {% if member.linkedin_username %}
                 <li>
                     <a href="http://tr.linkedin.com/in/{{ member.linkedin_username }}">
                         <i class="fa fa-linkedin-square fa-lg"></i>
                         <span class="username">{{ member.linkedin_username }}</span>
                     </a>
                 </li>
                 {% endif %}
                 {% if member.stackoverflow_username %}
                 <li>
                     <a href="http://stackoverflow.com/users/3210551/chrisport">
                        <i class="fa fa-stack-overflow fa-lg"></i>
                       <span class="username">{{member.stackoverflow_username}}</span>
                     </a>
                  </li>
                 {% endif %}
                 </ul>
                 </div>
      </div>
{% endfor %}
