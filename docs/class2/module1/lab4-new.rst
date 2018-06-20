Lab 4 - Geolocation
-------------------

Scenario:
~~~~~~~~~

All of your critical applications are protected by F5 Advanced Web Application Firewall (AWAF), and leverage F5's Layer 7 DoS feature to mitigate bot activity and protect application resources from layer 7 volumetric attacks.  To simplify, the initial deployment, the application security team elected to disable F5's Proactive Bot Defense (PBD) feature.  Recently, the business analaysis team has noticed a significant increase in application traffic from Russia, and believe much of this traffic to be bot related activity.  Because this traffic is having an negative impact on the business's ability to analyze data, and increasing load on the server infrastructure, the business is requesting that more aggressive action be taken on traffic sourced from Russia.  The security team would like to leverage Proactive Bot Defense for this traffic to block simple automated bot activity.

Restraints:
~~~~~~~~~~~
The following restraints complicate this request from the business:

- AWAF DoS Profile allows you to whitelist/blacklist geolocations globally across the DoS profile, and allows for specific thresholds to be defined for geolocations for Transaction Per Second (TPS) and Stress-based protections.  However, it does not allow for per geolocation enabling/disabling of PBD by default.

Requirements:
~~~~~~~~~~~~~
To meet the business’s objectives while still maintaining a strong security policy, an iRule solution must meet the following requirements:

- Proactive Bot Defense should be enabled for all traffic from Russia, but disabled for traffic initiated from everywhere else
- Bot Signature protection should remain enforced for all traffic
- Selectively enabling PBD should **not** affect any of the existing L7DoS protections currently enforced.

Baseline Testing:
~~~~~~~~~~~~~~~~~
Prior to defining a solution, validate the issue by testing the application to validate AWAF's current behavior:
- RDP to the lab jump station 
- From the jump station
 
 - Open Terminal application
 - From Terminal run the following command against the test web application
 ..code-block:: console
    
    f5student@xjumpbox~$ curl -k http://hackazon.f5demo.com/ -H "X-forwarded-for: 5.16.0.1" | grep -i ?type=

- The result of the test should look similar to below, with grep returning no match, and the object response size ~64k

.. image:: /_static/class2/pbd_baseline_test.png
   :width: 1200

- PBD is not active, and not responding to HTTP request with javascript challenge
- From Terminal, run the same command but change the value of the ``X-forwarded-for`` header to be 2.2.2.2
- Currently, there are no on-going L7 DoS attacks, so the behavior for traffic sourced from Russia should match the behavior of all other geolocations, and no proactive bot defense challenges should be issued.


The iRule:
~~~~~~~~~~~

.. code-block:: tcl 
   :linenos:

   when CLIENT_ACCEPTED {
   set geopbd_debug_verb 1
   set geopdb_debug 1
}

when HTTP_REQUEST {
    if { [HTTP::header exists "X-Forwarded-For"] } {
        set XFF [getfield [lindex [HTTP::header values X-Forwarded-For] 0] "," 1]
    }
    else {
        set XFF [IP::client_addr]
    }

    if {$geopbd_debug_verb} {
        log local0. "Coninent: [whereis $XFF continent]"
        log local0. "Country: [whereis $XFF country]"
        log local0. "State: [whereis $XFF state] "
        log local0. "ISP: [whereis $XFF isp] "
        log local0. "Org: [whereis $XFF org] "
    }
    
    if {!([whereis $XFF country] equals "RU") or !([whereis [IP::client_addr] country] equals "RU")} {
        if {$geopdb_debug} {
            log local0. "De-activating PBD: Not Russia source"
        }
        BOTDEFENSE::disable
    }

}

when BOTDEFENSE_ACTION {

   #catch the inbound status
   if {$geopdb_debug} {
     log local0. " Geolocation Country: [whereis $XFF country] "
     log local0. " Bot Defense Status: [BOTDEFENSE::reason] "
     log local0. " Bot Defense Action: [BOTDEFENSE::action] "
   }
   
}

Analysis:
~~~~~~~~~
Event/Command details:

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

- ``BOTDEFENSE`` command enables or disables bot defense processing
- ``BOTDEFENSE_ACTION`` event is triggered after the HTTP request has been processed, and just prior to taking action on transaction.  The event is triggered whenever PBD is enabled, if a DoS L7 attack is configured to trigger PBD, or when a Bot Signature was detected on the request.
- ``BOTDEFENSE::reason`` returns the reason the for the bot defense action
- ``BOTDEFENSE::action`` returns the action to be taken by bot defense feature

Rule Details:
~~~~~~~~~~~~~
This rule does the following:
.. NOTE::
- This rule depends on the following to have been previously configured:
 - DOS Profile, iRules_Sec, created with the following options:
  - Proactive Bot Defense: disabled
  - Bot Signatures: Enabled, with HTTP Crawler Libary Signatures set to Report
  - TPS protections: Enabled, Source IP TPS thresholds set to 3, and mitigation set to Request Blocking, Rate Limit.
All of these settings have been configured for you as part of lab setup  

- Inspects the inbound X-Forwarded-For header or Client IP address, and performs a geolocation lookup on the value.  If either the XFF or the Client IP do **not** match the Russia country code, "RU", then botdefense is disabled. Otherwise Bot Defense is enabled.
- Logs the geolocation information on to a local logger
- Logs the botdefense reason and action to a local logger

Testing:
~~~~~~~~~
- From BIG-IP UI, make the following changes to the configuration:
 - Security -> DoS Protection -> DoS Profiles -> iRules_Sec -> Application Security Tab
  - Click the Proactive Bot Defense button, and set the Operation Mode to Always
  - Click Update
 - Local Traffic -> Virtual Servers -> Virtual Server List -> ``vs_hackazon_http``
  - Click the Resources tab, then the Manage button to the right of the iRules section header
  - Move the iRule ``sec_irules_geobased_pbdswitcher`` from the Available box to the Enabled box
  - Click Finished

- Open Terminal application, and create a new tab, then run following command

 ..code-block:: console 
    
    f5student@xjumpbox~$ ssh root@10.1.1.245

- From BIG-IP console run the following command:
 
 ..code-block:: console 
    
    f5student@xjumpbox~$ tail -f /var/log/ltm 

- On original Terminal Application tab, run the following command:
 ..code-block:: console
    
    f5student@xjumpbox~$ curl -k http://hackazon.f5demo.com/ -H "X-forwarded-for: 5.16.0.1" | grep -i ?type=

- Response should look similar to below image.  You should see that PBD has injected a javascript challenge, and the response body should be ~5.8K

.. image:: /_static/class2/pbd_test1.png
   :width: 1200

- From Terminal, run the same command but change the value of the ``X-forwarded-for`` header to be 2.2.2.2

- Currently, there are no on-going L7 DoS attacks, so the behavior for non Russian sources should be no look different.  You will see the response is missing the javascript injection, and response body is ~64k

- From BIG-IP UI, view the Bot Defense logs:
 - Security -> Event Logs -> Bot Defense -> Requests
 - In this log, look at requests from ``5.16.0.1`` and ``2.2.2.2``
  - You will see both requests are properly classified as bots, but only requests from ``5.16.0.1`` are challenged 

- On Xubuntu Jumpbox, open another Firefox tab
 - browse to http://hackazon.f5demo.com/

- Return to BIG-IP Bot Defense log
 - Notice browser issued requests will source from 10.1.10.51, and will show the following:
  - Request Status = Legal
  - Action = allow
  - Reason =  Bot Defense Inactive

.. NOTE::
Bot Defense is inactive, because the request wasnt sourced from "Russia", and we have disabled PBD.

- Return to Firefox, and right click the Firefox Modify Header Add-on on the right-side of the screen
 - Select Open options page
  - Scroll all the way to buttom of options screen, and click the disable box in the rule for http://hackazon.f5demo.com, verify the box turns blue.  This enables insertion of X-Forwarded-For header in browser request
- Again, browse to http://hackazon.f5demo.com

- Return to BIG-IP Bot Defense log:
 - Notice browser issued requests will source from 5.16.0.1, and will show the following:
  - Geolocation = RU
  - Request Status = Legal
  - Action = browser_challenged (on request for first object), and allow on subsequent requests
  - Reason = No Valid Cookie: Challenge is possible (on request for first object), and Valid Cookie: No need to review on subsequent requests

Review:
~~~~~~~
Geolocation, while not foolproof, is often an important piece of context about a user or device.  Proactive Bot Defense is a very powerful feature for mitigating bot and automated activity, but sometimes challenging to implement in a single broad stroke.  In the above lab, we have used iRules to take advantage of additional context gained through the iRule geolocation commands to leverage, in a targeted manner, a very powerful security feature.  This is precisely the kind of challenge iRules are best suited for, stitching together pieces of information and features to deliver a solution customized to solve a business challenge.


Bonus Activity:
~~~~~~~~~~~~~~~
On of our existing requirements was to not change any of our existing L7DoS protections.  In the lab, we demonstrated, changes via iRule didnt affect Bot Signatures.  As a bonus, you can also verify the iRule enforced PBD for Russian sources also doesn't impair the pre-existing L7DoS protections configured in the DoS profile.

- Return to Firefox, and right-click the Firefox Modify Header Add-on on the right-side of the screen
 - Again, click the Disable button, this time turning it gray
- From browser tab opened to http://hackazon.f5demo.com, click the refresh icon rapidly for ~30 seconds
 - Eventually, you will see requests beginning to fail.  This is the L7DoS protection kicking in and rate limiting requests from non-Russian sources.
- Return to BIG-IP UI:
 - Security -> Event Logs -> DoS -> Application Events
  - You should see a L7DoS attack has been triggered and detected by Source IP TPS

- Repeat same steps, but after re-enabling the X-Forwarded-For header in browser add-on
 - Again, you should be able to trigger an attack, but this time using a Russian source.

With the above steps, you have demonstrated that you can inject PBD challenges from sources from a given geolocation, while maintaining all pre-existing protections.  We have just used more context, to enable more security, using an iRule!
 
