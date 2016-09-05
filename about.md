---
layout: page
title: About
---
<!--
<p class="message">
  Hey! All content is objectively opinionated, feel free to disagree. Carry on!
</p>-->

<p class="message">
  JewelRuby's bits and pieces are composed by <a class="bold" title="Karol Buček" href="http://kares.org">@kares</a> ('s mostly left hand).
</p>
<!-- To our very best knowledge, no animals were hurt while doing so. -->

For some the <b><a href="https://www.youtube.com/watch?v=F4h6VCMTZ_0" target="_blank" class="hidden">Cat Down the Road</a></b>
desired to write down notes on occasional
experiments going on [here][1] [there][2] [and][3] [all-over][4]. Thus here we are - learning along!

[JRuby](http://jruby.org){:target="_blank"} the precious jewel, the most ambitious JVM language -
colliding 2 worlds like no other ever before, brought to us by the ever magnificent
[JRuby Team](https://twitter.com/intent/user?screen_name=JRuby){:target="_blank"}.

{% highlight ruby %}
require 'jruby/core_ext'
class Jay
  java_signature 'void greet(java.lang.String)'
  def greet(name)
    java.lang.System.out.println "Hello #{name}!"
  end
end
java_class = Jay.become_java! # a java.lang.Class
out = java.lang.System.out
def out.println(str)
  super str.sub('Hello', '{{ site.hello }}')
end
java_class.newInstance.greet 'World'
{% endhighlight %}

*p.s. ... feel the <span class="">J</span>oy and remember,
it's all <span class="redish bold">J</span>ust <span class="redish bold">Ruby</span>
(served with
  <span class="cafe-babe">
  <span class="x">0x</span><span class="c">C</span><span class="a">A</span><span class="f">F</span><span class="e">E</span><span class="b">B</span><span class="a">A</span><span class="b">B</span><span class="e">E</span>
  </span>
  ) !*

[0]: https://www.youtube.com/watch?v=F4h6VCMTZ_0
[1]: https://github.com/jruby/jruby-openssl
[2]: https://github.com/jruby/activerecord-jdbc-adapter
[3]: https://github.com/jruby/jruby-rack
[4]: https://github.com/jruby/jruby/commits?author=kares
[9]: http://www.stavacademy.co.uk/mimir/catproverbs.htm
