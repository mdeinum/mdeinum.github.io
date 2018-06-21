---
author: mdeinum
comments: true
date: 2013-07-01 17:46:19+00:00
layout: post
link: https://mdeinum.wordpress.com/2013/07/01/yeoman-behind-a-corporate-proxy/
slug: yeoman-behind-a-corporate-proxy
title: Yeoman behind a (corporate) proxy
wordpress_id: 98
categories:
- Web Development
tags:
- bower
- nodejs
- npm
- windows
- yeoman
---

After playing around with AngularJS and Yeoman at home I decided to try it out at work. To clarify @Home I use a Mac with OSX and use [Homebrew](http://mxcl.github.io/homebrew/) to install additional packages. @Work I have a virtual workstation running Windows 7 installed. Trying out that setup on that configuration was, somewhat, of a challenge. Steps



	
  1. Install [NodeJS](http://www.nodejs.org/)

	
  2. Configure proxy for NPM and [Bower](http://bower.io/)

	
  3. (Optional) Configure NPM to store modules and cache on local drive instead of network

	
  4. Install [Yeoman](http://yeoman.io/)

	
  5. Configure GIT

	
  6. Add NPM Modules directory to PATH

	
  7. Test your installation




## <!-- more -->Install NodeJS


After downloading NodeJS it is as simple as running the installer.


## Configure Proxy for NPM (and Bower)


[![proxy-config](http://mdeinum.files.wordpress.com/2013/07/proxy-config.png?w=300)](http://mdeinum.files.wordpress.com/2013/07/proxy-config.png)Open your Computer Settings (right click computer) and navigate to the Advanced Settings -> Environment Variables... Add 2 properties **HTTP_PROXY** and **HTTPS_PROXY**. (format for the url http(s)://username@password:server:port, only server is required). I have installed a local proxy due to the fact I'm behind a NTLM proxy and maven doesn't support that).


## (Optional) Configure NPM to store modules and cache on local drive instead of network


By default NPM stores modules and cache in the AppData\Roaming directory, which in my case, is on the network. For optimum performance I wanted it to be stored on a local drive instead for 2 reasons:



	
  1. I already was reaching the limits of my network storage

	
  2. Performance, local drive is faster compared to a network share.


To configure npm to store the modules and cache localy we need to execute the following to commands from the command line

    
    <code>npm config set prefix D:\data\nodejs\npm --global
    npm config set cache D:\data\nodejs\npm-cache --global</code>




## Install Yeoman


After installing NodeJS and configuring the HTTP(S) proxy, and optionally the module and cache directories, we are ready to install Yeoman. On the command line type the following:

    
    <code>npm install -g yo grunt-cli bower</code>


This can run for some time, it downloads quite some packages and it depends on your internet connection and speed of the location the packages are installed (see previous step).


## Install GIT


Yeoman uses Bower as a JavaScript package manager. Bower needs access to git repositories and in general does this by using the git: protocol based URLs. However most corporate proxies don't allow those to pass. We need to tell git that instead of git: use **https:**. We can do this by issueing yet another command from the command line:

    
    git config --global url."https://".insteadOf git://




## Add NPM Modules directory to path


Could be my installation but after installing Yeoman I couldn't run it. When running the following command,

    
    yo webapp


I was greeted with a nice exception (even after opening a new command line).

    
    'yo' is not recognized as an internal or external command, operable program or batch file.


I manually needed to add the modules directory to the path. To figure out this directory type the following in the command line.

    
    npm config ls -l


This will list the full configuration (it also displays the defaults), now find the entry with the label '_prefix_'.
[![search-prefix](http://mdeinum.files.wordpress.com/2013/07/search-prefix.png?w=300)](http://mdeinum.files.wordpress.com/2013/07/search-prefix.png)

Next open the environment variables screen. And edit (or when not already there add) the variable PATH and include the path which was determined in the previous step.


## Test your installation


After all the steps lets see if it worked. Type the following commands in the command line (when scaffolding the webapp you will be asked a couple of questions which libraries to include, answer them to your liking).

    
    npm install -g generator-webapp
    mkdir test
    cd test
    yo webapp
    grunt server


The last command starts a web server, opening a browser to [http://localhost:9000](http://localhost:9000) should should present you with the following.
[![install-success](http://mdeinum.files.wordpress.com/2013/07/install-success.png?w=300)](http://mdeinum.files.wordpress.com/2013/07/install-success.png)

We now have setup Yeoman and it runs behind a (corporate) proxy. Happy coding.
