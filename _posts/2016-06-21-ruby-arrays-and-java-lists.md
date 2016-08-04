---
title: Ruby Arrays and Java Lists
tags: "jruby, java-integration, release, jruby-9000, jruby-9.1.0.0"
description: "Java integration improvements to java.util.List and how lists act like Ruby Arrays in JRuby"
keywords: "jruby, java-integration, Array, ArrayList, java.util.ArrayList, LinkedList, java.util.LinkedList, sort, ruby_sort"
image:
  path: /public/jewel_128.png
  twitter: /public/jewel_128.png
  height: 128
  width: 128
---

<div class="message">
  Code samples target JRuby <b>9.1.0.0</b>+ and might not work in earlier versions.
</div>

[Java Integration][0] received a serious boost in [JRuby **9.1**][1] and not just in
terms of [actual speed][9]. There's more consistency, effectiveness and edge case
handling going on.

Let's look at Java's most used collection [`java.util.ArrayList`][2]. Java, being
a static language, uses interfaces (to specify a common "quack-like" behavior for
objects) and array lists (conform) implement the [`java.util.List`][3] interface.

Unlike Java or other languages, Ruby does not have a separate array and list type,
instead `Array` acts like a mutable (unless `freeze` happens) list of elements.

In fact, when a Ruby `Array` is making it into a Java method it will do top-notch
when the parameter is of a list type ([`RubyArray implements java.util.List`][4]).

Likewise, Java [lists][3] behave like Ruby's array type, esp. `java.util.ArrayList` :

{% highlight ruby %}
list = java.util.ArrayList.new [1, 2]
list.get(0) # 1
list[-1] # 2
list[10] # nil
list[3] = 0 # [1, 2, nil, 0]
list.index(0) # 3
list.rindex { |e| e.nil? } # 2
list[-3, 2] # [2, nil] sub-list
list.first(2) # [1, 2] sub-list
list.last(2) # [nil, 0] sub-list
{% endhighlight %}

Compared to the list's native [`get(int)`][7] the `[]` method does not raise on out of
bounds and negative indexes and has all the functionality of an `Array#[]`.
The array-like behavior is added to the `java.util.List` interface - acts as a module
in Ruby land. The above snippet thus works for any type (implementing the interface)
e.g. [`java.util.Stack`][5] or [`java.util.LinkedList`][6].

Internally, `index` and `rindex` fall-back to list iterators for non random access list,
compared to always looping using (ineffective) `get` positioning previously.

{% highlight ruby %}
list = java.util.LinkedList.new
list << 'c'; list << 'aaa'; list << 'bb'
list2 = list.dup # cloned LinkedList (same elements)
list2.sort! { |e1, e2| e1.size <=> e2.size }
java.util.LinkedList.class_eval { alias sort ruby_sort }
list = list.sort
list.class == java.util.LinkedList
list != list2 # true ['aaa', 'bb', 'c'] != ['c', 'bb', 'aaa']
{% endhighlight %}

The later example show how Java collections `dup` very much like Ruby objects.
There's `alias sort ruby_sort` to handle the native [`sort`][8] method added to lists
since Java 8. JRuby favors existing Java methods (they're coming from the class while
`sort` is on the `java.util.List` mixin), but provides a convention for dealing with
common ambiguities. Using the `ruby_sort` name the Ruby-like manner might be brought in as needed.
Lastly, the Ruby `sort` returns the same object type as its invoked on, which wasn't
the case before JRuby **9.1**.

Explaining ambiguities, the first example (literally) won't work, since there's
the [`getFirst()`][10] instance method on `java.util.LinkedList` which is recognized
as a getter (Java reader) and maps to `first()` without any method parameters :

{% highlight ruby %}
list = java.util.LinkedList.new [1, 2]
list[3] = 0 # [1, 2, nil, 0]
java.util.LinkedList.class_eval { alias first ruby_first }
list.first(2) # [1, 2] sub-list
list.ruby_last(2) # [nil, 0] sub-list
{% endhighlight %}

Check-out [the](http://kares.org/jruby-ji-doc) [**doc**][0] for more about upgrades everyday Java types are eligible to.

[0]: http://kares.org/jruby-ji-doc/_index.html
[1]: https://github.com/jruby/jruby/wiki/JRuby-9.1.0.0-Release-Notes#java-integration
[2]: http://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html
[3]: http://kares.org/jruby-ji-doc/Java/java/util/List.html
[4]: https://github.com/jruby/jruby/blob/5ab0852/core/src/main/java/org/jruby/RubyArray.java#L94
[5]: http://docs.oracle.com/javase/8/docs/api/java/util/Stack.html
[6]: http://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html
[7]: http://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html#get-int-
[8]: http://docs.oracle.com/javase/8/docs/api/java/util/List.html#sort-java.util.Comparator-
[9]: https://gist.github.com/kares/8339169c60affddc31897a141c826ff0
[10]: http://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html#getFirst--

<!-- https://github.com/jruby/jruby/wiki/CallingJavaFromJRuby -->
