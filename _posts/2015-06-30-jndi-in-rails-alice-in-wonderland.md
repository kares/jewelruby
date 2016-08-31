---
title: JNDI in Rails - Alice in Wonderland
tags: "activerecord, activerecord-jdbc-adapter, jndi, rails"
description: "Dealing with no connection available using JNDI in Rails when an initializer talks to the database"
keywords: "jruby, activerecord, activerecord-jdbc-adapter, jndi, rails, jdbc, configure connection, initializer"
---

<div class="message">
  AR-JDBC fix mentioned in this post is available since the
  <a href="https://github.com/jruby/activerecord-jdbc-adapter/releases/tag/v1.3.17"><b>1.3.17</b></a> release.
</div>

[AR-JDBC][0] has provided support for using JNDI with ActiveRecord since the Rails "zenith" (2.x) days.
Internally JNDI relies on pool `checkin`/`checkout` [callbacks][1] being [triggered][8]
on adapter instances to minimize connection occupancy.

Of course, as ActiveRecord's `ConnectionPool` is baked in and thus hard ([although not impossible][2]) to avoid.
One needs to account for possibly double configuring pooling properties such as the size.
But that's [another story]({% post_url 2016-03-14-activerecord-pooling-alternatives %}).

<!--
Works fairly well, maybe except for the need to double configure pooling properties such as the connection limit.
Since ActiveRecord's `ConnectionPool` is baked in it's hard ([but not impossible][2]) to avoid.
Rails might also be considered greedy on connection usage with its default middleware stack. But that's another story.
-->

Recently, due changes in ActiveRecord **4.2**, a [bug][3] report came in that
revealed an interesting issue with a `jndi:` configuration. Turns out for adapters
that execute queries during `initialize` (e.g. PostgreSQL and MySQL) the JDBC
connection ends up in an inconsistent state when an **initializer** performs any
database access.

The [fix][4] is fairly simple once the moving parts are understood. To make sure
the callbacks are consistently triggered, while inside the `initialize` method,
they needed to be moved outside.

Still, not quite the happy end. With an initializer talking to the database
a single JNDI connection is left open after boot (there might be more if threads are being started).
Alice will stay wondering forever or at least until an [abandoned connection timeout][5] is reached.

Meet `ActionDispatch::Reloader` [doing][9] `ActiveRecord::Base.clear_cache!`.
The theme around connection related caching in ActiveRecord seems that a connection
is mandatory. If the current thread is not connected `clear_cache!` will connect
(and there's no one left to clear-up).

Rails has been rolling this way for some time (at least all 4.x versions), and while
~master~ (>= 5.0) accepted a [patch][6] there's no word whether this small change
will get back-ported to 4.2.

> Sounds right to stress out standalone ActiveRecord uses are not affected, as long as they do not rely on `ActiveRecord::Railtie`.

<!--
Lastly, there's small JNDI (performance) goodies expected for AR-JDBC [1.4][7] e.g. treating the internal
connection handles lazy or optionally allowing to skip `configure_connection` execution.
 -->

[0]: https://github.com/jruby/activerecord-jdbc-adapter
[1]: http://api.rubyonrails.org/classes/ActiveSupport/Callbacks.html
[2]: https://github.com/kares/activerecord-bogacs
[3]: https://github.com/jruby/activerecord-jdbc-adapter/issues/649
[4]: https://github.com/jruby/activerecord-jdbc-adapter/commit/46794950527bdef4a3c703fd5973e71a335d28cf
[5]: https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Common_Attributes
[6]: https://github.com/rails/rails/pull/20516
[7]: https://github.com/jruby/activerecord-jdbc-adapter/issues/572
[8]: https://github.com/rails/rails/blob/v4.2.3/activerecord/lib/active_record/connection_adapters/abstract/connection_pool.rb#L360-L372
[9]: https://github.com/rails/rails/blob/v4.2.3/activerecord/lib/active_record/railtie.rb#L148-L153
