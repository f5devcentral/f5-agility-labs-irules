Additional Lab 3 - HSTS / HPKP
------------------------------

HSTS
~~~~
As per OWASP and RFC 6797, HTTP Strict Transport Security (HSTS) is an
opt-in security enhancement that is specified by a web application
through the use of a special response header. Once a supported browser
receives this header that browser will prevent any communications from
being sent over HTTP to the specified domain and will instead send all
communications over HTTPS. It also prevents HTTPS click-through prompts
on browsers. In layman’s terms, HSTS is an HTTP header sent from the
server to the client browser. The information within that header
indicates to the browser that any communication with that domain must be
over HTTPS, regardless of any HTTP URL’s encoded into the HTML pages, or
what the user types into the browser address bar. This information is
also placed into the browser’s long term storage, so that subsequent
requests will immediately communicate over HTTPS. The header looks like
this:

``Strict-Transport Security: max-age=31536000; includeSubDomains``

The max-age attribute defines how long the browser should store this
information. In this case, it is 1 year. The includeSubDomains attribute is
optional and indicates that the browser must communicate over HTTPS to
this and any sub-domain of the current domain. To be most effective,
this response header should a) only be transmitted over HTTPS in the
first place to prevent stripping, and b) be transmitted at the lowest
possible point in the domain to include all sub-domains. For example, to
cover all sub-domains of domain.com, you might initially redirect a
client to https://domain.com and then send an immediate redirect back to
the requested HTTPS URL and include the HSTS header.

.. NOTE:: 

   Prior to BIG-IP LTM version 13.0, HSTS was implemented with an iRule (see below). As of v13, HSTS is simply enabled  
   within an HTTP profile.

   1. Create an HTTP profile.
   2. Under the new "HTTP Strict Transport Security" section (bottom), set Mode to enabled (checked), set a maximum age in seconds and 
      check the "Include Subdomains" option if you want the HSTS header to be sent for subdomains of this URL. 
   
   3. The Preload option is used by browser vendors to hard code this information into future browser updates. You must separately 
      submit the URL to the vendors' preload lists. They will check that the preload option is set before hard coding your URL.
   
   A word of warning: once browser vendors hard code this URL into new versions, it is practically impossible to remove it, so make sure this is exactly what you want and that no "mixed" content (HTTP and HTTPS) exists for this URL.


HPKP
~~~~
As per OWASP and RFC 7469, HTTP Public Key Pinning (HPKP) is a new HTTP
header that allows SSL servers to declare hashes of their certificates
with a time scope in which these certificates should not be changed. In
other words, it is a method by which a server can indicate to a browser
the certificate that the client browser should see. This technology
helps to prevent active man-in-the-middle attacks whereby an agent is
able to silently decrypt and re-encrypt traffic between a client and
server. Without knowledge of the server’s private key, the agent won’t
possibly be able to spoof the server’s real certificate. Therefore the
"forged" certificate coming from the agent will not match the
certificate embedded in the HPKP header. That header looks like this:

``Public-Key-Pins: max-age=2592000;``

``pin-sha256="E9CZ9INDbd+2eRQozYqqbQ2yXLVKB9+xcprMF+44U1g=";``

``pin-sha256="LPJNul+wow4m6DsqxbninhsWHlwfp0JecwQzYpOLmCQ=";``

``report-uri="http://example.com/pkp-report"``

The header name is ``Public-Key-Pins`` and has a similar max-age attribute
as HSTS. The header can include multiple encoded certificate hashes
indicated as the ``pin-sha256`` values. If the incoming certificate does
not match anyone of these values, a "report-uri" attribute can send the
browser to a reporting facility to report this error (possible attack).
To be most effective, this response header should only be transmitted
over HTTPS to prevent stripping. To create the encoded value for a given
certificate, use the following OpenSSL commands:

``openssl x509 –in <cert> -pubkey –noout | openssl rsa –pubin –outform der | openssl dgst –sha256 –binary | openssl enc –base64``


Scenario
~~~~~~~~~
In this lab exercise, we demonstrate how to deploy both the HSTS and HPKP using an iRule.

Requirements
~~~~~~~~~~~~

-  BIG-IP LTM, web server, client browser, and a SSL server certificate.
   If you don’t have certificates to test with, you can use the CA
   certificate, server certificate, and the private key provided in the
   the Client Certificate Inspection lab from Additional labs section.

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when RULE_INIT {
       set static::fqdn_pin1 "X3pGTSOuJeEVw989IJ/cEtXUEmy52zs1TZQrU06KUKg="

       # Set max_age to 180 days
       set static::max_age 15552000
   }
   when HTTP_RESPONSE {
       # Insert an HSTS header
       HTTP::header insert Strict-Transport-Security "max-age=$static::max_age; includeSubDomains"
       # Insert an HPKP header
       HTTP::header insert Public-Key-Pins "pin-sha256=\"$static::fqdn_pin1\" max-age=$static::max_age; includeSubDomains"
   }


Analysis
~~~~~~~~

-  The above iRule example will perform two functions. Upon receipt of
   the HSTS header, any attempt to communicate with that VIP again will
   only use HTTPS. With the injection of the HPKP header, if the
   certificate (CA or server certificate) does not match the one
   provided in the SSL handshake, the browser will either end the
   connection and potentially follow a report-uri URL if it exists.

Testing
~~~~~~~

- Apply this iRule to an HTTPS virtual server (VIP).
- Repeatedly navigate to the HTTP URL http://www.f5demolabs.com to 
   verify that you are indeed talking to the HTTP VIP.

- Navigate to the HTTPS URL https://www.f5demolabs.com one time to
   verify that you can access it.

- Now attempt to go to the HTTP URL http://www.f5demolabs.com again.
   Depending on the browser it should immediately go to the HTTPS URL.

- If you’re using a Chrome browser, you can navigate to
   ``chrome://net-internals/#hsts`` to see this URL value now added to
   Chrome's HSTS list.  Under Query Domain, enter ``www.f5demolabs.com`` to 
   Domain: entry box and click Query.  ``Be sure to delete domain before
   moving on or else you will have an issue with a later lab.``

- Unfortunately, unless you’re using a server certificate that chains
   up to a public root, you won’t be able to test HPKP here. Per the
   Mozilla Developer Network, "Firefox (and Chrome) disable Pin
   Validation for Pinned Hosts whose validated certificate chain
   terminates at a user-defined trust anchor (rather than a built-in
   trust anchor). This means that for users who imported custom root
   certificates all pinning violations are ignored."
   
.. HINT:: You can still use Chrome Developer Tools to see the HPKP header.    

