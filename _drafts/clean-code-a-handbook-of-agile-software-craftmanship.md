---
layout: post
title: Clean Code - A Handbook of Agile Software Craftsmanship
tags: [code, jekyll]
categories:
- blog
---

During an internship, I inherited a big project in insurance, featuring the
processing and visualization of natural-phenomenon-related risk data and decision
making of insurance contracts. The big words of contracts, forewriting, 


Quotes:

"If I don’t do what my manager says, I’ll be fired." Probably not. Most managers want the truth, even when they don’t act like it. Most managers want good code, even when they are obsessing about the schedule. They may defend the schedule and requirements with passion; but that’s their job. It’s your job to defend the code with equal passion.
You will not make the deadline by making the mess. Indeed, the mess will slow you down instantly, and will force you to miss the deadline. The only way to make the deadline - the only way to go fast - is to keep the code as clean as possible at all times.

The Boy Scouts of America have a simple rule that we can apply to our profession: Leave the campground cleaner than you found it. If we all checked-in our code a little cleaner than when we checked it out, the code simply could not rot.

Variables: You should name a variable using the same care with which you name a first-born child.

Naming conventions: Some interesting ones:

Use pronounceable names.

The length of a name should correspond to the size of its scope (for search-friendliness).

Instead of overloading constructors, use static factory methods with names that describe the arguments.
Pick one word for one abstract concept and stick with it. For instance, it’s confusing to have fetch, retrieve, and get as equivalent methods of different classes.

3. Functions: They should be short and only do one thing. So, another way to know that a function is doing more than “one thing” is if you can extract another function from it with a name that is not merely a restatement of its implementation. Try to keep the number of arguments as low as possible. 3 or more is a bad smell. Prefer Exception to returning error codes (often simplifies the code path).

4. Comments: Limit use of comments, which may wrongly express the code's behavior. Except legal comments, information, explanation of intent, clarification, warnings, TODO, JSDoc, public 
APIs

6. Objects vs Data Structures: Fundamental Dichotomy between objects and data structures:

Procedural code (code using data structures) makes it easy to add new functions without
changing the existing data structures. OO code, on the other hand, makes it easy to add
new classes without changing existing functions.

The complement is also true:
Procedural code makes it hard to add new data structures because all the functions must
change. OO code makes it hard to add new functions because all the classes must change.

{% highlight java %}

public class Square {
    public Point topLeft;
    public double side;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.141592653589793;
    public double area(Object shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square)shape;
            return s.side * s.side;
        }
        else if (shape instanceof Circle) {
            Circle c = (Circle)shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}

{% endhighlight %}

in comparison with

{% highlight java %}

public class Square implements Shape {
    private Point topLeft;
    private double side;
    public double area() {
        return side*side;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
    public double area() {
        return PI * radius * radius;
    }
}
{% endhighlight %}

The Law of Demeter: only talk to tour immediate friend, i.e. do not invoke a function of an object retrieved from another function.

{% highlight java %}
    x = shapePool.getCircle().getCenter().getX()
{% endhighlight %}



Classes: The Single Responsibility Principle (SRP) states that a class or module should have one, and only one, reason to change.


Procedural code (code using data structures) makes it easy to add new functions without changing the existing data structures.
OO code, on the other hand, makes it easy to add new classes without changing existing functions. (by overwriting a base class)
Procedural code makes it hard to add new data structures because all the functions must change.
OO code makes it hard to add new functions because all the classes must change.

Third party libraries: Sometimes useful to wrap third party libraries, so if they change only your wrapper implementation needs to change
Error handling is important, but if it obscures logic, it’s wrong. Throwing errors makes code cleaner than checking for status values throughout the code.

Tests: It is unit tests that keep our code flexible, maintainable, and reusable. The reason is simple. If you have tests, you do not fear making changes to the code! Test code should be treated like production code. Talks about learning an external API by writing tests for it (learning tests).

Concurrency is a decoupling strategy. It helps us decouple what gets done from when it gets done. Concurrency is hard. Concurrency can sometimes improve performance, but only when there is a lot of wait time that can be shared between multiple threads or multiple processors.

Nothing has a more profound and long-term degrading effect upon a development project than bad code. Bad schedules can be redone, bad requirements can be redefined. Bad team dynamics can be repaired. But bad code rots and ferments, becoming an inexorable weight that drags the team down.

#Personal opinion

Of course these principles are just guide lines and therefore come down to personal taste, and shouldn't be followed blindly. For instance, if you take his advice on refactoring any function that has more than three (!) lines, the functions become ridiculously small and harder to read in my opinion. Also, the book focuses on Java and OOP a lot, so you sometimes get advice that is mostly applicable to only the Java world.


I happened to be plagued  
Tattooed roof party *vinyl* freegan single-origin coffee wayfarers tousled, umami yr 
meggings hella selvage. Butcher bespoke seitan, cornhole umami gentrify put a bird 
on it occupy trust fund. Umami whatever kitsch, locavore fingerstache Tumblr pork belly
[keffiyeh](#). Chia Echo Park Pitchfork, Blue Bottle [hashtag](#) stumptown skateboard selvage 
mixtape. Echo Park retro butcher banjo cardigan, seitan flannel Brooklyn paleo fixie 
Truffaut. Forage mustache Thundercats next level disrupt. Bicycle rights forage tattooed
chia, **wayfarers** swag raw denim hashtag biodiesel occupy gastropub!

---

# It's all in the game.

## You come at the king, you best not miss.

### Be subtle with it, man. You know what subtle means?

VHS post-ironic cred **bespoke** banjo. Yr wayfarers literally gentrify, flexitarian fap 
dreamcatcher plaid cornhole Intelligentsia paleo. Beard try-hard direct trade, shabby chic 
Helvetica `look ma, I can code`. Lo-fi American Apparel tattooed [Vice](#) tofu, yr vinyl. 
Williamsburg butcher hella mumblecore fixie mlkshk, cliche wolf keytar mixtape kitsch banh mi 
salvia. High Life Odd Future *chambray* kale chips hoodie, cray pop-up. Helvetica narwhal 
iPhone try-hard jean shorts.

> This is a quote from someone famous about productivity


Syntax highlighting with Solarized theme.

{% highlight ruby %}
class User < ActiveRecord::Base
  attr_accessible :email, :name

  ... tons of other crap ...

end

{% endhighlight %}
