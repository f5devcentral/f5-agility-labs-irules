Lab 6 - Content Rewrite
-----------------------

It probably goes without saying that many application developers aren’t
aware of (or at least good at) secure coding practices. How many
applications in your environment have mixed HTTP and HTTPS content?
iRule content rewrite solutions have been around for a long time, so you
may have actually seen some of this code before. And yes, in the case of
mixed content HSTS does prevent this sort of oversight. But it’s also
important to understand that HSTS is a chainsaw when you may only need a
butter knife. Once you’ve enabled HSTS in the browser, if you actually
have legitimate HTTP content, you’ll most definitely break user access.
The tried and tested example below is a perfect way to force HTTPS upon
the browser, only as needed.

Objectives:

-  Deploy and test the example Streaming iRule code

Lab Requirements:

-  BIG-IP LTM, web server, client browser, and SSL server certificate.
   If you don’t have certificates to test with, you can use the CA
   certificate and server certificate and private key provided in the
   Client Certificate Inspection lab.

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:
   
   when HTTP_REQUEST {
       # Explicitly disable the stream profile for each request so it doesn’t stay
       # enabled for subsequent HTTP requests on the same TCP connection.
       STREAM::disable
       HTTP::header remove "Accept-Encoding"
   }
   
   when HTTP_RESPONSE {
       # Apply stream profile against text responses from the application
       if { [HTTP::header value Content-Type] contains "text"} {
          # Look for the http:// and replace it with https://
          STREAM::expression "@http://@https://@"
          # Enable the stream profile
          STREAM::enable
      }
   }

Apply this iRule to an HTTPS virtual server (VIP).

Analysis
~~~~~~~~

- Assuming application developers are going to continue to include HTTP object
  references in their HTML code, no matter how many times you’ve told them not
  to, the above is designed to find these references.  It will use the very
  powerful ``STREAM`` command to effortlessly sweep through the response payload 
  and replace any instance of http://

Testing
~~~~~~~

#. Access the HTTPS URL https://www.f5test.local/content.html
   without the iRule code.

#. Click the ``Test Link`` which is represented in the page with the href
   code below.

   ``<a href="http://www.f5test.local/f5.png">Test Link</a>``

   The link will open as an HTTP URL.

#. Add the above iRule to the HTTPS VIP. The ``Test Link`` href will be
   rewritten to an HTTPS URL.

Bonus version
~~~~~~~~~~~~~

Needless to say, ``STREAM`` is an incredibly powerful command, and a
very useful tool in your security arsenal. For example, what if you
also wanted to sanitize Social Security and credit card numbers

.. code-block:: tcl

   STREAM::expression "@\d3-\d2-\d4@***-**-****@ @\d4-\d4-\d4-\d4@xxxx-xxxx-xxxx-xxxx@"
   