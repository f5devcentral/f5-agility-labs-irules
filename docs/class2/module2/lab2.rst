Additional Lab 2 - Content Rewrite
----------------------------------

Scenario:
~~~~~~~~~

It probably goes without saying that many application developers aren’t
aware of (or at least good at) secure coding practices. How many
applications in your environment have mixed HTTP and HTTPS content?
iRule content rewrite solutions have been around for a long time. So you
may have actually seen some of this code before.

And yes, in the case of mixed content HSTS does prevent this sort of 
oversight. But it’s also important to understand that HSTS is a chainsaw 
when you may only need a butter knife. Once you’ve enabled HSTS in the 
browser and if you actually have legitimate HTTP content, you’ll most 
definitely break user access. 

The tried and tested example below is a perfect way to force HTTPS upon
the browser, only as needed.

Requirements:
~~~~~~~~~~~~~
-  BIG-IP LTM, web server, client browser, and SSL server certificate.
   If you don’t have certificates to test with, you can use the CA
   certificate and server certificate and private key provided in the
   Client Certificate Inspection lab from Additional labs section.

-  A stream profile attached to the virtual server.

Baseline Testing:
~~~~~~~~~~~~~~~~~
- Access the HTTPS URL https://www.f5demolabs.com/content.html

- Click the ``Test Link`` which is represented in the page with the href
  code below.

  ``<a href="http://www.f5demolabs.com/images/f5logo.gif">Test Link</a>``

The link will open as an HTTP URL.


The iRule:
~~~~~~~~~~
.. code-block:: tcl
   
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

Analysis:
~~~~~~~~~
Event/Command details:

- HTTP_REQUEST event is triggered when there is a request for web page
- HTTP_RESPONSE event is triggered when the servers send a web page in response to a request
- STREAM::disable command disables the stream profile attached to the Virtual Server
- STREAM::enable command enables the stream profile attached to the Virtual Server. 

.. NOTE::
   The STREAM::expression command always precedes the STREAM::enable command

- HTTP::header remove "Accept-Encoding" command removes "Accept-Enoding" from the http header
- HTTP::header value Content-Type command returns the type of content present in the payload.
- STREAM::expression command is used to search and replace the first argument with the second argument in the data stream

please check the wiki guide for iRules API in DevCentral at https://devcentral.f5.com/wiki/irules.homepage.ashx for additional info


Rule Details:
~~~~~~~~~~~~~
This rule does the following:

- Disables the stream filter attached to the virtual server for every incoming HTTP request
- Removes the support for encoding in the HTTP header for every incoming HTTP request
- Replaces every occurance of "http://" with "https://" in the payload returned in the 
  HTTP response when the content type is in the text format

.. NOTE::

   The STREAM command requires a stream profile attached to the virtual server 


Testing:
~~~~~~~~
In the BIG-IP, 

- Add the above iRule to the HTTPS VIP
- Access the HTTPS URL https://www.f5demolabs.com/content.html
- Click the ``Test Link`` which is represented in the page with the href
   code below

   ``<a href="https://www.f5demolabs.com/images/f5logo.gif">Test Link</a>``

- Check that the link will open as an HTTPS URL


Review:
~~~~~~~
Some of the application developers are going to continue to include HTTP object references 
in their HTML code no matter how many times you’ve told them not to. In the above lab, we 
have used iRules to find and replace these references.  In the iRule, we used the very
powerful ``STREAM`` command to effortlessly sweep through the response payload 
and replace any instance of http:// with https://. 

Please note that this string matching and replacing is not just limited to http://. It can 
be applied to any type of text.

Bonus Activity:
~~~~~~~~~~~~~~~
Needless to say, ``STREAM`` is an incredibly powerful command, and a
very useful tool in your security arsenal. For example, what if you
also wanted to sanitize Social Security and credit card numbers

.. code-block:: tcl

   STREAM::expression "@\d3-\d2-\d4@***-**-****@ @\d4-\d4-\d4-\d4@xxxx-xxxx-xxxx-xxxx@"

Please refer to https://devcentral.f5.com/wiki/irules.stream.ashx for more details on the 
STREAM feature and its commands. You can also find some examples that show the application 
of the STREAM feature under each command.


   
