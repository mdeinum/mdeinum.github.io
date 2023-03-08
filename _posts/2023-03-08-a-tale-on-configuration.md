---
layout: post
title: A Tale of configuration formats
date: 2023-03-08
categories:
- Java
- Spring
tags:
- configuration
published: false
gh-repo: mdeinum/blog-configuration
gh-badge: star	
---

When you are developing a Java application you will eventually need some configuration for your application. When you thought the fun part was over in coding your Java programm the fun now just begins with selecting your way of providing configuration to your application. When you are a single developer this can be an easy discussion, however in a team this can be an interesting discussion. 

At first you might think what is there to choose, I need configuration so why not just put all that information in a properties file. Well there is more then you might think there is to choose as there is a variety of formats to pick from. Here is an (not exhaustive) list:

* Java Properties in properties or xml file
* [TOML](https://toml.io/en/)
* [HOCON](https://github.com/lightbend/config/blob/main/HOCON.md)
* [YAML](https://yaml.org)
* [JSON](https://www.json.org/json-en.html)
* XML
* [INI](https://en.wikipedia.org/wiki/INI_file) files
* [PList](https://en.wikipedia.org/wiki/Property_list)
* Unix Config file

This is just a list of file formats and we aren't even considering things like a environment variables, Java system variables, datastores and property vaults (common in cloud environments). So the list is even longer and I probably left out some other formats, but these are the formats I have encountered in my career so-far. 
