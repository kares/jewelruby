---
title: ActiveRecord Pooling Alternatives
tags: "jruby, activerecord, activerecord-bogacs, activerecord-jdbc-adapter, rails"
description: "Connection pooling matters in Rails and a solution beyond ActiveRecord's built-in pool"
keywords: "jruby, activerecord, connection pooling, activerecord-bogacs, activerecord-jdbc-adapter, rails, jdbc"
---

Any kind of resource pooling, in general, should not be reflected too much in a
public interface of a library (as most details are often useful to be changed).
With Rails `ActiveRecord::ConnectionAdapters::ConnectionPool` leaks to user-land
and is even part of the [documentation][2]. It's essential since the API
provided isn't available elsewhere e.g. check if there's currently an
[`active_connection?`][4] available or using [`with_connection`][3].

Pooling internals changed with major releases and each version (Rails 2.3, 3.x, 4.x)
had its own characteristics - some problematic on non-GIL rubies esp. under high
connection utilization.

So, the built-in pool is not easy to avoid (although some have [tried][4]), but
knowing Ruby what if we used a different implementation quacking the same interface
as `ConnectionPool` has. Actually, that [**works**][6] and has been a feasible
solution, compared to monkey-patching the whole connection pool, with a simple hook :

{% highlight ruby %}
module MyRails # config/application.rb
  class Application < Rails::Application
    # ...
  end
end
require 'active_record/bogacs'
if Rails.env.production? # replace AR's pool impl
  pool_impl = ActiveRecord::Bogacs::DefaultPool
  ch = ActiveRecord::ConnectionAdapters::ConnectionHandler
  ch.connection_pool_class = pool_impl
end
{% endhighlight %}

<!--
with
contention/concurrency issues faced under heavy concurrency (with JRuby).-->

Using [Bogacs'][6] "default" pool implementation, we effectively back-port the pool
from ActiveRecord 4.x, adapted for compatibility with previous versions, for applications
running old Rails - say 3.2.x (by the way, resolves most contention issues
faced when serving concurrent requests under JRuby).

Another pool implementation provided is `ActiveRecord::Bogacs::FalsePool` which
does the expected API, keeps connections in a thread-local but does not do any
actual pooling on its own. There's plenty of use-cases where you do not want
pooling or would like to employ an underlying pool outside ActiveRecord's scope.

Under JRuby, using [activerecord-jdbc-adapter][7], the above setup hook and having
`jndi: jdbc/MyRailsDS` is all there is to it. Just keep in mind that `pool: 50` or
`checkout_timeout: 5` won't have any effect - connection pooling details are only
to be accounted when configuring the underlying data source.

p.s. [**ActiveRecord-Bogacs**][6] was [presented][8] and released at
<a target="_blank" href="http://kares.org/jrubyconf-2014/#/section-7/page-1" title="JRuby (on Rails) Knacks - JRubyConf 2014">JRubyConf</a>!

[1]: http://guides.rubyonrails.org/v4.2/active_record_basics.html
[2]: http://api.rubyonrails.org/v4.2/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html
[3]: http://api.rubyonrails.org/v4.2/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html#method-i-with_connection
[4]: http://api.rubyonrails.org/v4.2/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html#method-i-active_connection-3F
[5]: https://gist.github.com/nicksieger/300782
[6]: https://github.com/kares/activerecord-bogacs
[7]: https://github.com/jruby/activerecord-jdbc-adapter
[8]: http://kares.org/jrubyconf-2014/
