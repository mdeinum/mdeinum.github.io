---
layout: page
title: Welcome
use-site-title: true
cover-img: { "/assets/img/IMG_0031.jpg" : "Dolphine @ Wadi Lachmi - 2005"}
css: '/css/home.css'
show-avatar: false
---
<div class="spacer"></div>

<div class="row introduction">
  <div class="col-md-4 col-md-offset-0 col-sm-4 col-sm-offset-0 col-xs-12 col-xs-offset-0">
    <img src="/assets/img/avatar-icon.png" alt="">
  </div>
  <div class="col-md-8 col-md-offset-0 col-sm-8 col-sm-offset-0 col-xs-12 col-xs-offset-0 text-left">My name is <strong>Marten Deinum</strong> and I'm the (co)author of a few <a href="https://deinum.biz/books/">books</a> related to Spring. I also work as a Consultant for a company called <a rel="noreferrer noopener" href="https://conspect.nl" target="_blank">Conspect</a> and I work as a <a rel="noreferrer noopener" href="https://rebeldiving.nl" target="_blank">diving instructor</a>.<br><br>I maintain a few <a rel="noreferrer noopener" href="https://www.github.com/mdeinum" target="_blank">projects</a> containing utilities to make development of applications with Java and Spring easier. When not coding or diving I answer Spring related questions on <a rel="noreferrer noopener" href="https://stackoverflow.com/" target="_blank">StackOverflow</a>.
  </div>
</div>

----

<h1 class="text-center">Current Projects</h1>

<div class="spacer"></div>

<div class="row text-center">
  <div class="col-md-4 col-md-offset-0 col-sm-4 col-sm-offset-0 col-xs-12 col-xs-offset-0 text-center">
    <div class="project-card">
      {%- assign gh-user = "mdeinum"-%}
      {%- assign gh-project = "spring-batch-excel" -%}
      <a target="_blank" href="https://github.com/spring-projects/spring-batch-extensions" class="project-link" title="Go to Github Poject Page">
        <span class="fa-stack fa-4x">
          <i class="fa fa-square fa-stack-2x stack-color"></i>
          <i class="fa fa-terminal fa-stack-1x fa-inverse"></i>
        </span>
        <h4>{{- gh-project -}}</h4>
        <hr class="seperator">
        <p class="text-muted">Spring Batch extension which contains an <code>ItemReader</code> implementation for Excel based on Apache POI.</p>
        <hr class="seperator">
        <img src="https://img.shields.io/github/forks/spring-projects/spring-batch-extensions.svg?style=social&label=Fork" alt="Github" title="Github Forks">
        <img src="https://img.shields.io/github/stars/spring-projects/spring-batch-extensions.svg?style=social&label=Stars" alt="Github" title="Github Stars">
      </a>
    </div>
  </div>
  <div class="col-md-4 col-md-offset-0 col-sm-4 col-sm-offset-0 col-xs-12 col-xs-offset-0 text-center">
    <div class="project-card">
      {%- assign gh-project = "spring-utils" -%}
      <a target="_blank" href="https://github.com/{{- gh-user -}}/{{- gh-project -}}" class="project-link" title="Go to Github Poject Page">
        <span class="fa-stack fa-4x">
          <i class="fa fa-square fa-stack-2x stack-color"></i>
          <i class="fa fa-file-code-o fa-stack-1x fa-inverse"></i>
        </span>
        <h4>{{- gh-project -}}</h4>
        <hr class="seperator">
        <p class="text-muted">Utility and helper classes collected and created over the years. Some of these might be useful to others</p>
        <hr class="seperator">
        <img src="https://img.shields.io/github/forks/{{- gh-user -}}/{{- gh-project -}}.svg?style=social&label=Fork" alt="Github" title="Github Forks">
        <img src="https://img.shields.io/github/stars/{{- gh-user -}}/{{- gh-project -}}.svg?style=social&label=Stars" alt="Github" title="Github Stars">
      </a>
    </div>
  </div>  
</div>

----

<h1 class="text-center">Recent Posts</h1>
<div class="spacer"></div>

<div class="posts-list">
  {% for post in site.posts limit:3 %}
  <article class="post-preview">
    <a href="{{ post.url | prepend: site.baseurl }}">
      <h2 class="post-title">{{ post.title }}</h2>

      {% if post.subtitle %}
      <h3 class="post-subtitle">
        {{ post.subtitle }}
      </h3>
      {% endif %}
    </a>

    <p class="post-meta">
      Posted on {{ post.date | date: "%B %-d, %Y" }}
    </p>

    <div class="post-entry-container">
      {% if post.image %}
      <div class="post-image">
        <a href="{{ post.url | prepend: site.baseurl }}">
          <img src="{{ post.image }}">
        </a>
      </div>
      {% endif %}
      <div class="post-entry">
        {{ post.excerpt | strip_html | xml_escape | truncatewords: site.excerpt_length }}
        {% assign excerpt_word_count = post.excerpt | number_of_words %}
        {% if post.content != post.excerpt or excerpt_word_count > site.excerpt_length %}
          <a href="{{ post.url | prepend: site.baseurl }}" class="post-read-more">[Read&nbsp;More]</a>
        {% endif %}
      </div>
    </div>

    {% if post.tags.size > 0 %}
    <div class="blog-tags">
      Tags:
      {% if site.link-tags %}
      {% for tag in post.tags %}
      <a href="{{ site.baseurl }}/tags#{{ tag }}">{{ tag }}</a>
      {% endfor %}
      {% else %}
        {{ post.tags | join: ", " }}
      {% endif %}
    </div>
    {% endif %}

   </article>
  {% endfor %}
</div>