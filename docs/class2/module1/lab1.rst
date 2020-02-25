Lab 1 - TLS Version Control 
---------------------------

Scenario
~~~~~~~~

If you care about security at all, then you also need to care about SSL
and TLS. These are the protocols that provide encryption to the Internet
traffic. It’s the "S" in HTTPS, FTPS, and IMAPS. You get the picture!

Without encryption, all communication between a client and server is 
visible for any third party to see and this can have some pretty 
devastating effects. Encryption is a cornerstone of network security. 
If you’ve been paying attention to the news the last few years, you’re 
probably also aware that SSL and TLS are not without their own flaws. 
Exploits like BEAST, Heartbleed, and POODLE take advantage of
specific holes in the SSL and TLS protocols. To stay ahead of these
vulnerabilities, industry best practices suggest that we avoid some
versions of these protocols specifically SSLv2, SSLv3, and now TLSv1.
This also applies to certain ciphers such as MD5, RC4, and eventually 
the RSA key exchange. 

The F5 BIG-IP platforms make it extremely easy to control and enforce 
these protocols and ciphers but at the same time, you may not simply 
want to "break" some users. If your income depends on Internet commerce, 
the last thing you may want to do is block customers from buying from you 
because they have a slightly out-of-date browser. Eventually, most of these
older browsers will go away or at least enough of them that you can justify 
disabling lower encryption protocols and ciphers. But, how do you know? 

F5 iRules have access to everything in the OSI layers from 3 to 7 that includes 
some pretty rich information about SSL and TLS. Let’s take a look at an iRule 
that will catalog the SSL/TLS versions that clients are using.

.. NOTE:: A word on "mixed content" – All too often, developers or development
   platforms provide access to both HTTP and HTTPS content within the same
   application. This is referred to as "mixed content" and can have some
   significant security implications. For example, an application may
   redirect a user to an HTTPS URL to authenticate, but then switch back to
   HTTP after authentication to access normal application content. At the
   very least, the HTTPS authentication is very likely going to set up a
   "session" with the user, which will also very likely to be maintained by an
   HTTP cookie in the browser. If the application session is now
   communicating via HTTP, all of that user session information is in the
   clear. Long story short, don’t allow mixed content in your applications.
   If there is *anything* in your application worth protecting with
   encryption, it is a best practice to encrypt *everything*.

Requirements
~~~~~~~~~~~~

-  BIG-IP LTM, web server, client browser, and a SSL server certificate.
   If you don’t have certificates to test with, you can use the CA
   certificate, server certificate, and private key provided in the
   Client Certificate Inspection lab from Additional labs section.

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when CLIENTSSL_HANDSHAKE {
       ISTATS::incr "ltm.virtual [virtual name] c [SSL::cipher version]" 1
   }

Analysis
~~~~~~~~

-  iStats are user-created custom statistics, accessible from both the
   data plane (iRules) and the control plane (tmsh, on-box scripts,
   etc.). What we’re doing here is simply cataloging the SSL/TLS version
   from each client SSL handshake and storing these in an iStats table.
   That data can then be accessed from pretty much anywhere. For
   example, see David Holmes’ excellent guide on dumping this
   information to a Google pie chart:
   
   https://devcentral.f5.com/s/articles/categorize-ssl-traffic-by-version-display-as-graph

Testing
~~~~~~~

- Apply this iRule to an SSL VIP and test across several browsers and platforms. 
  If at all possible, stage this iRule at someplace that it can be accessed by a 
  larger audience.
   
- At any point you can access the data collected in the iStats by
  simply typing the following at the BIG-IP command line:

   ``istats dump``

.. HINT:: For the purpose of this lab we can use

   ``openssl s_client -connect www.f5demolabs.com:443 <cipher>``

   where cipher options could include {-tls1, -tls1_1, -tls1_2}
   to simulate different connections.

Here’s an example output:

   .. code-block:: console

      [ ltm.virtual=/Common/stdsslvip ][TLSv1.1] = 54 (2015-09-01 13:52:20)
      [ ltm.virtual=/Common/stdsslvip ][TLSv1.2] = 282 (2015-09-01 13:52:20)
      [ ltm.virtual=/Common/stdsslvip ][TLSv1] = 32 (2015-09-01 13:52:20)

Bonus version
~~~~~~~~~~~~~

Now that you have a better idea of what protocols are being used on
your site, you may now want to ease your customers into a better
security posture than simply denying access with no warning. To
do this, we’ll first test the SSL/TLS version. if it’s below our
security threshold, we’ll redirect the user to another page or site
to inform them that they need to upgrade their browser.

.. code-block:: tcl
   :linenos:

   when HTTP_REQUEST {
       if { (( [SSL::cipher version] equals "TLSv1" ) or ( [SSL::cipher version] equals "SSLv3" )) and not ( [HTTP::uri] equals "/insecure.html" ) } {
           set redirect "https://www.f5demolabs.com/insecure.html"
           HTTP::respond 302 Location "${redirect}"
      }
   }

You’re still allowing SSLv3 and TLSv1 at this point, which is
definitely bad, but you’re not allowing access to the application
for anything less than TLSv1.1.

.. HINT:: 
   #. Change client ssl cipher from ``DEFAULT`` to ``DEFAULT:SSLv3``
   #. Use ``openssl s_client -connect www.f5demolabs.com:443 -tls1`` to connect
   #. Move bonus version of irule, sec_irules_tls_version_control_2, to the selected list of iRules on the generic-app HTTPS virtual server
   
.. NOTE:: 
   Lab Notes:
   
   - Use the Chrome browser to manage the BIG-IP.
   - Use the Firefox browser to perform access testing.
      - Modify Firefox's TLS version by navigating to about:config and modifying the "security.tls.version.max" value.
      - 1 = TLSv1.0
      - 2 = TLSv1.1
      - 3 = TLSv1.2 
   - The test site URL is https://www.f5demolabs.com. A hosts file entry is already applied to the lab desktop.
   - Use a command line client to also test access:
      - curl -vk https://www.f5demolabs.com --[tlsv1.0|tlsv1.1|tlsv1.2]
      - openssl s_client -connect www.f5demolabs.com:443 -[tls1|tls1_1|tls1_2]
   - Three TLS version control iRules are provided:
      - Basic istats capture
      - Redirect to insecure page if TLSv1 or SSLv3
      - Provide David Holmes' iRules and access to the /sslversions URL.
