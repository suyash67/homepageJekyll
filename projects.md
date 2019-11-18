---
layout: home
title: Projects
permalink: /projects/
order: 1
---

<div class = "home-proj-list">
    {%- if site.posts.size > 0 -%}
    <ul class="post-list">
        {% for category in site.categories %}
            <!-- List projects here -->
            {%- if category[0] == "project" -%}
                <li>
                    {% for post in category[1] %}
                        <li>
                        {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
                        <span class="post-meta">{{ post.date | date: date_format }}</span>
                        <h3>
                            <a class="post-link" href="{{ post.url | relative_url }}">
                            {{ post.title | escape }}
                            </a>
                        </h3>
                        {%- if post.image -%}
                            {% include image.html name=post.image caption="" %}
                        {%- endif -%}
                        {%- if post.tldr -%}
                            {{ post.tldr }}
                        {%- endif -%}
                        </li>      
                    {% endfor %}
                </li>
            {%- endif -%}
        {% endfor %}
    </ul>
    {%- endif -%}
</div>