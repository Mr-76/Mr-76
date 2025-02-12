---
layout: post
title:  "Welcome to Jekyll!"
date:   2025-01-21 16:59:28 -0300
categories: jekyll update
---
This is simply a doc about how to setup a docker swarm with load balacing with 4 single board computers, a bananapi m2 zero 2 w, a orange pi zero2 w, a raspberry pi 3b and a raspbery pi zero w.
All of which are in a direct liquid coling with mineral oil (for me its cool i like it...)

all of them use armbiam as the linux distro, 2 of them are aarch64 the other 2 32 bits only, didnt find a good way to use as a kubernets cluterso now trying as a docker swarm cluster....

user usermod -aG docker $USER to make the user acess docker without sudo... 

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
