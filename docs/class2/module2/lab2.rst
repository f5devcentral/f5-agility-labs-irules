
Lab 2 - DNS Hooks: Amplification attackers
------------------------------------------

Scenario:
~~~~~~~~~

You have a F5 DNS deployed to service DNS queries for external DNS.  To meet the business requirements, you must allow DNS queries from any DNS resolver.
Basic DNSSEC has been implemented as part of your GTM deployment.  As your DNS deployment expands for more applications, you experience a DNS Amplification attack.

A DNS amplification attack takes advantage of features that allow a very small request to return a much larger response.
These attacks also rely on the fact that the attacker can request these large responses on behalf of someone else (the victim).
More specifically, DNS amplification attacks are a popular type of a Distributed Denial of Service (DDoS) attack in which attackers use publicly accessible open DNS resolvers to flood
a target system with DNS response traffic.  One other key piece to this puzzle is that DNS uses the User Datagram Protocol (UDP) to send requests and responses.
An attacker floods your DNS system with an "ANY" query that returns all known information about a DNS zone in a single request.

Figure 1:
~~~~~~~~~
.. image:: /_static/class2/DNS_Resolver.gif
   :scale: 50 %

.. image:: /_static/class2/open_DNS_resolver.gif
   :scale: 50 %

Figure 2:
~~~~~~~~~
.. image:: /_static/class2/amplification_attack.gif
   :scale: 50 %
Here we see an example of an DNS Amplification attack using an open DNS resolver
   
Restraints:
~~~~~~~~~~~

The following restraints complicate the request from the business to relax the enforced security posture:

- Respond to queries from any source, even open resolvers.
- Respond to DNS queries over both TCP and UDP.


Requirements:
~~~~~~~~~~~~~

To meet the business’s objectives while maintaining a strong security policy, an iRule solution must meet the following requirements:

- Checks to see if the query type is "ANY" and responds with a truncated message which will force the legitimate client to use TCP.
- Allows for a flexible and updatable list of networks allowed to do recursive lookups.
- The only host allowed to do recursive lookups is 10.1.10.51


Baseline Testing:
~~~~~~~~~~~~~~~~~

Prior to defining a solution, validate the current behavior:

- RDP to lab jump station
- From the BIG-IP, Select Local Traffic >> Virtual Servers >> Statistics.  Reset the statistics for all the virtual servers.
- Open Terminal application
- From Terminal run the following command against the open resolver (F5 DNS)
 
  .. code-block:: console
    
     f5student@xjumpbox~$ dig f5.com @10.1.10.153 ANY +noall +comments

You should get an answer from the BIG-IP functioning as our open resolver that looks similar to below image, and most significantly is missing any notes about truncation:

  .. image:: /_static/class2/dig_notrunc.png

- From Terminal run the following command against the open resolver (F5 DNS)

  .. code-block:: console
      
     f5student@xjumpbox~$ dig f5.com @10.1.10.153 ANY

The resolver will recursively query the f5.com domain name, and return all records associated with this domain.  Take a look at the MSG SIZE field at the end of the response.

- From the Terminal run the following command against the open resolver (F5 DNS)

  .. code-block:: console
      
     f5student@xjumpbox~$ dig f5.com @10.1.10.153 

Again, the resolver will recursively query for the A records from f5.com.  Take a look at the MSG SIZE field.  The ANY response from f5.com was ~6X the size of the A query.  Now, imagine if the attacker sent a query to a bogus domain which they have populated with thousands of bogus records.  

- From the Terminal run the following command against the open resolver (F5 DNS)

  .. code-block:: console
      
     f5student@xjumpbox~$ dig test1.f5demolabs.com @10.1.10.153 

- Repeat the same command, this time add 'ANY' to the end of the query to request all records for test1.f5demolabs.com 

.. TIP:: 

   In this lab, we have two DNS Express zones defined, f5demolabs.com and badf5demolabs.com.  The above queries validate we are able to resolve names from f5demolabs.com DNSX zone.

- From the BIG-IP, Select Local Traffic >> Virtual Servers >> Statistics.  Check statistics on the ``sec_irules_dns_udp`` and ``sec_irules_dns_tcp`` virtual servers.  At this point, we are forcing any traffic to TCP listener, so all traffic should be hitting the udp virtual server.

With the above steps complete, we have verified that without our iRule solution in place we are able to do the following:
- Recursively resolve queries from any host for any record type, which is perfect for an attacker looking to trigger a DNS amplification attack.
- Resolve queries from DNS Express zones defined on F5 DNS.


The iRule:
~~~~~~~~~~

UDP VIP iRule

.. code-block:: tcl

    when RULE_INIT {
        set static::dns_dbg 1
    }

    when DNS_REQUEST {
      
      if {$static::dns_dbg} {
            log local0. "DNS Question Type: [DNS::question type]"
      }
      
      if { [DNS::question type] eq "ANY" } {
        DNS::answer clear
        DNS::header tc 1
        DNS::return
      }
    }

    when DNS_RESPONSE {
      if {$static::dns_dbg} {
        log local0. "DNS Origin: [DNS::origin] "
      }
      if { [DNS::origin] eq "TCL" } {
        return
      } elseif { [DNS::origin] ne "DNSX" } {
          if {$static::dns_dbg} {
            log local0. "Client IP: [IP::client_addr] "
          }
          if { not [class match [IP::client_addr] eq "admin_datagroup" ] } {
            DNS::drop
          }
      }
    }


TCP VIP iRule

.. code-block:: tcl

  when DNS_RESPONSE {
    if {$static::dns_dbg} {
      log local0. "Client IP: [IP::client_addr], DNS Origin: [DNS::origin]"
    }
    if { [DNS::origin] ne "DNSX" } {
      if { not [class match [IP::client_addr] eq "admin_datagroup" ] } {
        DNS::drop
      }
    }
}


Rule Details:
~~~~~~~~~~~~~

UDP VIP iRule

This first part checks if the DNS query type is "ANY" and responds with a truncated header.
The second part checks to see if the response packet is built from the first logic (origin = TCL).
If yes, then exit and do not process further.
If no, then check if the response is from DNS Express. if it is, allow an answer for non "ANY" type.
If it is not from DNS Express, check to see if it matches the admin_datagroup created for recursive allowed networks.
If it does not match both conditions, then drop.


TCP VIP iRule


Simple logic to check and see if the response is from DNS Express or a part of the admin_datagroup.
If it is not from DNS Express, check to see if it matches the admin_datagroup created for recursive allowed networks.
If it does not match both conditions, then drop.

Testing:
~~~~~~~~
- From the BIG-IP, Select Local Traffic >> Virtual Servers >> Statistics.  Reset the statistics for all the virtual servers.
- Navigate to Local Traffic -> Virtual Servers -> Virtual Server List -> ``sec_irules_dns_udp``
- Click the Resources tab, then the Manage button to the right of the iRules section header
- Move the iRule ``sec_irules_dns_hook-udp`` from the Available box to the Enabled box
- Click Finished
- Open Terminal application
- From Terminal run the following command against the open resolver (F5 DNS)
 
  .. code-block:: console
    
     f5student@xjumpbox~$ dig f5.com @10.1.10.153 ANY +noall +comments

You should get an answer from the BIG-IP functioning as our open resolver that looks below, this time you should see the DNS response has been truncated forcing the client to retry using TCP.

  .. image:: /_static/class2/dig_trunc.png

- Navigate to Local Traffic -> Virtual Servers -> Virtual Server List -> ``sec_irules_dns_tcp``
- Click the Resources tab, then the Manage button to the right of the iRules section header
- Move the iRule ``sec_irules_dns_hook-tcp`` from the Available box to the Enabled box

- From Terminal run the following command against the open resolver (F5 DNS)

  .. code-block:: console
      
     f5student@xjumpbox~$ dig f5.com @10.1.10.153 ANY


This time, you will see you get a truncated reponse over UDP, and attempts to execute the query over TCP fail.  The requests over TCP are failing b/c the iRule is filtering all requests for non DNS Express zones, and only allowing clients in the admin_datagroup whitelist.

- Navigate to Local Traffic -> iRules >> Data Group List, and select admin_datagroup
- Add the address 10.1.10.51 with no value to the list

- From Terminal, repeat the query we just issued in previous step

This time, the query sent over TCP should receive a valid response.  With the client IP added to the admin_datagroup whitelist, the client is now able to execute DNS queries for non DNS Express domains. 

- From BIG-IP return to the data group list, and remove 10.1.10.51 from the address section.

- From the Terminal run the following command against the open resolver (F5 DNS)

  .. code-block:: console
      
     f5student@xjumpbox~$ dig test1.f5demolabs.com @10.1.10.153 

- Repeat the same command, this time add 'ANY' to the end of the query to request all records for test1.f5demolabs.com 

- The last two queries were for records in the f5demolabs.com domain previously defined in a DNS Express zone on the F5 DNS.  So, even though our client is no longer defined in the admin_datagroup, it is still able to use the resolver to resolver entries in the DNS.

- From the BIG-IP, Select Local Traffic >> Virtual Servers >> Statistics.  Check statistics on the ``sec_irules_dns_udp`` and ``sec_irules_dns_tcp`` virtual servers.  
- With the iRules in place, you will see traffic is being picked up on both the TCP and UDP listeners.


Review:
~~~~~~~
It is absolutely bad practice for most organizations to publicly expose open DNS resolvers.  Doing so provides a perfect tool for attackers to trigger amplification attacks against unsuspecting targets.  Attackers take advantage of the fact that DNS leverages UDP, and therefore they can use spoofed IP addresses to trigger massive attacks.  In this lab, we demonstrated how a customer can use iRules to expose an open resolver, but force recursive queries of the type commonly used in amplification attacks (ANY) to use TCP.  TCP, makes these kinds of amplification attacks impossible b/c the attacker would essentially wind up attacking themselves.  Then, we extended our iRules to filter any queries requiring recursive lookups to be filtered against a predefined list of allowed sources.  Finally, our iRules allow all hosts to be able to execute queries against local hosted DNS Express zones without filtering.

Bonus Activity:
~~~~~~~~~~~~~~~
In this lab, we had pretty specific allow/disallow logic in the rules.  However, another approach might be to provide rate limiting on the number of recursive queries we would allow from a given host.  As a bonus activity, see if you can use some of the logic from the HTTP Throttling lab to provide a solution that rate limits all recursive requests ANY requests.
