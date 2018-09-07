---
layout: post
title: Popular misconceptions and errors in Javascripts
tags: [jekyll, syntax]
categories:
- blog
---

# Data comparison

The difference between two comparison operators:
- `==`: the equality operator coerces data type and compare data
- `===`: the identity (strict equality) operator strictly compares data without type coercion

Some example:
{% highlight js%}
0 == '0'; // true
0 === '0'; // false

0 == false; // true
0 === false; // false
{% endhighlight %}

Identity operator is considered safe since users just need to deal with data of the same types. For simplicity and readability of code, this should be the preferable choice.
On the other hand, equality operator behaves in a very different way due to its conversion mechanism, resulting in some unexpected results:

{% highlight js%}
0 == ''; // true
0 == ' '; // true
0 == '\n\t\r'; // true

0 == false; // true
false == 'false'; // false

null == undefined; // true

[] == true; // false
[1] == true; // true
[''] == true; // false
{% endhighlight %}