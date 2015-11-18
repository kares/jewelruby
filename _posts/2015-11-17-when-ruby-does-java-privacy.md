---
title: When Ruby Does Java Privacy
tags: "jruby, java-integration, release, jruby-9000, jruby-9.0.3.0, jruby-9.0.4.0"
---

<div class="message">
  Make sure you have JRuby <b>9.0.4.0</b> or higher when trying out the samples.
</div>

Ruby has little respect for privacy or private state. There sure is "sugar" such as
`private` or `private_constant` but none of these really hide anything away much,
it's a very flexible language after all.
Java on the other hand gets a bit restrictive, with respect for `private` visibility
enforced by the compiler.

Let's see how JRuby fits the two together in the context of retrieving classes.

<!-- using the available Java support when -->

Obviously, public Java classes simply work as if it they were from Ruby-land :

{% highlight ruby %}
java.util.Map::Entry # Map.Entry
java.util.Map.module_eval do
  Entry.class_eval do
    def value?; ! value.nil? end
  end
end
{% endhighlight %}

Contrary, private or package-only Java classes are hidden as if they did not exist.

Ruby has `private_constant` to make sure constants are only visible in the declared
(module) context but nowhere else.

{% highlight ruby %}
module ObjectUtils
  NULL = Object.new
  private_constant :NULL
  def null?; self.equal? NULL end
end
ObjectUtils::NULL # NameError
{% endhighlight %}

Turns out this fits non-public Java classes nicely. Let's showcase an example.

Suppose we'd like to be able to detect whether a `java.util.List` instance is synchronized
(naively assuming list types are only those available with the standard collections framework) :

{% highlight ruby %}
# most are not synchronized e.g. java.util.ArrayList
java::util::List.module_eval do
  def synchronized?; false end
end
# old pal java.util.Vector synchronized since day 1
java::util::Vector.class_eval do
  def synchronized?; true end
end
# a java.util.Collections.SynchronizedList wrapper
# returned from java.util.Collections.synchronizedList
java::util::Collections::SynchronizedList # NameError !
# but accessing inside its enclosing class works :
class java::util::Collections
  def SynchronizedList.synchronized?; true end
end
{% endhighlight %}

As expected the non-public inner `SynchronizedList` in Ruby scripts behaves
similarly to how it does in raw Java - only accessible in the enclosing class and not being
part of the exported API.

<!--
Turing machines skip a cell when things find their way to fit together nicely.

ByteArrayInputStream#rewind

java.lang.System.method(:currentTimeMillis).arity ... tweet
-->
