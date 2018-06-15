Lab 4 - Geolocation
-------------------

Scenario:
~~~~~~~~~

All of your critical applications are protected by F5 Advanced Web Application Firewall (AWAF), and leverage F5's Proactive Bot Defense (PBD) feature to mitigate bot activity and automated attacks against the application.  However, as precautionary measure, PBD is configured to operate only once a L7 DoS attack has been detected.  Recently, the business analaysis team has noticed a significant increase in application traffic from Russia, and believe much of this traffic to be bot related activity.  Because this traffic is having an negative impact on the business's ability to analyze data, and increasing load on the server infrastructure, the business is requesting that more aggressive action be taken on traffic sourced from Russia.

Restraints:
~~~~~~~~~~~
The following restraints complicate this request from the business:

- AWAF DoS Profile allows you to whitelist/blacklist geolocations globally across the DoS profile, and allows for specific thresholds to be defined for geolocations for Transaction Per Second (TPS) and Stress-based protections.  However, it does not allow for per geolocation enabling/disabling of PBD by default.

Requirements:
~~~~~~~~~~~~~
To meet the business’s objectives while still maintaining a strong security policy, an iRule solution must meet the following requirements:

- Proactive Bot Defense should be enabled for **all** traffic to critical applications, but only enabled in this fashion for traffic originated from Russia.

Baseline Testing:
~~~~~~~~~~~~~~~~~
Prior to defining a solution, validate the issue by testing the application to validate AWAF's current behavior:
- RDP to the lab jump station 
- From the jump station
 
 - Open Terminal application
 - From Terminal run the following command against the test web application
 ..code-block:: console
    
    f5student@xjumpbox~$ curl -kv https://hackazon.f5demo.com/ -H "X-forwarded-for: 1.1.1.1"

.. TODO::
   Need to add steps to verify PBD not running
   Need change IP address to something that matches Russia geo

- From Terminal, run the same command but change the value of the ``X-forwarded-for`` header to be 2.2.2.2
- Currently, there are no on-going L7 DoS attacks, so the behavior for traffic sourced from Russia should match the behavior of all other geolocations, and no proactive bot defense challenges should be issued.


The iRule:
~~~~~~~~~~~

.. code-block:: tcl 
   :linenos:


Analysis:
~~~~~~~~~

Rule Details:
~~~~~~~~~~~~~

Testing:
~~~~~~~~~

Review:
~~~~~~~







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
