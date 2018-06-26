#Lab Environment:

Requirements from ASM Hooks Lab:
- Base Template Blueprint discussed for Agility Labs (lbr-oscrc-f5-v13.1.0.3a-bp-180427)
- BIG-IP version 13.1.0.x (latest build we plan to use for Agility)
- Pre-configure BIG-IP w/ the following:
   - Provision ASM/LTM
   - Create virtual server to pick up traffic destined to hackazon web site
   - Create ASM policy, from Rapid Deployment Template.
      - Add Cross Site Signature Set to policy
      - Enforce all attack signatures
      - Enable Trigger iRule option on the ASM policy General Settings
   - Apply ASM policy to the test virtual server
   - Create sec_irules_asm_hook_1 irule based on above code

Requirements from HTTP Throttling Lab:
-  BIG-IP LTM, web server and client (Linux command line client
   preferred)
-  HTTP throttling scripts under scripts directory in Cygwin Terminal  
-  Need to add Methods_to_throttle String Datagroup with GET in the list of strings

Requirements from Geolocation lab:
- Base Template Blueprint discussed for Agility Labs (lbr-oscrc-f5-v13.1.0.3a-bp-180427)
- BIG-IP version 13.1.0.x (latest build we plan to use for Agility)
- Install and update geolocation databases
- Pre-configure BIG-IP w/ the following:
   - Provision ASM/LTM
   - Create virtual server to pick up traffic destined to hackazon web site
   - Create DoS Profile (iRules_Sec) with following properties:
    - Enable Application Security
    - Enable Trigger iRule option
    - Disable Proactive Bot Detection
     - Set Grace Period = 10secs
    - Enable TPS protections, and tweak Source IP thresholds to 3/3
   - Create local logging profile that with DoS Protection and Bot Defense enabled and logs all requests locally.
   - Apply DoS Profile to the test virtual server
   - create geo-based-pbdswitcher irule based on lab guide
- Add Modify HTTP Headers Add-On to Firefox on the jumpbox
 - create rule to inject X-Forwarded-For header = 5.16.0.1 to URL = http://hackazon.f5demo.com

