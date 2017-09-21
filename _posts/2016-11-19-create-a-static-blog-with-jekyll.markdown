---
layout: post
title:  "Create a static blog with Jekyll"
date:   2016-11-19 11:00:11 +1200
thumbnail: jekyll.png
---
# Blog concerns
I had considered making a blog a few times before.
As a red-blooded techie, I of course wanted to host it myself so that I could tinker and tweak and learn.
This presents problems, however as most blog platforms require a great deal of care and feeding.
Popular platforms such as [Wordpress][wiki-wordpress] require almost constant updates to address the shortcomings in their [security][wp-vulns] (partly due to being written in PHP by people that don't need to consider security as a priority).
There are alternative projects, written in languages such as NodeJS, Ruby and Python (including Django projects).
Most of these are in various states of completion or support which is concerning when the potential for vulnerabilities is so high.

# Enter Jekyll
The Jekyll project is supported by GitHub for use on their "GitHub pages" for hosting websites for projects.
The idea is that you can write website articles in markdown and use Jekyll to process them into a series of plain HTML files.
You get the benefits of a dynamic CMS, without having to set up a (potentially) vulnerable platform and a database.

Naturally, such an idea greatly appeals to me as you can version you website directly in Git and not have to worry about an external database.

<figure>
	<img src="{{ site.baseurl }}/assets/jekyll.png" alt="Jekyll Logo">
	<figcaption>
		Jekyll allows plain HTML to be generated from a group of markdown files
	</figcaption>
</figure>

## Getting setup
Unfortunately, Jekyll is a Ruby project.
As Ruby is a trendy language I have no interest in getting to know, I was seeking an alternative to having to install a Ruby runtime on my machine to try it out.
>Enter Docker.

At some stage in the future, I will put up a post with the details of getting Docker setup and how and why you should be using it.
To setup a base project with Jekyll you can run the following command:

`docker run --rm -v $PWD:/srv/jekyll jekyll/jekyll:page jekyll new blog`

Note that I use the `pages` tag for the jekyll repository, as this will use a version of jekyll that is as close to the GitHub pages instances as possible.
Interoperability is an important consideration for me, so this just makes sense.
This will create the templates in a new directory `blog` where you can customise to your heart's content.
The most important part of the base template is the `_config.yml` file (where site level config is done) and the `_posts` directory, where your markdown blog posts are stored.

## Creating a post
Simply copy the example blog post in the `_posts` directory and throw in some of your markup.
This is exactly the process I used to create this very post.
Once you are content with your content, you can run the built in web server for Jekyll to test out your new site.
Make sure you run this next command from inside the directory that was created by the last command.

`docker run --rm -it -v $PWD:/srv/jekyll -p 8080:4000 jekyll/jekyll:pages jekyll serve --watch`

[wiki-wordpress]: http://en.wikipedia.org/wiki/WordPress
[wp-vulns]: https://wpvulndb.com/
