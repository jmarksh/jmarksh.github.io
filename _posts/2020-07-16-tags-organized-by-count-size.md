---
title: Tags Organized by Count (Size)
category: Coding
tags: Jekyll Liquid
author: Josh
---

Jekyll is blog-aware and it runs on Liquid (plus Markdown, HTML, CSS, etc.). Liquid is a templating language, not a fully-fledged programming language. What that boils down to is Jekyll+Liquid can make a good website/blog, but it might lack a few things if you try to go super complex.

Making a complex array, that is, an array with an array in it or an array where elements have attributes simply isn't possible in Liquid. Liquid can *handle* such arrays (use all the YAML and JSON files you want), and it can even make those kinds of arrays on its own (we'll get to that), but **you** can't make those arrays with Liquid. And this stands in the way of organizing tags on your blog by post count.

## Categories Versus Tags

Jekyll is automatically aware of categories and tags, things most blogs have. You can label all of your posts from the get-go, and Jekyll will know what to do with them. By design, categories are similar to folders in which posts are filed, and tags are sticky-notes you can attach to the posts, noting the more-specific topic or, importantly, topic***s***.

Other people who use Jekyll have wanted multiple categories or only use tags instead, and by all means, have your fun, but the default setup is already a problem.

## Posts Served by Tags or Categories

With a few quick lines of code, you can pull groups of posts out to display that are connected by category or tag, which, obviously you should be able to. It's one of the first things people do when they start a blog, and [they even tell you exactly how to do this on the Jekyll website](https://jekyllrb.com/docs/posts/#tags-and-categories):

```
{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
```

The same deal works for categories if you just replace `tags` with `categories` above. That's because the built-in functionality of Jekyll creates a complex array out of the `site.tags` or `site.categories` variable. That is, each element in the array is an array where the first element is the tag (or category) name and the second is an array of the posts with that tag. Mathematically (not actually) this looks like:

```
{ {"Bananas", {#, #, #} }, {"Zuchinis", {#, #, #, #}} }
```

You have four posts about zuchinis and only three about bananas. It's okay, we overplanted zuchinis, too.

In Liquid, you can even run this through a sort filter and it will sort by the first element in each inner array, the tag/category, so that you get an alphabetical list. So Liquid is fine *dealing with* complex arrays, but you cannot create one. You can't `assign` an array to an array index position.

Why is this important?

## Categories Organized by Count (Size)

There may come a time when you want to indicate to your audience what you're into. Rather than simply telling them what you're into, you could demonstrate it by showing off what you like to write about. And you'd do that by creating a list of categories of your posts organized in descending order by which categories have the most posts. You're clearly into the things you write about most! Ahh, but how!?

We literally just created a complex array that had all of the posts in an array right next to the category label (`site.categories`). You can find out the size of the post array by calling `site.categories[Zuchinis].size`, and you'd be forgiven for thinking that becuase you could get that information, you could use it to do what you wanted to! If only you could organize that complex array by the *size* of the *second* element.

Luckily, you've limited yourself to only one category per post (very important), so there's an easy way around this:

```
{% assign categoriesBySize = site.posts | group_by: "category" | sort: "size" | reverse %}
{% for category in categoriesBySize limit:5 %}
  <span class="smaller">{{ category.size }} Posts</span><br /><a href="{{ site.baseurl }}/blog/category/{{ category.name | slugify }}">{{ category.name }}</a><br /><br />
{% endfor %}
```

Using another nifty filter that comes built in (`group_by`), you can create a complex array of your posts, categorized by their category **and** it even gives you the size of the post array *as an element* in the complex array. Notably, it also associates each position in the array with a name, so you can refer to it. Again, mathematically (not actually), this looks like:

```
{ {name="Bananas", items={#, #, #}, size=3}, {name="Zuchinis", items={#, #, #, #}, size=4} }
```

So you could sort by `name` or, like above, `size` to accomplish different things. You could use this as an alternative method for generating the list of posts by category in a more intuitive way (in my opinion, anyway).

## The Point

It was very important, though, in the last section that you only had one category per post. But say you had tagged the above posts variously as "breakfast", "dinner", "snack", and "baby". Those tags would pile up on a post about breakfast baby food. And instead of neatly separated `name` elements from the above array, your names would also be an array `name={"breakfast", "baby"}`.

You would not be getting a list of posts with neatly separated tags, you'd be getting posts separated by all the *sets* of tags you used.

As it turns out, this seemingly basic thing just isn't possible.

That's okay, really. The language wasn't *designed* to do this, so the fact that it can't shouldn't be shocking, even if it does cause you to waste a lot of time coding and re-coding *BECAUSE IT JUST SHOULD WORK!*

Because of that, people have created plugins to accomplish the task. One of which, [jekyll-archives](https://jekyllrb.com/docs/plugins/your-first-plugin/) is right at the top of the page about plugins.

But you don't need a plugin.

## The Solution

There is a way to solve this without a plugin. It was posted to [Stack Overflow](https://stackoverflow.com/questions/24700749/how-do-you-sort-site-tags-by-post-count-in-jekyll#answer-24744306) by [Christian Specht](https://stackoverflow.com/users/6884/christian-specht) in response to someone asking about a plugin to solve this same problem. It's been described as "slick" (by someone else) and "hacky" (by me), but it works without a ton of code. A commenter, [Mincong Huang](https://stackoverflow.com/users/4381330/mincong-huang) even made it better with a seemingly small note. The code, modified to work on my site:

```
{% capture tags %}
  {% for tag in site.tags %}
    {{ tag[1].size | minus: 10000 }}#{{ tag[0] }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign tagsBySize = tags | split: " " | sort %}
{% for tag in tagsBySize limit:5 %}
  {% assign tagArray = tag | split: "#" %}
  <span class="smaller">{{ tagArray[2] }} Posts</span><br /><a href="{{ site.baseurl }}/blog/tag/{{ tagArray[1] | slugify }}">{{ tagArray[1] }}</a><br /><br />
{% endfor %}
```
Essentially, you create a string that contains the number of posts in a tag formatted in a sortable way, the tag name, and the number of posts in a readable way. You sort while that is all one string, and then you break it into pieces to use the parts of the string as you need to.

The sortable format is the slick part. If you use the post count in a string and sort, "10\#Baby" will come before "2\#Breakfast". The original poster solved this by adding 1000 to create leading zeroes, just how we do with filenames to keep them in order (e.g. image002 &gt; image010 not image10 &gt; image2). In that way, 1002\#Breakfast will come before 1010\#Baby. You then just have to reverse the order so that the highest number comes first (that is, so that 1010 is higher than 1002).

This created a problem, though. If two tags have the same number of posts, then reversing the sort will reverse alphabetize the names: 1010\#Baby &gt; 1002\#Dinner &gt; 1002\#Breakfast. If you *subtract* 1000 (or in my case, 10000, to give myself more leeway), you get the correct order without having to reverse, so the alphabetizing is preserved: 9990\#Baby &gt; 9998\#Breakfast &gt; 9998\#Dinner.

*Et voil&#0225;!* That only took all night.
