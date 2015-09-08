---
title: An Obvious Java Dispatch
tags: "jruby, java-integration, release"
---

<div class="message">
  Features mentioned here are available since JRuby <b>9.0.1.0</b> and <b>1.7.22<!--<sup>*</sup>--></b>
</div>

JRuby's ["Java Integration"][1] behind the magical `require 'java'` has been a mesmerizing
case ever since it was first introduced and stands as a reliable ground for a lot of
[very][2] [appealing][3] [libraries][4] out there.

Although most the time it's very intuitive e.g. `java.util.ArrayList.new(8)`.
Sometimes the finest details behind the curtains need real world use-cases to get right.
While in Ruby, where there's always a single method given a name, in Java method dispatch
accounts for static parameter types.
Add automatic [Ruby to Java][5] type conversion on top and things get
**ambiguous** [every][6] [so][7] [often][8] as JRuby tries to match the right Java method for you.

One such case that needs special care is matching a "proc-to-interface" method when
users implement an interface using a Ruby proc/block :

{% highlight ruby %}
# listFiles( FileFilter#accept(File) )
java.io.File.new('.').listFiles do |pathname|
  pathname.file? && pathname.can_read
end
{% endhighlight %}

The intent is clearly [`listFiles( FileFilter )`][9] as the `java.io.FileFilter`
functional interface requires an `accept(File pathname)` that takes a single argument.
But what if the block arity changes :

{% highlight ruby %}
# listFiles( FilenameFilter#accept(File, String) )
java.io.File.new('.').listFiles do |dir, name|
  java.io.File.new(dir, name).can_read
end
{% endhighlight %}

Obviously, in this case (the overloaded) [`listFiles( FilenameFilter )`][10]
method is meant since `java.io.FilenameFilter` prescribes a method with [2 parameters][11]
to be implemented.

The issue also revealed that matching wasn't 100% deterministic on non-unique method
matches in general (not just those ending up with a proc) and depended on JVM's reflected
method order, thus expect a lot of [confusing][8] "warnings" to be ironed out.
It's also why the change is not considered breaking for 1.7.x stable.

[1]: https://github.com/jruby/jruby/wiki/CallingJavaFromJRuby
[2]: https://www.elastic.co/products/logstash
[3]: http://shoesrb.com/
[4]: https://github.com/jruby/jrubyfx
[5]: https://github.com/jruby/jruby/wiki/CallingJavaFromJRuby#conversion-of-types
<!-- ambiguous -->
[6]: https://github.com/jruby/jruby/issues/2595
[7]: https://github.com/jruby/jruby/issues/3263

[9]: http://docs.oracle.com/javase/7/docs/api/java/io/File.html#listFiles%28java.io.FileFilter%29
[10]: http://docs.oracle.com/javase/7/docs/api/java/io/File.html#listFiles%28java.io.FilenameFilter%29
[11]: http://docs.oracle.com/javase/7/docs/api/java/io/FilenameFilter.html

[8]: https://github.com/jruby/jruby/issues/2865
