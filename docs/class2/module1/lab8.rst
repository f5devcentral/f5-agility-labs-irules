Lab 8 - Securing Cookies
------------------------

HTTP cookie misuse represents one of the greatest vulnerability vectors
known to Internet communications. It’s number 2 on the Open Web
Application Security Project’s (OWASP) Top Ten list as “weak
authentication and session management”
(https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet), and also
touches other top ten list items, including XSS, security
misconfiguration, sensitive data exposure, and cross site request
forgery. So why do we use cookies if they’re so easy to get wrong? Well,
as it turns out, HTTP as a “stateless” protocol doesn’t provide its own
session management mechanism, so cookies are basically the best and
potentially only way to maintain information about a user session across
multiple HTTP requests and responses. Too often, however, applications
employ mixed content (HTTP and HTTPS in the same application), or worse,
store too much information in that cookie. If an HTTP cookie is being
used to maintain an authentication application session, it’s always best
practice to encrypt every part of that application. And it is never best
practice to store anything more than an identifier (some seemingly
random and unpredictable blob of characters) in a cookie. There are two
built-in mechanisms defined in RFC 6265 that can help. But first, let’s
first understand what an HTTP cookie is. An HTTP cookie is essentially
an HTTP header that is sent from the server to the client, and then sent
back to the server in each and every subsequent request. To send a
cookie, the server will format the header like this:

``Set-Cookie: foo=bar; path=/; domain=domain.com; expires=Sat, 02 May 2009 23:38:35 GMT``

where

- ``foo=bar`` represents the mandatory name=value content

- ``path=/`` represents the mandatory path, or scope of the cookie,
  such that the browser will only return this cookie if the request is
  in the designated path (in this case the entire website)

- ``domain=`` represents another, albeit optional, scoping mechanism
  that tells the browser that this cookie can be sent with any request
  in the same domain or subdomain. Unlike the path attribute which
  reduces visibility, the domain attribute increases visibility by
  making the cookie potentially available across multiple subdomains.
  Be careful with the domain attribute though. This is an often-used
  way to build single sign-on, where users authenticate at one domain,
  but are allowed access to other subdomains by virtue of the cookie
  being available to all. At the very least, if you don’t own every
  subdomain that this cookie could possibly be sent to, then someone
  else may get your user’s session cookies.

- ``expires=`` represents an optional attribute that indicates how
  long the client should retain this cookie. If the expires attribute
  is not in the Set-Cookie header, then the cookie is considered
  “session-based” and will generally live for as long as the browser
  session (ie. closing the browser deletes the cookie). If the expires
  attribute exists, the cookie is typically stored on disk or other
  long-term memory that will persist after the browser is closed and a
  new one is opened. This is, generally speaking, not a best practice
  from a security perspective. If a cookie is used to maintain some
  sort of client state information, and the computer itself is
  compromised, then that state can also be compromised. It is
  typically far better to not include an expires attribute, thus
  deleting the cookie when the browser closes. You can, however,
  delete an existing in-memory session-based cookie by sending a new
  cookie with the same attributes but with an expires attribute in the
  past.

Once the cookie has been received, and depending on scope, the client
will transmit that information back to the server in every request.

``Cookie: foo=bar``

Notice that a request cookie doesn’t have path, domain or expires
information. This information is meant for the client only, so doesn’t
need to be relayed back to the server.

Aside from adjusting path and domain attributes accordingly to limit or
expand visibility, there are two additional scoping attributes that play
a crucial part in cookie security.

- ``secure`` represents a single (no value) flag that tells the browser
  client to only transmit this cookie back in a secure (ie. encrypted
  HTTPS) request. This attribute is critical against attacks like cross
  site scripting (XSS) or request forgery (CSRF) where a browser may
  otherwise be tricked into sending session cookies to an unencrypted
  host. If you’re encrypting the entire application, the secure cookie
  flag is an excellent option.

- ``httpOnly`` represents a single (no value) flag that tells the
  browser client to only transmit this cookie back to non-scripted user
  agents. In other words, if a JavaScript agent makes a request inside the
  browser, the cookie will not be sent with this request. Many XSS and
  CSRF exploits rely on the ability to grab session cookies with rogue
  browser scripting (ex. JavaScript, vbscript, etc.). There are of course
  instances where a JavaScript agent needs to send the cookie, like in
  side-channel Ajax requests, but if not, this flag is highly useful.

So putting these attributes together might look something like this:

``Set-Cookie: foo=bar; path=/; secure; httponly``

I’ve removed the **expires** attribute because file-based cookies are
almost always a bad idea. And I removed the **domain** attribute because
there are better and more secure ways to do single sign-on. So in this
example, I’m setting a cookie called “foo” with a value of “bar”, that
is scoped to all paths within this host (path=/), and will only be
transmitted over HTTPS and only to non-script agents. As I mentioned a
few times, there’s simply no substitute for a good security product (ie.
web application firewall, malware scanner, etc.) and no excuse not to
write secure code, but if you find yourself in a situation where secure
cookie coding isn’t happening in the application, then here’s a quick
and easy way to enable it with F5 iRules.

Objectives:

-  Deploy and test the example cookie protection iRule code

Lab Requirements:

-  BIG-IP LTM, web server, client browser, and SSL server certificate.
   If you don’t have certificates to test with, you can use the CA
   certificate and server certificate and private key provided in the
   Client Certificate Inspection lab.

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when HTTP_RESPONSE {
       set ckname "mycookie"
       if { [HTTP::cookie exists $ckname] } {
           HTTP::cookie secure $ckname enable
           HTTP::cookie httponly $ckname enable
       }
   }

Apply this iRule to an HTTPS virtual server (VIP).

Analysis
~~~~~~~~

-  In this very simple iRule, we’re triggering an event on the HTTP
   response being sent from the application server, looking for the
   cookie ``mycookie``, and if it exists, enabling the ``secure`` and
   ``httpOnly`` flags. This command effectively includes the ``secure``
   and ``httpOnly`` flags in the ``Set-Cookie`` header being sent to the
   client.

Testing
~~~~~~~

#. Access HTTPS URL without iRule to see current cookie status.

   ``curl –vk https://www.f5test.local``	

#. Attach the iRule to the HTTPS VIP

#. Access the HTTPS URL to see the change in the cookie information.

   ``curl -vk https://www.f5test.local``

A word on cookie security – the ``secure`` and ``httpOnly`` flags are
exceedingly important for the proper and secure use of HTTP cookies, but
alone they are not perfect. There are still ways to compromise HTTP
cookies, even with these flags enabled, so do take additional
precautions which should definitely include a solid web application
firewall product and malware scanning and intrusion detection products.
