---
title: Jekyll Tag Cloud
category: Coding
tags: Jekyll Liquid Math
author: Josh
---

Building from [Thursday's post]({% post_url 2020-07-16-tags-organized-by-count-size %}), an alternative method for displaying tags by count had occurred to me: the classic "tag cloud". Displaying tags in a list (or, ideally, within a bunch or a circle) with the font size of the tag representing visually the relative importance of a tag to the blog compared to other tags. It accomplishes the same goal as a simple ordered-by-post-count list (showing what you care about), but it's visual and interesting.

I could think of some ways to do this (particularly after I spent all that time fiddling with tags), but I figured, why reinvent the wheel? I set to searching and immediately found a few things: [A](https://superdevresources.com/tag-cloud-jekyll/), [B](http://vvv.tobiassjosten.net/jekyll/jekyll-tag-cloud/), [C](https://jovandeginste.github.io/2016/05/04/add-a-tag-cloud-to-my-jekyll-site.html). (A doesn't even use Jekyll anymore, which I find kind of ironic.) They all can work (though maybe only in principal), but the mathematician in me was upset.

## The Goal

For me, a tag cloud should have a lower and upper bound on font-size that is (1) reasonable for displaying, (2) something that you chose, and (3) actually utilized. The tags should then be distributed between the lower and upper bounds, based on their relative sizes, by a method that is mathematically-derived. And I don't just mean "uses math".  It should be (4) obvious what the method is trying to accomplish and (5) work generally, for anyone, not specific blogs. The methods above all fail at least some of these five requirements.

## The Math

I immediately set to understanding what each poster was trying to get at with their various techniques for creating relative sizes for their tags to display, and some of the issues were glaring.

### A. SuperDevResources

In this case, they determine font-size by finding the size of the tag in `site.tags` (the number of posts with a particular tag), multiplying the size by 4, adding 80, and then using the resulting number as a percentage. Adding 80 is genius because it prevents an incredibly small font-size. Multiplying by 4 is arbitrary, but gives some increments to the tag sizes–their method for distributing them.

In this example, if I had 4 posts tagged with "bricks", I'd get the word "bricks" with `font-size: 96%`. They note this in the text, "By doing this, tags with just one post will be sized at 84%, those with 2 posts will have size of 88% and so on..." seemingly having no problem with how this series of numbers is going to go as they continue to write more and more. This method fails (1), (2), (3), and (5). Most importantly, there is no upper bound! 296 posts about "bricks" later and that 1280% font-size is going to hit them like a ton of tags.

### B. Tobias Sj&#0246;sten

He has a good goal: "count [a category's] usage. Divide that by the total number of posts". This gives the percentage of posts with a particular category, between 0 (exclusive) and 100 (inclusive). This is particularly easy to see when categories are limited to one per post. He then uses that percentage as the font-size, adding 70 to it that to help out. Having a font-size between 70% and 170% is reasonable.

This actually works in principal, only failing (in my opinion) (3). I actually don't mind that the lower bound isn't used because it could be so close to it that it doesn't matter. My issue is that in most use cases, the sizes will never be close to 70% or 170%, crowding everything around the middle, unintentionally.

I get the feeling that he never implemented this code, though. For one, he makes mention of what his blog "would probably" look like if he used it. And two... It's not what the code he provided does. `site.categories.size`, the denominator in his formula, is not the number of posts on the website; it's the number of categories used on the website. Imagine a scenario where you wrote a post about "cement" before you got into "bricks" and wrote a post about that. You have 2 posts and 2 categories. With his code, each category will display at 120% font-size (1 \* 100 / 2 + 70 = 120), which is reasonable. Imagine you got really into bricks, though, and kept writing until you got to 300 posts about them. Now you have 301 posts and 2 categories. The category "bricks" is now going to display at a whopping 15,070% (300 \* 100 / 2 + 70 = 15,070). (Oh gosh, that extra 70 really got me.)

So the "fix" for his code is to use `site.posts.size` instead. He doesn't have comments or I'm sure someone would have told him this since 14 May 2013. Also, he seems like a solid guy, and I know the order of operations doesn't matter between multiplication and division, but I still hate that he multiplies by 100 *before* dividing by the total.

### C. Yet Another SysAdmin trying to do his job... Jo Vandeginste

Is this is an improvement? He seems very confident that it is. He pointed this out on his post (and a comment on A, just for good measure).

To determine relative frequency, C modifies the method that B used in his code (not the method B *intended* to use), using the total number of tags instead of the number of *unique* tags as the denominator. Since there are more tags than unique tags, this would (generally) produce smaller percentages overall than B's method, pushing utilized font-sizes to the lower end of the range. He uses em's instead of percentages, though, so the differences are larger for smaller increments (and he multiplies the percent by 4 to increase the increments). He still gets a lower and upper bound, which in his case is 1em (exclusive) to 5em (inclusive). Honestly, sizes much larger than 2em through 5em are kind of large.

[Jo's own tag cloud](https://jovandeginste.github.io/tags/) stays between 1.06em and 1.54em to decent effect. There aren't very many posts, though, and you can easily imagine a scenario where the font-size gets huge or becomes undifferentiated. The thing is, this method passes (4) and in practicality will probably work generally (5), but whether it passes the other conditions is up for debate. (1) Is 5em an okay display size? (2) 5em is a chosen upper bound but it doesn't seem like it is the intention to actually see it used (3), so is the realized upper bound actually chosen?

It seems like this works *conveniently* rather than out of design.

## My Solution

To address (1) and (2), I decided that a range of 80% to 180% is reasonable (equivalent to .96em to 2.16em for my site, to compare to C). To address (3), I ensure that the tag with the least posts is 80% and the tag with the most posts is 180%. To address (4), I distribute the remaining tags spaced between the upper and lower bounds based on the sizes within the range. Division occurs on the range to determine the relative spacing, not the tags. And to address (5), I bail out if there is only 1 tag (to avoid dividing by zero), and post nothing if there are no tags (duh). Please tell me if I haven't thought of a blog scenario this doesn't work for!

The complication for this method is that you have to know which tags have the most and the least posts. As I discussed before, you can't do this directly. So I used the same method as before to first sort the tags by size. From there, I divide the 100% range by the range between the largest and smallest tag counts to determine the increment. The size of a tag (minus the smallest tag size as the size baseline) times the range distributes the tags throughout the range, then add 80% for the percentage baseline. This ensures that the smallest tag will be 80% and the biggest will be 180%.

As a bonus, I did not rely on Jekyll not sorting the tags I use alphabetically. I used the sample filter to randomly order the tags within the cloud. This will reorder when the site is rebuilt and will be random each time. For some reason, this did not work without first sorting `site.tags` first ([still looking for a reason why](https://stackoverflow.com/questions/62964510/why-does-jekylls-sample-filter-not-work-on-the-site-tags-variable-array)), so a few extra lines.

As an include:

{% highlight liquid %}{% raw %}{% assign n = site.tags | size %}
{% if n > 0 %}
  <h2>Tag Cloud</h2>
  <div class="threeColumns">
    <div class="tagCloud">
      {% capture tags %}
        {% for tag in site.tags %}
          {{ tag[1].size | minus: 10000 }}#{{ tag[0] }}#{{ tag[1].size }}
        {% endfor %}
      {% endcapture %}
      {% assign tagsBySize = tags | split: " " | sort %}
      {% if n > 1 %}
        {% assign biggestTag = tagsBySize | first | split: "#" %}
        {% assign smallestTag = tagsBySize | last | split: "#" %}
        {% assign range = biggestTag[2] | minus: smallestTag[2] %}
        {% assign increment = 100 | divided_by: range %}
        {% assign sortedTags = site.tags | sort %}
        {% assign sampleTags = sortedTags | sample: n %}
        {% for tag in sampleTags %}
          <a href="{{ site.baseurl }}/blog/tags/{{ tag[0] | slugify }}" style="font-size: {{ tag[1].size | minus: smallestTag[2] | times: increment | plus: 80 }}%">{{ tag[0] }}</a>
        {% endfor %}
      {% else %}
        {% assign tag = tagsBySize | split: "#" %}
        <a href="{{ site.baseurl }}/blog/tags/{{ tag[1] | slugify }}" style="font-size: 130%">{{ tag[1] }}</a>
      {% endif %}
    </div>
  </div>
{% endif %}
{% endraw %}{% endhighlight %}
