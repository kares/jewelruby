---
layout: page
title: About
---
<!--
<p class="message">
  Hey! All content is objectively opinionated, feel free to disagree. Carry on!
</p>-->

<p class="message">
  JewelRuby's bits and pieces are composed by hand (mostly left) by <a class="bold" href="http://kares.org">@kares</a>
</p>
<!-- To our very best knowledge, no animals were hurt while doing so. -->

For some the Cat Down the Road wanted me to write down notes on the occasional
experiments going on [here][1] [there][2] [and][3] [all-over][4]. And here we are ...

[JRuby](http://jruby.org) the precious jewel, the most ambitious JVM language -
colliding 2 worlds like no other had ever before, brought to life by the magnificent
[JRuby Team](https://twitter.com/intent/user?screen_name=JRuby){:target="_blank"}.

*p.s. ... have <span class="">J</span>oy and remember,
it's <span class="redish bold">J</span>ust <span class="redish bold">Ruby</span>
(and some
  <span class="cafe-babe">
  <span class="x">0x</span><span class="c">C</span><span class="a">A</span><span class="f">F</span><span class="e">E</span><span class="b">B</span><span class="a">A</span><span class="b">B</span><span class="e">E</span>
  </span>
  )!*

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
  super str.sub('Hello', 'Hoja')
end
java_class.newInstance.greet 'World'
{% endhighlight %}

[1]: https://github.com/jruby/jruby-openssl
[2]: https://github.com/jruby/activerecord-jdbc-adapter
[3]: https://github.com/jruby/jruby-rack
[4]: https://github.com/jruby/jruby/commits?author=kares
