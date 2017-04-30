---
title: Slicing an array for assignment in Liquid
description: Need to assign a new variable with a subset of an array. Insert slicing.
project: site-overhaul
layout: post
image: /assets/images/tiles.jpg
---

While liquid has a way to ensure that a for loop will exit once it hits a certain iteration. You may need to slice an array of items outside of a for loop. After some time on Google, I couldn't find any straight forward way of getting a subset of an array in liquid. I came across a site with some unofficial documentation about strings in liquid. It said it was possible to slice strings. Knowing that liquid ran in ruby and that you can treat strings much like an array it made me wonder if you could slice an array in a similar way as how it described to do it with a string. Sure enough you can.

{% highlight liquid %}
{% raw %}
{% assign first_posts = site.posts | slice:0, 6 %}
{% endraw %}
{% endhighlight %}

The slice works just like it does in most programming languages. The first number is the index in the array that you wish the slice to start at, and the second is the number of items from that index you would like to be sliced into your new array.

While getting my website set up using [jekyll](https://jekyllrb.com) I came across a situation where I wanted to make one of the pieces of the template a little more modular. The theme I'm using, [https://github.com/andrewbanchich/forty-jekyll-theme](https://github.com/andrewbanchich/forty-jekyll-theme), has an include called "tiles" which are the tile links for each post on my home page. In order to get the right data for these tiles, the code was looping over all of my posts and using the `limit` function in liquid to ensure the loop would only run 6 times.

![Posts laid out in tiles on my home page](/assets/images/tiles.jpg)

I wanted to use this same design on my project pages so it would be a cohesive experience of how post links are rendered across my site. That meant modifying the include as well as the home page template to not use the for loop anymore. I ended up using slice to make it easy for me to still only render 6 items and use the new, more modular template.

{% highlight liquid %}
{% raw %}
{% assign tiles = site.posts | slice:0, site.tiles-count %}
{% include tiles.html %}
{% endraw %}
{% endhighlight %}