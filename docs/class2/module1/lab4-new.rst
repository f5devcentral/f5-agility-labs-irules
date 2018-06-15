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

