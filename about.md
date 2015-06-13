---
layout: page
title: About
---

<p class="message">
  Hey! All content is objectively opinionated, feel free to disagree. Carry on!
</p>

JewelRuby's bits and pieces are composed by hand (mostly left) by [kares](http://kares.org).

For some time I wanted to write down notes on the occasional experiments that I
am seeing or doing, [here][1] [and][2] [there][3]. As these are generally at the
"native" level (sometimes in <span class="redish">JRuby</span> itself) it's not
something to feed the docs (who will read those anyways esp. when mimicking not-so
documented behavior), a (b)log diary seemed the best.

[JRuby](http://jruby.org) the precious jewel, the most ambitious JVM language -
colliding 2 worlds like no other had ever before, brought to us by the magnificent
[JRuby Team](https://twitter.com/intent/user?screen_name=JRuby){:target="_blank"}.

<!--
## Setup

```
jruby -S irb
> require 'java'
 => false
> $answer = 24
 => 24
> scope = org.jruby.embed.LocalContextScope::SINGLETHREAD
 => #<Java::OrgJrubyEmbed::LocalContextScope:0x338ff9a1>
> container = org.jruby.embed.ScriptingContainer.new scope
=> #<Java::OrgJrubyEmbed::ScriptingContainer:0x25af578c>
> container.runScriptlet 'ANSWER = 42'
=> 42
> ANSWER
=> 24
``` -->

*p.s. ... have <span class="redish">J</span>oy and remember,
it's <span class="redish">J</span>ust <span class="redish">R</span>uby!*

[1]: https://github.com/jruby/jruby-openssl
[2]: https://github.com/jruby/activerecord-jdbc-adapter
[3]: https://github.com/jruby/jruby-rack
