---
title: Validating ActiveRecord Connections
tags: "jruby, activerecord, activerecord-jdbc-adapter, rails"
---

[ActiveRecord][1] has been around as long as Ruby on Rails and since the framework
was pretty much build all from scratch database connection pooling has been invented
along the way (as it got [critical][2] [driven by <span class="redish">JRuby</span>]).

<!--
Pooling is necessary as opening new connections is not cheapest but also since it
allows control of the database resource utilization. -->

What pooling does is that instead of each thread opening its own connection a pool
decides whether it will open a new connection or can re-use an existing one.
Which sometimes leads to pooled connections sitting around idle (when application
load decreases after a high load).

<!--
Any kind of pooling in general is usually not part of the API, ActiveRecord does
it the other way and there's (almost) no way you get around ActiveRecord::ConnectionPool.
-->

A fairly known issue when keeping (idle) connections open, for longer periods, is that they
eventually become stale.
Thus pools tend to [verify][3] connections being still [active][4] on each checkout.
Yet, talking over <span title="in terms of being controlled by a client or provider">non-reliable</span>
<span title="cloudy">:cloud:</span> TCP networks, this is often not enough as
sockets might end up in a "black-hole" state where pings seems to be sent
fine but never return properly and only error after a
<span title="usually 900 seconds/15 minutes">socket timeout is reached<span>.
The application thread/request seemingly hangs until the timeout kicks in.

<!--
Especially true with all kind of firewalls and cloudy deployments where one is
not in full control of the networking stack.-->

One solution would be to set a lower timeout while creating a network socket ([decent][5]
[drivers][6] have a socket-timeout configuration option) however that's more of a work-around.
Instead validating "ping" queries with a guard timer, expected to return in a given
time frame (say 3 seconds), should be used. The connection gets killed if a response
is not received and a reconnect is attempted.

This pattern works well for most databases and is built into
<span title="JDBC">Java's database connectivity<span> APIs.
Thus under JRuby, using [activerecord-jdbc-adapter][9], all is easy.
Consult the JDBC driver if validation [timeouts][7] are supported and than configure
`:connection_sql_timeout: 3` (in *database.yml*).

<div class="message">
  <code>:connection_sql_timeout</code> only has effect under AR-JDBC <b>1.3.19</b> or later.
</div>

Note that setting a timeout isn't necessary when a JDBC pool is used (`jndi: ` is set),
in such cases instead make the underlying pool perform periodic pings for idle connections.
Alternatively, under MRI, go and invent a validating thread [...][8]

<!--
+  properties:
+    autoReconnect: true
+    maxReconnects: 3 # no of reconnects to attempt
+    initialTimeout: 1 # sec to wait between reconnects
+    failOverReadOnly: false # not read-only when failing over
+    connectTimeout: 5000 # ms when establishing a connection
+    socketTimeout: 30000 # ms for socket operations
-->

<!-- {% gist 8387126 %} -->

[1]: http://guides.rubyonrails.org/active_record_basics.html
[2]: http://guides.rubyonrails.org/2_2_release_notes.html
<!-- [2]: https://github.com/kares/activerecord-bogacs#setup -->
[3]: https://github.com/rails/rails/blob/v4.2.5/activerecord/lib/active_record/connection_adapters/abstract/connection_pool.rb#L456
[4]: https://github.com/rails/rails/blob/v4.2.5/activerecord/lib/active_record/connection_adapters/abstract_adapter.rb#L328
[5]: https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html
[6]: https://jdbc.postgresql.org/documentation/92/connect.html
[7]: http://docs.oracle.com/javase/7/docs/api/java/sql/Connection.html#isValid(int)
[8]: https://github.com/kares/activerecord-bogacs/commit/4924e0c32c4361b15153973e6a72495630aa0e06
[9]: https://github.com/jruby/activerecord-jdbc-adapter
