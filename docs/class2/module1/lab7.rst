Lab 7 - TLS Version Control 
---------------------------

If you care about security at all, then you also need to care about SSL
and TLS. These are the protocols that provide encryption to Internet
traffic. It’s the "S" in HTTPS, and FTPS, and IMAPS, and you get the
picture. Without encryption, all communication between a client and
server is visible for any third party to see, and that can have some
pretty devastating effects. Encryption is a cornerstone of network
security, but if you’ve been paying attention to the news the last few
years, you’re probably also aware that SSL and TLS are not without their
own flaws. Exploits like BEAST, Heartbleed, and POODLE take advantage of
specific holes in the SSL and TLS protocols. To stay ahead of these
vulnerabilities, industry best practices suggest that we avoid some
versions of these protocols, specifically SSLv2, SSLv3, and now TLSv1.
This also applies to certain ciphers, like MD5, RC4, and eventually even
the RSA key exchange. The F5 BIG-IP platforms makes it extremely easy to
control and enforce these protocols and ciphers, but at the same time
you may not simply want to "break" some users. If your income depends on
Internet commerce, for example, the last thing you may want to do is
block customers from buying from you because they have a slightly
out-of-date browser. Eventually most of those older browsers will go
away, or at least enough of them that you can justify disabling lower
encryption protocols and ciphers. But then how do you know? F5 iRules
have access to everything in OSI layers 3 to 7, which includes some
pretty rich information about SSL and TLS. Let’s take a look at an iRule
that will catalog the SSL/TLS versions that clients are using.

.. NOTE:: A word on "mixed content" – All too often developers or development
   platforms provide access to both HTTP and HTTPS content within the same
   application. This is referred to as "mixed content" and can have some
   significant security implications. For example, an application may
   redirect a user to an HTTPS URL to authenticate, but then switch back to
   HTTP after authentication to access normal application content. At the
   very least, the HTTPS authentication is very likely going to set up a
   "session" with the user, which will also very likely be maintained by an
   HTTP cookie in the browser. If the application session is now
   communicating via HTTP, all of that user session information is in the
   clear. Long story short, don’t allow mixed content in your applications.
   If there is *anything* in your application worth protecting with
   encryption, it is best practice to encrypt *everything*.

Objectives:

-  Deploy and test the example TLS Version iRule code

Lab Requirements:

-  BIG-IP LTM, web server, client browser, and SSL server certificate.
   If you don’t have certificates to test with, you can use the CA
   certificate and server certificate and private key provided in the
   Client Certificate Inspection lab.

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when CLIENTSSL_HANDSHAKE {
       ISTATS::incr "ltm.virtual [virtual name] c [SSL::cipher version]" 1
   }

Apply this iRule to an SSL virtual server (VIP).

Analysis
~~~~~~~~

-  iStats are user-created custom statistics, accessible from both the
   data plane (iRules) and the control plane (tmsh, on-box scripts,
   etc.). What we’re doing here is simply cataloging the SSL/TLS version
   from each client SSL handshake and storing these in an iStats table.
   That data can then be accessed from pretty much anywhere. For
   example, see David Holmes’ excellent guide on dumping this
   information to a Google pie chart:
   
   https://devcentral.f5.com/codeshare/categorize-ssl-traffic-by-version-display-as-graph/tag/istats

Testing
~~~~~~~

#. Apply this iRule to an SSL VIP and test across several browsers and
   platforms. If at all possible, stage this iRule someplace that it can
   be accessed by a larger audience.
   
#. At any point you can access the data collected in the iStats by
   simply typing the following at the BIG-IP command line:

   ``istats dump``

.. HINT:: For the purposes of this lab we can use
   ``openssl s_client -connect www.f5test.local:443 <cipher>``
   where cipher options could include {-ssl3, -tls1, -tls1_1, -tls1_2}
   to simulate different connections.

Here’s an example output:

   .. code-block:: console

      [ ltm.virtual=/Common/stdsslvip ][TLSv1.1] = 54 (2015-09-01 13:52:20)
      [ ltm.virtual=/Common/stdsslvip ][SSLv3] = 21 (2015-09-01 13:52:20)
      [ ltm.virtual=/Common/stdsslvip ][TLSv1.2] = 282 (2015-09-01 13:52:20)
      [ ltm.virtual=/Common/stdsslvip ][TLSv1] = 32 (2015-09-01 13:52:20)

Bonus version
~~~~~~~~~~~~~

Now that you have a better idea of what protocols are being used on
your site, you may now want to ease your customers into a better
security posture, versus simply denying access with no warning. To
do this we’ll first test the SSL/TLS version, and if it’s below our
security threshold, we’ll redirect the user to another page or site
to inform them that they need to upgrade their browser.

.. code-block:: tcl
   :linenos:

   when CLIENTSSL_HANDSHAKE {
       if { ( [SSL::cipher version] equals "TLSV1" ) or ( [SSL::cipher version] equals "SSLv3" ) } {
           set redirect "http://www.f5test.local/insecure.html"
           SSL::respond "HTTP/1.1 302 Found\r\nLocation: $redirect\r\n\r\n"
       }
   }

You’re still allowing SSLv3 and TLSv1 at this point, which is
definitely bad, but you’re not allowing access to the application
for anything less than TLSv1.1.

.. HINT:: 
   #. Change client ssl cipher from ``DEFAULT`` to ``DEFAULT:SSLv3``
   #. Use ``openssl s_client -connect www.f5test.local:443 -ssl3`` to connect  