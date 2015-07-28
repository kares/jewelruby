---
title: The Dude JRuby-OpenSSL Release
tags: "jruby-openssl, openssl, release"
---

<div class="message">
  JRuby-OpenSSL <b>0.9.8</b> is dedicated to
  <a href="https://youtu.be/GZR58d77a4A" title="The Big Lebowski">"The Dude"</a>
  as '98 was the year!
</div>

[JRuby-OpenSSL][0] has taken up another step in C OpenSSL compatibility for JRuby.

* no more `uninitialized constant OpenSSL::SSL::Session` <br>
  (although session support is limited by what Java APIs allow us)
* `PKCS5.pbkdf2_hmac_sha1` work with an empty salt/key + is a little faster
* `OpenSSL::HMAC.hexdigest` accepts an empty key (Java APIs don't)
* .rb parts have been updated to align with Ruby 2.2 (for [JRuby 9000][1])
* improved "incomplete" X.509 certificates parsing (the MRI-way)

Regarding certificate parsing JRuby-OpenSSL used to rely on what's there with the
security provider. Since, we're trying to do more work using [Bouncy-Castle][4]
directly and only touch the provider for things the library can no handle.

While generating X.509 certificate objects (e.g. when using libraries such as
[ActiveMerchant][5] that set `Net::HTTP#ca_path=` for **https://** endpoints),
both the Sun provider (present in OracleJDK) and BC exhibit some contention. <br>
Meaning, that while certificate parsing seems fast with a simple test, under
concurrent threads doing the same operation might cause a slow-down.
An experimental feature to cache certificates on lookup has been included - try it
with `-J-Djruby.openssl.x509.lookup.cache=true` and report if there's any
discrepancy [improving the numbers][6].

Also, there's always a little bit of security in the end. For TLS we've disabled
the **DHE** algorithm on Java 7 while on Java 8 we're forcing it to have a key size
of 2048. Not a long-term solution but needed due compatibility, we recommend
that you **try disabling DHE** `-J-Djdk.tls.disabledAlgorithms="SSLv3, DHE"`
and switch to **ECDHE** (elliptic curves) instead.

<!-- Expect us in JRuby 1.7.22 and 9.0.1.0 ... -->

[0]: https://github.com/jruby/jruby-openssl
[1]: http://blog.jruby.org/2015/07/jruby_9000/
[4]: http://bouncycastle.org/java.html
[5]: https://github.com/activemerchant/active_merchant/blob/v1.52.0/lib/active_merchant/connection.rb#L121-L123
[6]: https://github.com/jruby/jruby-openssl/issues/43
