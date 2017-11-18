---
title: Tips for developing a Blender addon
description: Some tips for developing a Blender addon
image: /assets/images/blender-addon-tips/blender.jpg
layout: post
---

Lately I've gotten into the world of working on python scripts and addons in the open source 3d software Blender. I've been learning 3d animation for a while now but this has been my first attempt at writing any code that worked with Blender. In doing so I came across a few things that cause a little bit of pain at first but figured out ways to relieve the pain. So here are a few tips that I came across.

### Symlink your addon
When I write code, I like to keep my code broken out into sevaral files instead of it all being in the same file. Of the few addons that I've looked at, the files are super long because most of the code is in one file. Because of this I didn't want to use the script editor inside Blender, instead I will write my code in whatever editor I desire. But when I want to run my addon, I would have to zip up my addon and install it in Blender. That's a lot of work for every little change. Instead what we can do is just symlink from the Blender installed addon directory into your addon project directory. A symlink is essentially a marker that can either point to a file or folder somewhere else on the file system, such that when you navigate to the symlink you are taken to the other folder, or in the case of a file, when you interact with it, it is actually interacting with the original file.

The first step to get this working is to figure out where Blender installs 3rd party addons. The way I did that for myself was to download an addon such as the Blender Cloud addon, [https://cloud.Blender.org/services](https://cloud.Blender.org/services), using the "Install Add-on from File..." button in the Add-ons User Preferences tab.

![User preferences Install addon from file](/assets/images/blender-addon-tips/install-from-file.jpg)

Once it is installed you can open it up and it will tell you where it was installed into.

![Blender Cloud addon install location](/assets/images/blender-addon-tips/install-location.jpg)

Now from this point how you create the symlink will depend on whether you are on windows or a unix system.

On Windows you can follow these instructions where `Link` is the name of the marker or symbolic link you want to create and `Target` is the path to your addon project.[https://www.howtogeek.com/howto/16226/complete-guide-to-symbolic-links...](https://www.howtogeek.com/howto/16226/complete-guide-to-symbolic-links-symlinks-on-windows-or-linux/)

And on Unix you can follow these instructions where `source_file` is your addon project directory and `myfile` is the name of the marker or symbolic link you want to create.
[https://kb.iu.edu/d/abbe](https://kb.iu.edu/d/abbe)

If it all worked properly, you should be able to restart Blender and see your addon in the list of addons.

### Reload all your modules
So I have several module files in my addon, this poses a different problem for Blender. When you install your addon, Blender will run all the code at the root of your entry point file as well as any file that gets imported in the process. When this import happens python will make a cache of all of the files that were imported. So when you make a change in one of the module files, that change won't show up in Blender, even when you toggle the addon. When your addon runs again, it will use the cached version from the first run.

It's not ideal to have to restart Blender whenever you make a change. Python gives us a way to get around this problem in the form of the package `importlib`. `importlib` is a package that gives you access to the functionality available to you in the `from/import` statements. `importlib` has a method called `reload` that when passed a module, will reload that module. The caveat is that in your entry point python file, you have to import all the modules in your addon. While there is probably a better way to do this, this is how I've done it.

{% highlight python %}
import importlib
from . import my_module
from .operations import my_operation

importlib.reload(my_module)
importlib.reload(my_operation)
{% endhighlight %}

Now when you make changes in a module and then save your addon entry file (You have to save your addon entry file, `__init__.py`, in order for Blender to see that there was a change to your addon and reload), you should be able to just toggle your addon off and on again and your changes should show up.

It would be great if there was just a command line flag you could pass into python or Blender to perform a reload whenever it does and `from/import` statement, which would remove the need to do this.
