---
layout: page
title: Welcome
use-site-title: true
css: '/css/home.css'
---
<div class="spacer"></div>

<div class="row text-center">
<div class="hero">My name is <strong>Marten Deinum</strong> and I'm the (co)author of a few <a href="https://deinum.biz/books/">books</a> related to Spring. I also work as a Consultant for a company called <a rel="noreferrer noopener" href="https://conspect.nl" target="_blank">Conspect</a> and I work as a <a rel="noreferrer noopener" href="https://rebeldiving.nl" target="_blank">diving instructor</a>.<br><br>I maintain a few <a rel="noreferrer noopener" href="https://www.github.com/mdeinum" target="_blank">projects</a> containing utilities to make development of Java with Spring easier. When not coding or diving I answer Spring related questions on <a rel="noreferrer noopener" href="https://stackoverflow.com/" target="_blank">StackOverflow</a>.</div>
<div><img style="width:150px;vertical-align:text-top" src="https://mdeinum.files.wordpress.com/2020/07/headshot.png" alt=""></div>

</div>

<h1 class="text-center">Current Projects</h1>

<div class="spacer"></div>

<div class="row text-center">
  <div class="col-md-4 col-md-offset-0 col-sm-4 col-sm-offset-0 col-xs-12 col-xs-offset-0 text-center">
    <div class="project-card">
      {%- assign gh-user = "mdeinum"-%}
      {%- assign gh-project = "spring-batch-excel" -%}
      <a target="_blank" href="https://github.com/{{- gh-user -}}/{{- gh-project -}}" class="project-link" title="Go to Github Poject Page">
        <span class="fa-stack fa-4x">
          <i class="fa fa-square fa-stack-2x stack-color"></i>
          <i class="fa fa-terminal fa-stack-1x fa-inverse"></i>
        </span>
        <h4>{{- gh-project -}}</h4>
        <hr class="seperator">
        <p class="text-muted">Spring Batch extension which contains an <code>ItemReader</code> implementation for Excel based on Apache POI.</p>
        <hr class="seperator">
        <img src="https://img.shields.io/github/forks/{{- gh-user -}}/{{- gh-project -}}.svg?style=social&label=Fork" alt="Github" title="Github Forks">
        <img src="https://img.shields.io/github/stars/{{- gh-user -}}/{{- gh-project -}}.svg?style=social&label=Stars" alt="Github" title="Github Stars">
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
        <p class="text-muted">Utility classes collected and created over the years. Some of these might be useful to others</p>
        <hr class="seperator">
        <img src="https://img.shields.io/github/forks/{{- gh-user -}}/{{- gh-project -}}.svg?style=social&label=Fork" alt="Github" title="Github Forks">
        <img src="https://img.shields.io/github/stars/{{- gh-user -}}/{{- gh-project -}}.svg?style=social&label=Stars" alt="Github" title="Github Stars">
      </a>
    </div>
  </div>  
</div>