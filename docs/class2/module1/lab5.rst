Lab 5 - IP Reputation
---------------------

IP Reputation is a subscription-based service that provides an F5 BIG-IP
with an active and constantly updating set of data for known "bad
actors". In this case we’re going to look at the source address (or in
our test case an X-Forwarded-For HTTP header) to determine if this is a
known bad guy. If you don’t already have an IP Reputation subscription,
contact your local F5 representative for a time-limited evaluation
license.

Objectives:

-  Deploy and test the example IP reputation iRule code

Lab Requirements:

-  BIG-IP LTM, web server, client (command line cURL)

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when HTTP_REQUEST {
       # Use [HTTP::header values "X-Forwarded-For"] for testing
       #set iprep_categories [IP::reputation [IP::client_addr]]
       set iprep_categories [IP::reputation [HTTP::header values "X-Forwarded-For"]]
       set is_reject 0
       if { $iprep_categories contains "Windows Exploits" } {
           set is_reject 1
       }
       if { $iprep_categories contains "Web Attacks" } { 
           set is_reject 1
       }
       if { $iprep_categories contains "Scanners" } { 
           set is_reject 1
       }
       if { $iprep_categories contains "Proxy" } { 
           set is_reject 1
       }
       if { $is_reject } {
           log local0. "Attempted access from malicious IP address [HTTP::header values "X-Forwarded-For"]($iprep_categories) - rejected" 
           HTTP::respond 200 content "<HTML><HEAD><TITLE>Rejected Request</TITLE></HEAD><BODY>The request was rejected   . <BR>Attempted access from malicious IP address</BODY></HTML>"
       }
   }

Apply this iRule to an HTTP virtual server (VIP).

Analysis
~~~~~~~~

-  With an active subscription to the IP Reputation service, iRules have
   access to a wealth of near real-time information about bad actors,
   including exploit sites, scanners, proxies, and others.

-  With this example iRule, a detected bad site will immediately
   generate an HTML failure page and reject access to the web
   application.

Testing
~~~~~~~

Just like the geolocation lab, to test this from a command line,
issue a cURL command and include an ``X-Forwarded-For`` header:

``curl http://www.f5test.local -H "X-Forwarded-For: 186.64.120.104"``

Here are a few bad IP addresses to test:

- 103.4.52.150
- 101.200.81.187
- 103.19.89.118
- 103.230.84.239

Here are a few bots to test:

- 187.174.252.247
- 188.120.224.250
- 188.219.154.228
- 188.241.140.212

.. NOTE:: Using ``iprep_lookup <IP_addr>`` on the BIG-IP command line
   will provide ip reputation information.