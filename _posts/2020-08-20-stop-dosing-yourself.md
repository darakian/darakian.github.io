---
layout: post
title:  "Stop DOSing yourself"
date:   2020-08-19 15:30:00 -0700
---
# Deep in the woods
Some time ago a colleague of mine was working on a problem with a license server we run and had asked me for help. He was trying to determine why some license checkouts to the server would be connect and checkout without incident and others would timeout. Some background; we dealt with a good amount of third party, close source code in order to run our customer workflows and licensed software was the norm for most of those workflows. To smooth everything out we stood up and maintained our own internal set of license servers which were basically invisible to our customers and while this made for a nice product to sell it also left us with the support burden of maintaining these code blobs.

Now my colleague had a habit not seeing the forest for getting lost in the trees and in this instance he had pinged me with a question about tcp header options. He had constructed a test scenario where he would emulate a busy server by using `nc` to connect to it while also using a third party tool to check out a license. In dumping packets he noticed a differences in `TS val` output between two captured tcp streams and had become convinced that the answer to the different behavior in the license server must be down to a difference in the traffic. It was not and if you're curious the `TS val` is short for `timestamp value`.

# Side effects in distributed systems
As it turns out the tcp was not to blame. I asked for a copy of the packet captures that my colleague was working with, dropped one into wireshark and immediately saw a tls connection
![wireshark](https://raw.githubusercontent.com/darakian/darakian.github.io/master/_images/2020-08-20-stop-dosing-yourself/tcp.png)
Seeing this it was clear that there was no sense in chasing tcp packets, we needed to reason about the issue at the application layer. I asked him to replace his `nc` call with `openssl s_client -connect ip:port` which had an almost miraculous effect; licenses were now being checked out without issue.

There were differences though, `nc` would open and block in the terminal while `openssl` would not. The `openssl` terminated with a very helpful of `verify error:num=19:self signed certificate in certificate chain`. Fantastic confirmation that the license server is speaking tls and standards compliant tls to boot! Running the `openssl` connection again while disabling certificate checking reviled one more element; the server was expecting a valid client certificate. To recap; we observed that opening a plain tcp connection to the server causes license checkouts to fail, opening a tls connection fails and license checkouts continue as they should, and the license server is expecting clients to provide certificates.

On a hunch I ran the server binary through the `strings` tool and discovered that the openssl had been linked in. At this point I surmised that openssl was responsible for all connection handling and that there was some sort of blocking in there. After digging through the openssl docs I found the [`SSL_accept`](https://www.openssl.org/docs/man1.0.2/man3/SSL_accept.html) function with the following note.
```
NOTES
The behaviour of SSL_accept() depends on the underlying BIO.
If the underlying BIO is blocking, SSL_accept() will only return once the handshake has been finished or an error occurred.
```

In the case of our `nc` call the handshake was never failing because it was never starting. A shared network socket was blocking on a connection which would never succeed.

# The actual problem
So, why was `nc` even used in testing at all? Well, given the importance of these license servers someone had the idea that we should have a health check for them and that health check looked like this
```
try:
  s = socket.create_connection(address, timeout=2)
  s.close()
  return true
except socket.error:
  return false
```
Initiate a connection and then immediately close it. We were investigating a denial of service directly caused by our own health check. The fix for this issue ended up giving code like
```
try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.settimeout(2)
        sock.connect((address[0], address[1]))
        try:
            context = ssl.create_default_context()
            context.check_hostname = False
            context.verify_mode = ssl.CERT_NONE
            with context.wrap_socket(sock):
                return True
        except (ssl.SSLError, OSError):
            return True
except OSError:
    return False
```
Admittedly this is pretty convoluted logic. A tls connection is attempted with no certificate checking and regardless of success or failure we return true. Only in the even that the underlying socket fails do we return false. In particular why do we return true if a TLS connection succeeds? Wouldn't that be a violation of the client certificate model discussed above? Yes it would, however this code is also used on license servers which don't have the client cert restriction.

# Conclusion
It seems that whoever wrote the original health check code didn't consider that their code may have had side effects on the servers that it was checking. I think the real problem though is that we had no metrics watching checkouts and we couldn't see the effect of the health check. Once this health check was added it became trusted and lead support engineers down false paths. The impact of this bug resulted in over a year of degraded service. So, validate your assumptions. Stop DOSing yourself.

```
Thanks for reading
```
