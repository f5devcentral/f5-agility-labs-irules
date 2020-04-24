Lab 2 - HTTP Throttling
-----------------------

Scenario:
~~~~~~~~~

Your company has setup a new web application and did not have the time to develop a WAF 
policy. Due to your company's profile, several bad actors have threatened to attack using 
SlowLoris and SlowPost attacks to create a denial of service.  Although Advanced WAF or ASM can handle 
this very easily and would be the best tool to use, we don't have the resources available 
to test implementing the policy.  In this lab, we are going to use an iRule that throttles 
the number of requests coming into the application.

Restraints:
~~~~~~~~~~~

The following restraints complicate the request to implement the throttling of HTTP requests:

-  You need to understand the number of the requests that would be coming from a single IP address.

-  Some requests may be coming from companies using a single proxy IP address to make the request.  Throttling those requests to only 10 per 10 seconds could impact the ability of a partner company to access the site.  

Requirements:
~~~~~~~~~~~~~

To meet the business's objectives the iRule must meet the following requirements:

-  The rule must keep a single client from making too many HTTP requests to a single VIP thus stopping a SlowLoris or SlowPost attack.

-  The rule should be able to adjust to the multiple customers coming through a proxy IP address.
 

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when RULE_INIT {
      # This defines the maximum requests to be served within the timing interval defined by the static::timeout variable below.
      set static::maxReqs 4;

      # Timer Interval in seconds within which only static::maxReqs Requests are allowed.
      # (i.e: 10 req per 2 sec == 5 req per sec)
      # If this timer expires, it means that the limit was not reached for this interval and the request
      # counting starts over.
      # Making this timeout large increases memory usage.  Making it too small negatively affects performance.
      set static::timeout 30;
   }

   when HTTP_REQUEST {
     # The iRule allows throttling for only sepecific Methods.  You list the Methods_to_throttle
     # in a datagroup.
     # If you need to throttle by URI use an statement like this:
     #                               if { [class match [HTTP::uri] equals URIs_to_throttle] }
     # Note: a URI is everything after the hostname: e.g. /path1/login.aspx?name=user1
     #

     if { [class match [HTTP::method] equals Methods_to_throttle] } {

        # The following expects the IP addresses in multiple X-forwarded-for headers.  It picks the first one.
        if { [HTTP::header exists X-forwarded-for] } {
           set client_IP_addr [getfield [lindex  [HTTP::header values X-Forwarded-For]  0] "," 1]
        } else {
           set client_IP_addr [IP::client_addr]
        }
        # The matching below expects a datagroup named: Throttling_Whitelist_IPs
        # The Condition of the if statement is true if the IP address is NOT in the whitelist.
        if { not ([class match $client_IP_addr equals Throttling_Whitelist_IPs ] )} {
            set getcount [table lookup -notouch $client_IP_addr]
            if { $getcount equals "" } {
                table set $client_IP_addr "1" $static::timeout $static::timeout
                # record of this session does not exist, starting new record, request is allowed.
             } else {
                   if { $getcount < $static::maxReqs } {
                       log local0. "Request Count for $client_IP_addr is $getcount"
                       table incr -notouch $client_IP_addr
                       # record of this session exists but request is allowed.
                   } else {
                        HTTP::respond 403 content {
                             <html>
                             <head><title>HTTP Request denied</title></head>
                             <body>Your HTTP requests are being throttled.</body>
                             </html>
                        }
                   }
            }
         }
      }
   }



Analysis
~~~~~~~~

-  Notice that RULE\_INIT sets up a set of variables that respectively
   define the maximum rate of requests and a timeout value. Alter these values according
   to your local preferences.

-  The iRule creates an internal table for each client source address,
   and an entry for each request is added to this table.

-  As each new request arrives, the iRule counts the number of entries
   in the respective table. If the count doesn’t exceed the maxRate
   threshold, a new entry is created and the request is allowed through.

-  If the request exceeds the maxRate threshold, the iRule returns an
   HTTP error response to the client.


Testing
~~~~~~~

A very simple way to test this iRule implementation is with a cURL
script from the Terminal command line. Here’s a Bash representation
of that script.  We have already put the script on the Ubuntu client and instructions follow the sample code below.

.. code-block:: console
   :linenos:

   #!/bin/bash
   while [ 1 ]
   do
      curl http://www.f5demolabs.com --write-out "%{http_code}\n" --silent -o /dev/null
   done
   
- At the Ubuntu client command line, cd to the ~/scripts directory and run ``bash http_throttling``.
- Notice that you are getting 200 responses from each request.  We will now add the iRule to the VIP.
- Login to BIG-IP from Chrome browser.
- Go to Local->Virtual Servers and select the http virtual server.
- Select the resources tab and select Manage for iRules.
- Select the Lab2_1 irule and move it into Enabled.
- Select Finished.
- To view logging information on the F5 BIG-IP follow these instruction:
- Modify the iRule on the F5 to uncomment the line that states:
    ``log local0. "Request Count for $client_IP_addr is $getcount"``
- Click on Update on the iRule.
- Open ssh, connect to BIG-IP, and enter the bash shell.
- Run a tail of the BIG-IP LTM log from command line as follows:

   ``tail –f /var/log/ltm``

The script will make repeated HTTP GET requests. When it exceeds the
threshold the iRule will generate a 403 error response and prevent
access to the web server until the **timeout** static variable time
is reached. 

- Use the CTRL-C keyboard combination to stop the script.

Bonus version
~~~~~~~~~~~~~

The above iRule presents an extremely simple approach to HTTP
request throttling and is based solely on client source address. The
following bonus example extends that functionality to allow for
throttling of specific URLs.

.. code-block:: tcl
   :linenos:

   when RULE_INIT {
       # The max requests served within the timing interval per the static::timeout variable
       set static::maxReqs 4
       # Timer Interval in seconds within which only static::maxReqs Requests are allowed.  
       # (i.e: 10 req per 2 sec == 5 req per sec) 
       # If this timer expires, it means that the limit was not reached for this interval and    
       # the request counting starts over. Making this timeout large increases memory usage.   
       # Making it too small negatively affects performance.  
       set static::timeout 2
   }
   when HTTP_REQUEST {
       # Allows throttling for only specific URIs. List the URIs_to_throttle in a data group. 
       # Note: a URI is everything after the hostname: e.g. /path1/login.aspx?name=user1
       if { [class match [HTTP::uri] equals URIs_to_throttle] } {
           # The following expects the IP addresses in multiple X-forwarded-for headers. 
           # It picks the first one. If XFF isn’t defined it can grab the true source IP.
           if { [HTTP::header exists X-forwarded-for] } {
               set cIP_addr [getfield [lindex  [HTTP::header values X-Forwarded-For]  0] "," 1]
           } else {
               set cIP_addr [IP::client_addr]
           }
           set getcount [table lookup -notouch $cIP_addr]
           if { $getcount equals "" } {
               table set $cIP_addr "1" $static::timeout $static::timeout
               # Record of this session does not exist, starting new record 
               # Request is allowed.
           } else {
               if { $getcount < $static::maxReqs } {
                   log local0. "Request Count for $cIP_addr is $getcount"  
                   table incr -notouch $cIP_addr
                   # record of this session exists but request is allowed.
               } else {
                   HTTP::respond 403 content {
                   <html>
                   <head><title>HTTP Request denied</title></head>
                   <body>Your HTTP requests are being throttled.</body>
                   </html>
                   }
               }
           }
       }
   }

By running the ``http_throttling_bonus`` script, you are checking HTTP requests
limits against the URL paths in the ``URIs_to_throttle`` datagroup. Here’s a 
Bash representation of that script.

.. code-block:: console
   :linenos:

   #!/bin/bash
   while [ 1 ]
   do
      curl http://www.f5demolabs.com/admin/ --write-out "%{http_code}\n" --silent -o /dev/null
   done   
