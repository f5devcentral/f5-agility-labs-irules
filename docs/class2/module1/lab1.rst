Lab 1 - HTTP Throttling
-----------------------

There are a number of well-known vulnerabilities that focus on attacking
HTTP (web servers) with large numbers of requests, or large request
payloads (ex. Slow Loris, Slow Post) to produce a denial of service. As
mentioned in the introduction, custom code should never replace a solid
security product that can handle these types of vulnerabilities, and
indeed F5’s Application Security Manager (ASM) is well-versed at
protecting against HTTP attacks. That said, you may still want to do
some HTTP request throttling in an iRule, if only for a specific URL,
request method, or any number of other HTTP attributes. Here’s an
example of what that might look like.

Objectives:

-  Deploy and test the example HTTP throttling iRule code

Lab Requirements:

-  BIG-IP LTM, web server and client (Linux command line client
   preferred)
   
-  HTTP throttling scripts under scripts directory in Cygwin Terminal   

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:
   :emphasize-lines: 13,18

    when RULE_INIT {
        # This is the life timer of the subtable object
        # Defines how long this object exists in the subtable
        set static::maxRate 10
        # This defines how long is the sliding window to count the requests. 
        # This example allows 10 requests in 10 seconds
        set static::windowSecs 10
        set static::timeout 30
    }
    when HTTP_REQUEST {
        if { [HTTP::method] eq "GET" } {
            set getCount [table key -count -subtable [IP::client_addr]]
            log local0. "getCount=$getCount"
            if { $getCount < $static::maxRate } {
                incr getCount 1
                table set -subtable [IP::client_addr] $getCount "ignore" $static::timeout     $static::windowSecs
            } else {
                log local0. "[IP::client_addr] has exceeded the number of requests allowed."
                HTTP::respond 501 content "Request blocked Exceeded requests/sec limit."
                return
            }
        }
    }

Apply this iRule to an HTTP virtual server (VIP).

Analysis
~~~~~~~~

-  Notice that RULE\_INIT sets up a set of variables that respectively
   define the maximum rate of requests, the sliding window for timing
   the rate, and a separate timeout value. Alter these values according
   to your local preferences.

-  The iRule creates an internal table for each client source address,
   and an entry for each request is added to this table.

-  As each new request arrives, the iRule counts the number of entries
   in the respective table. If the count doesn’t exceed the maxRate
   threshold, a new entry is created and the request is allowed through.

-  If the request exceeds the maxRate threshold, the iRule returns an
   HTTP error response to the client.

-  The **WindowSecs** static variable defines an idle timeout for each
   request entry, and the **timeout** static variable defines a total
   lifetime for that table entry, irrespective of the idle time.

Testing
~~~~~~~

A very simple way to test this iRule implementation is with a cURL
script from the Cygwin Terminal command line. Here’s a Bash representation
of that script.

.. code-block:: console
   :linenos:

   #!/bin/bash
   while [ 1 ]
   do
      curl http://www.f5test.local --write-out "%{http_code}\n" --silent -o /dev/null
   done
   
Under Cygwin Terminal, cd to scripts directory and run ``bash http_trottling``.
To view logging information, open a tail of the BIG-IP LTM log from command line.

``tail –f /var/log/ltm``

The script will make repeated HTTP GET requests. When it exceeds the
threshold the iRule will generate a 501 error response and prevent
access to the web server until the **timeout** static variable time
is reached. Use the CTRL-C keyboard combination to stop the script.

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
                   # log local0. "Request Count for $cIP_addr is $getcount"  
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
      curl http://www.f5test.local/admin --write-out "%{http_code}\n" --silent -o /dev/null
   done   