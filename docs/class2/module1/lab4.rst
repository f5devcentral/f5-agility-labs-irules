Lab 4 - Geolocation
-------------------

Attacks can come from anywhere, but there are definitely times when it
can be tracked from specific regions. Geolocation is a powerful tool
that allows iRules to access and make decisions based upon IP location
intelligence information, and there are any number of uses for this
capability. The below iRule example very simply redirects application
traffic to different pools of resources based on where the client is
coming from.

Objectives:

-  Deploy and test the example geolocation iRule code

Lab Requirements:

-  BIG-IP LTM, web server, client (command line cURL)

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

   when CLIENT_ACCEPTED {
       switch [whereis [IP::client_addr] country] {
           US { pool usa_pool }
           CA { pool canada_pool }
           MX { pool mexico_pool }
           KP { pool majestic_unicorn_pool }
           default { pool northamerica }
       }
   }

For the sake of testing this though, we’ll need to replace
``IP::client_addr`` with an ``X-Forwarded-For`` HTTP header that we can
control, and generally alter the iRule for log display only.

.. code-block:: tcl
   :linenos:
   :emphasize-lines: 3-7

   when HTTP_REQUEST {
       set XFF [getfield [lindex [HTTP::header values X-Forwarded-For] 0] "," 1]
       log local0. "continent: [whereis $XFF continent]"
       log local0. "country: [whereis $XFF country]"
       log local0. "state: [whereis $XFF state]"
       log local0. "isp: [whereis $XFF isp]"
       log local0. "org: [whereis $XFF org]"
   }

Apply this iRule to an HTTP virtual server (VIP).

Analysis
~~~~~~~~

-  The iRules ``whereis`` command can take several options, including:

   - ``[whereis [IP::client_addr] continent]``: returns the three-letter
     continent

   - ``[whereis [IP::client_addr] country]``: returns the two-letter
     country code

   - ``[whereis [IP::client_addr] <state|abbrev>]``: returns the state as
     word or as two-letter abbreviation

   - ``[whereis [IP::client_addr] isp]``: returns the carrier

   - ``[whereis [IP::client_addr] org]``: returns the registered
     organization

Testing
~~~~~~~

To test this from a command line, issue a cURL command and include an
X-Forwarded-For header:

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

.. NOTE:: Using ``geoip_lookup <IP_addr>`` on the BIG-IP command line
   will provide geographic location information.

Bonus versions
~~~~~~~~~~~~~~

From a security perspective, the above iRule example doesn’t do
much. You might, for example, want to block all non-US requests.

.. code-block:: tcl
   :linenos:

   when HTTP_REQUEST {
       set XFF [getfield [lindex [HTTP::header values X-Forwarded-For] 0] "," 1]
       if { [whereis $XFF country] ne "US" } {
           drop
           event disable all
       }
   }  
       
Or you might only want to allow access to a small set of countries
that you can maintain in a data group.

.. code-block:: tcl
   :linenos:
   :emphasize-lines: 4-5

   when HTTP_REQUEST {
       set XFF [getfield [lindex [HTTP::header values X-Forwarded-For] 0] "," 1]
       if { not ( [class match [whereis $XFF country] equals list_of_countries ] )} {
           log local0. "[whereis $XFF country] is not a part of accepted countries, traffic is dropped"
           drop
           event disable all
       }
   }

where the data group is a string-based list of two-letter country codes.

.. code-block:: console
   :linenos:

   ltm data-group list_of_countries {
      type string
      members {
           US {}
           CA {}
           IN {}
           LB {}
      }
   }
