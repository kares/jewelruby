---
title: Better Closures for (Functional) Java
tags: "jruby, java-integration, release, jruby-9000, jruby-9.1.0.0"
description: "JRuby improvements for scripting Java 8's functional APIs using Ruby closure blocks."
keywords: "jruby, java-integration, lambda, block, closure, FunctionalInterface, java.util.functional, java.util.stream, sort"
image:
  path: /public/jewel_128.png
  twitter: /public/jewel_128.png
  height: 128
  width: 128
---

One of the neat features of JRuby's [Java Integration][0] is the ability to
dynamically implement interfaces using a block a.k.a. the neat [proc-to-interface][1] conversion.

{% highlight ruby %}
module CalculatorMachine
  def self.calculate(n); (1..n).reduce(1, '*') end
end

java.util.concurrent.ForkJoinPool.class_eval do
  # help matching due overload : submit(java.lang.Runnable)
  java_alias :exec, :submit, [java.util.concurrent.Callable]
end

pool = java.util.concurrent.ForkJoinPool.common_pool
pool.exec do # submit(java.util.concurrent.Callable)
  pool.exec do # spawn a sub-task from a task
    res = CalculatorMachine.calculate(1000)
    puts "sub1 calculated: 1000! ~ 10^#{res.to_s.size}"
  end
  pool.execute do # another execute(java.lang.Runnable)
    res = CalculatorMachine.calculate(10_000)
    puts "sub2 calculated: 10_000! ~ 10^#{res.to_s.size}"
  end
  res = CalculatorMachine.calculate(100)
  puts "task calculated: 100! ~ 10^#{res.to_s.size}"
end

res = CalculatorMachine.calculate(100_000)
puts "main calculated: 100_000! ~ 10^#{res.to_s.size}"
{% endhighlight %}

No need to have a class (or 2) <span title="implementing">'including'</span> the
`java.util.concurrent.Callable` for the [submit][2] method, since JRuby passes
the block as an implementation of the (last) interface argument.

Great way to ease scripting over Java APIs that has been available since ancient
JRuby times.
However, there's been [some][3] collisions notably around on Java 8's novel stream
support where Ruby block semantics otherwise match up very well with functional interfaces.

The problem is that proc-to-interface dispatch logic is using `method_missing`,
thus a potential naming conflict (either from a core extending gem or built-in method)
fails to execute the intended logic e.g. a block-based implementation for [`Predicate#test`][4]
ends up calling Ruby's [`Kernel#test`][5] instead of the actual body (the bellow
example would fail).

<span id="stream-example">
{% highlight ruby %}
numbers = java.util.Arrays.asList 1, 2, 3, 4, 5, 6, 7, 8

supplier = -> { java.util.ArrayList.new } # Supplier<R>
accumulator = -> (c, e) { c << e } # BiConsumer<R,? super T>
combiner = -> (c1, c2) { c1.addAll(c2) } # BiConsumer<R,R>

three_even_squares = numbers.stream().
  filter( ->(n) { puts "filtering #{n}"; n % 2 == 0 } ).
  map( ->(n) { puts "  mapping #{n}"; n * n } ).
  limit(3).collect(supplier, accumulator, combiner).
  to_s # an ArrayList of "[ 4, 16, 36 ]"
{% endhighlight %}
</span>

This has been improved in JRuby 9.1 to always bind concrete (implemented) interface
methods, besides `method_missing`, to make sure the intended block body is executed
regardless of the inherited runtime state. Looks and feels just like Java these days.

<div class="message">
  Make sure you have JRuby >= <b>9.1</b> when scripting functional Java, like above.
</div>

[0]: http://kares.org/jruby-ji-doc/_index.html
[1]: https://github.com/jruby/jruby/wiki/CallingJavaFromJRuby#closure-conversion
[2]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html#submit-java.util.concurrent.Callable-
[3]: https://github.com/jruby/jruby/issues/3475
[4]: http://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html#test-T-
[5]: http://ruby-doc.org/core-2.3.1/Kernel.html#method-i-test
