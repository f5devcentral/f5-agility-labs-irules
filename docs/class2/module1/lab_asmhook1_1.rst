Lab X - ASM Hooks
-----------------


Scenario:
~~~~~~~~~

As applications are moved to production in your environment, they are secured with F5 Application Security Manager (ASM), and each have a well-defined security policy.  The policies deployed have been tested, tuned, and are currently part of automated build scripts for deployment.  Amongst other protections, the ASM policy filters all inbound requests for attempts to inject a null byte character from ever reaching the protected web servers.  In general, this protection is well advised, and does not cause issues for most of your customers.  However, it has recently come to the attention of your business representatives that the policy is blocking traffic from one of your most important business partners.  This partner uses automated scripts to scrape your current inventory.  A bug in the partners scraping code is adding extra characters to each request which is violating ASM’s policy.  The business team would like your security team to disable this protection, so that it no longer causes issue for an important partner.  


Restraints:
~~~~~~~~~~~

The following restraints complicate the request from the business to relax the enforced security policy:
- In ASM, the change to relax protection for null byte injection attacks is a global policy setting, and it is enforced across all requests for all users.  Disabling this protection weakens your security policy at a more global level than you are comfortable.
- This baseline policy is deployed across a number of applications and is ingested by the DevOps team when deploying new applications.  The impact of a policy modification may have broader impact than intended.
- Your security and compliance teams run routine scans of your applications looking for vulnerabilities which can be exploited.  By modifying the policy on a global basis, you are certain to get notifications and audit findings for a number of applications.  


Requirements:
~~~~~~~~~~~~~

To meet the business’s objectives while still maintaining a strong security policy, an iRule solution must meet the following requirements:
- Any modifications to the application security policy must only affect the relevant business partner, and only for the in-scope application
   - Partner IP = 10.1.10.4
   - URL = http://hackazon.f5demo.com/product/view?id=[0-100]%00
- If request contains violations other than the violation identified above, the request should be blocked.
- Direct modifications to the ASM security policy are not allowed
- Prior to releasing the in-scope requests to the application server, the request must be sanitized by removing the “%00” null byte string terminating partners request.

Baseline Testing:
~~~~~~~~~~~~~~~~~

Prior to defining a solution, validate the issue by testing the application to validate ASM’s behavior:
1.	RDP to the lab jump station 
2.	From jump station
   a.  Open chrome
   b.  Browse to http://hackazon.f5demo.com/product/view?id=72%00
   c.  Verify that you receive the ASM block page
   d.  Test a few more products (e.g. id 73, 7, 45)
3.	From jump station, open Terminal application
   a.  Curl http://hackazon.f5demo.com/product/view?id=72%00
   b.  Again, verify the content returned is a block page from ASM

4.	Open another Chrome window, and launch BIG-IP Configuration Utility (https://10.1.10.10) 
5.	From BIG-IP, verify violation in event logs:
   a.  Click Security -> Application Security -> Event Logs -> Application -> Requests
   b.  Examine requests for [HTTP] /product/view
   c.  Check icon on request, then click All Details in the request detail to verify the Request Status is blocked 


The iRule:
~~~~~~~~~~


 .. code-block:: tcl 
    :linenos:

when ASM_REQUEST_DONE {
    set asm1_debug_verb 1
    set asm1_debug 1
    
    if { [ASM::status] equals "blocked" } {
        
        if { $asm1_debug_verb } { 
            log local0. "Violation count: [ASM::violation count] "
            log local0. "Violation names: [ASM::violation names] "
            log local0. "Violation attack types: [ASM::violation attack_types] "
            log local0. "Violation details: [ASM::violation details] "
        }
        
        if { [ASM::violation count] <= 1 } {
        # Allow only if this request only violates a specific element of the policy 
            if { [lindex [ASM::violation names] 0] equals "VIOLATION_HTTP_SANITY_CHECK_FAILED" } { 
                if {$asm1_debug} {
                    log local0. "ASM_OVERRIDE: HTTP Request Blocked by ASM with SANITY CHECK VIOLATION, URI = [HTTP::uri] "
                }
                if { [HTTP::uri] starts_with "/product/view?id" && [HTTP::uri] ends_with "%00" } {
                    if { $asm1_debug } {
                        log local0. "ASM_OVERRIDE: URI Request pattern matches override request"
                    }  
                    
                    if { [ASM::client_ip] equals "10.1.10.4" } {
                        if { $asm1_debug } {
                            log local0. "ASM_OVERRIDE: Partner IP: [ASM::client_ip] matches override request" 
                        }
                        #we have a request that matches the OVERRIDE request, override and modify
                        set new_uri [string trimright [HTTP::uri] "%00"]
                        HTTP::uri $new_uri
                        ASM::unblock
                        if { $asm1_debug } {
                            log local0. "ASM_OVERRIDE: Modified request URI, new uri = [HTTP::uri]"
                            log local0. "ASM_OVERRIDE: Unblocking request and releasing to server"
                        }
                    }
                }    
            }
        }
        else {
            if { $asm1_debug } {
                log local0. "ASM:OVERRIDE: Request contains multiple violations, will not override sec policy"
            }
        }
    }
}


Analysis:
~~~~~~~~~

ASM Event/Command Details:
- ASM_REQUEST_DONE event is triggered after ASM has finished processing the request and found all violations of the ASM policy.
- [ASM::violations] command will return the list of violations found in the request or response with details on each violation
- ASM::unblock command overrides the blocking action for a request that had blocking violation

Rule Details:
~~~~~~~~~~~~~

The rule does the following:
- Inspects the blocking status of the request.  If the request was blocked, the rule validates that request only contains a single violation, the violation is the one which  approval has been given to override (VIOLATION_HTTP_SANITY_CHECK_FAILED), and the request originates from the expected business partner.
- If the request matching the above conditions, the rule will then do the following:
   - Strip the expected violation from the request
   - Unblock the request

Testing:
~~~~~~~~

1.	From BIG-IP Configuration Utility, open Local Traffic -> Virtual Servers, select Hackazon_protected_virtual, click the Resources tab, in the iRules section, click Manage.  Move sec_irules_asm_hook_1 from Available section to the Enabled section, then click the Finished button.
2.	From the Jump Station, open the Terminal application and SSH to the BIG-IP: ssh root@10.1.10.10.
   a.  From BIG-IP bash shell, run the following command ‘tail -f /var/log/ltm’
3.	From the Jump Station, re-open the Chrome window used in the Baseline Testing section, and again browse to http://hackazon.f5demo.com/product/view?id=72%00
   a.  Earlier, this request was receiving an ASM block page.  Now, you should be getting access to the page.
4.	From the SSH session, review the log messages associated with the above request.  Details on the request, and the override decision should be present in the logs.
5.	From BIG-IP, verify violation in event logs:
   a.  Click Security -> Application Security -> Event Logs -> Application -> Requests
   b.  Examine requests for [HTTP] /product/view
   c.  Check icon on request, then click All Details in the request detail to verify the Request Status is unblocked

6.	Test additional conditions:
   a.	From Chrome Window, modify the request to include an additional violation, http://hackazon.f5demo.com/product/view?id<script>=72%00
      - This request should receive a block page, b/c it contains violations which have not been approved per override request
b.	From Chrome window, send requests for additional URLs matching the override pattern, http://hackazon.f5demo.com/product/view?id=73%00, http://hackazon.f5demo.com/product/view?id=7%00


Review:
~~~~~~~

While a relatively simple scenario, the above demonstrates how you can use iRules in concert with F5 ASM to handle special situations.  The example above, if relaxed directly by ASM policy tweaks, would have required a broader weakening of an organization’s application security policy.  Also, this type of change, when deployed through a policy re-configuration, often has downstream impact on orchestration and automation tools, and can lead to false positives with vulnerability.  Using an iRule, we were able to temporarily override the security policy without, mitigate the exposed vulnerability, and meet the requirements outlined by the business representatives.

#Lab Environment:
#•	Base Template Blueprint discussed for Agility Labs (lbr-oscrc-f5-v13.1.0.3a-bp-180427)
#•	BIG-IP version 13.1.0.x (latest build we plan to use for Agility)
#o	Pre-configure BIG-IP w/ the following:
#	Provision ASM/LTM
#	Create virtual server to pick up traffic destined to hackazon web site 
#	Create ASM policy, from Rapid Deployment Template.
#•	Add Cross Site Signature Set to policy
#•	Enforce all attack signatures
#•	Apply ASM policy to the test virtual server
#	Create sec_irules_asm_hook_1 irule based on above code
#•	Add Hackazon web application to new Ubuntu machine (existing or new TBD)
