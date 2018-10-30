---
layout: post
title: My first step into Python programming
tags: [jekyll, syntax]
categories:
- blog
---

# Characteristics
- Readability: easy to learn, read and maintain thanks to its concise syntax and structures
- Portability: multi-platform
- Brevity: concise syntax fostering reading, typing,
- Generality: applicable for web applications, Desktop GUI, scripting, multimedia processing, data science, machine learning
- Interactiveness: CLI tool for direct interaction
- Extendability:
- Scalability: 
- Error checking: runtime
- Cost:
- Type: it is either interpreted language facilitates code deployment or byte-code compilable for large applications 
- Weak/strong typing, type checking: weak, allowing dynamic data coupling
- Object oriented aspect:
- Programming paradigm: functional programming, procedural programming
- Garbage collection

# Data types
- String
- Number
- Tuple
- Dictionary

## Number types
- int
- long
- float
- complex


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