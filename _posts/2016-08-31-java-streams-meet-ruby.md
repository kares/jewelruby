---
title: Java Streams Meet Ruby
tags: "jruby, java-integration, release, jruby-9000, jruby-9.1.3.0"
description: "JRuby's Enumerator implements java.util.Iterator so it could be turned into a Java stream."
keywords: "jruby, java-integration, Enumerator, to_enum, java.util.Iterator, spliterator, java.util.stream, stream"
image:
  path: /public/jewel_128.png
  twitter: /public/jewel_128.png
  height: 128
  width: 128
---

Java 8 brought some interesting functional aspects to the Java standard library -
a fairly decent support for transforming streams of data. Ruby, obviously, has its
own perks to do quite more (as binding and closures are first class) but there's
no reason to be left out on JRuby.

And so, without much fanfare, `Enumerator` implements [`java.util.Iterator`][1] and
adds a `stream` helper. We've touched on Java streams [previously][2], the same is
feasible using a Ruby array (block-less `each` returns an enumerator) :

{% highlight ruby %}
numbers = [ 1, 2, 3, 4, 5, 6, 7, 8 ]

supplier = -> { Array.new } # Supplier<R>
accumulator = -> (c, e) { c << e } # BiConsumer<R,? super T>
combiner = -> (c1, c2) { c1.addAll(c2) } # BiConsumer<R,R>

three_even_squares = numbers.each.to_java.stream.
  filter( ->(n) { puts "filtering #{n}"; n % 2 == 0 } ).
  map( ->(n) { puts "  mapping #{n}"; n * n } ).
  limit(3).collect(supplier, accumulator, combiner).
  to_s # an ArrayList of "[ 4, 16, 36 ]"
{% endhighlight %}

... or an example with an explicit `Enumerator` calculating fibonacci till the end :

{% highlight ruby %}
enum = Enumerator.new do |out|
  out << prev = 1; sum = 1
  loop { prev, sum = sum, prev + sum; out << sum }
end

enum.to_java.stream.filter( ->(n) { n > 100 } ).
     limit(10).forEach( ->(n) { puts n } )

# prints 10 numbers from the sequence skipping <= 100
{% endhighlight %}

<!--
{% highlight ruby %}
# calculate something expensive `n!`
def calculate(n); (1..n).inject(1) { |i, p| p * i } || 1 end

# a thread-safe loop Enumerator starting at 1
def safe_loop
  i = java.util.concurrent.atomic.AtomicInteger.new
  Enumerator.new { |out| loop { out << i.increment_and_get } }
end

t = Time.now

collector = java.util.stream.Collectors.toCollection do
  java.util.concurrent.ConcurrentLinkedQueue.new
end
result = safe_loop.to_java.stream(true).
  map( ->(n) { calculate(n) } ).
  limit(3000).collect(collector)

puts "loop took: #{Time.now - t} elements: #{result.size}"
{% endhighlight %} -->

<div class="message">
  JRuby >= <b>9.1.3</b> is needed for a Java
  <a href="http://www.tutorialspoint.com/java8/java8_streams.htm" target="_blank">stream</a>-able
  Enumerator experience.
</div>

On a final note, using [parallel][3] streaming does not make much sense.
Although (non-blocking) thread-safe `Enumerator` instances are achievable, parallelizing
iteration over such won't provide the desired benefit until JRuby's underlying
(thread-based) implementation details are synchronizing over `next`-ed values.

[0]: http://kares.org/jruby-ji-doc/_index.html
[1]: https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html
[2]: /better-closures-for-functional-java#stream-example
[3]: http://docs.oracle.com/javase/8/docs/api/java/util/stream/StreamSupport.html#stream-java.util.Spliterator-boolean-
