---
layout: post
title:  "Post With A Code Snippet"
date:   2014-12-13
---
You'll find this post in your `_posts` directory - edit this post and re-build (or run with the `-w` switch) to see your changes! To add new posts, simply add a file in the `_posts` directory that follows the convention: YYYY-MM-DD-name-of-post.ext. $$\sigma x_a$$

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
{% endhighlight %}

----------
Just to test the R code as follows:

{% highlight r %}
> library(pryr)
> x <- 1:10
> c(address(x), refs(x))
> [1] "0x103100060" "1"
{% endhighlight %}

----------
There are generally two ways to format strings in Python. Check them out there!

**Use Format**
{% highlight python %}
my_name = 'Michael';
print ("Hello, my name is {name}".format(name=my_name));

"Hello, my name is Michael"
{% endhighlight %}

**Use  ```%``` in older version of Python**
{% highlight python %}
name = "Mike";
print "Hello %s" % (name);

Hello Mike
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
