---
title: Aggregating posts with jekyll using a custom variable
description: Setting up a custom variable on your posts that makes it really easy to aggregate posts on a page
project: site-overhaul
layout: post
image: /assets/images/jekyll.jpg
---

Jekyll makes it super easy to create pages and posts and then loop over them to render lists of them. There are a few commands that you can utilize in Liquid to narrow down that list for only rendering a subset of posts or pages. Those are `assign` and `where` statements.

The `assign` statement lets you create new variables that can be referenced in that page as well as any include that happen on that page. And the `where` statement is a way to narrow down a list to a subset of that list for use in the `assign` statement.

To use the assign statement you simply do

{% highlight liquid %}
{% raw %}
{% assign my_posts = site.posts %}
{% endraw %}
{% endhighlight %}

Now the variable `my_posts` will contain all of the site's posts and you can loop over it just as you would with site.posts

{% highlight liquid %}
{% raw %}
{% for post in my_posts %}
  ...
{% endfor %}
{% endraw %}
{% endhighlight %}

A `where` statement lets you narrow down a list during an assignment

{% highlight liquid %}
{% raw %}
{% assign my_posts = site.posts | where:'title', 'Awesome Title' %}
{% endraw %}
{% endhighlight %}

This will make `my_posts` a list of posts where the title of the post is *Awesome Title*.

A lot of things I work on in my free time are parts of a larger project I'm working on. These are represented by my project pages which are an aggregation of all posts related to a specific project. `assign` and `where` enabled the posts to be scoped to a specific project. For example this post is scoped to the `Website Overhaul` project.

Each project I start, there is a corresponding "project" page. That page will then render a list of all posts that are about that project.

{% highlight liquid %}
{% raw %}
<!-- give me just the posts that have a variable named project, that match this project page's project name -->
{% assign tiles = site.posts | where: 'project', page.project %}
{% include tiles.html %}
{% endraw %}
{% endhighlight %}


