Lab X - DNS Hooks: Amplification Attach
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Scenario:
~~~~~~~~~

You have F5 DNS deployed servicing DNS queries for external DNS.  To meet the business requirements, you must allow DNS queries from any DNS resolver.
Basic DNSSEC has been implemented as part of your GTM deployment.  As your DNS deployment expands for more applications you experience a DNS Amplification attach.
A DNS amplification attack takes advantage of features that allow a very small request to return a much larger response.
These attacks also rely on the fact that the attacker can request these large responses on behalf of someone else (the victim).
More specifically, DNS amplification attacks are a popular type of a Distributed Denial of Service (DDoS) attack in which attackers use publicly accessible open DNS resolvers to flood
a target system with DNS response traffic.  One other key piece to this puzzle is that DNS uses the User Datagram Protocol (UDP) to send requests and responses.
An attacker floods your DNS system with an "ANY" query that returns all known information about a DNS zone in a single request.

Figure 1:
~~~~~~~~~
.. image:: docs/_static/clas2/DNS_Resolver.gif

.. image:: docs/_static/clas2/open_DNS_resolver.gif


Here we see an example of an DNS Amplification attack using an open DNS resolver:

Figure 2:
~~~~~~~~~
.. image:: docs/_static/clas2/amplification_attack.gif


Restraints:
~~~~~~~~~~~

The following restraints complicate the request from the business to relax the enforced security posture:
-	Respond to queries from any source, even open resolvers.
- Respond to DNS queries over both TCP and UDP.


Requirements:
~~~~~~~~~~~~~

To meet the business’s objectives while maintaining a strong security policy, an iRule solution must meet the following requirements:
-	checks to see if the query type is "ANY" and, if so, it responds with a truncated message which will force the legitimate client to use TCP.
-	Allows for flexible updatable list of networks allowed to do recursive lookups.
-	The only host allowed to do recursive lookups is 10.1.10.20


Baseline Testing:
Prior to defining a solution, validate the issue by testing DNS answers UDP “ANY” requests and that any resolver can do recursive lookups.

The iRule:
~~~~~~~~~~

UDP VIP iRule

This first part checks if the DNS query type is "ANY" and responds with a truncated header.
The second part checks to see if the response packet is built from the first logic (origin = TCL)
If yes, then exit and do not process further
If no, then check if the response is from DNS Express...if it is, allow an answer for non "ANY" type
If not from DNS Express, check to see if it matches the admin_datagroup created for recursive allowed networks
If it does not match both conditions, then drop.

.. code-block:: tcl

when DNS_REQUEST {
if { [DNS::question type] eq "ANY" } {
DNS::answer clear
DNS::header tc 1
DNS::return
}
}
when DNS_RESPONSE {
if { [DNS::origin] eq "TCL" } {
return
} elseif { [DNS::origin] ne "DNSX" } {
if { not [class match [IP::client_addr] eq "admin_datagroup" ] } {
DNS::drop
}
}
}


TCP VIP iRule

Simple logic to check and see if the response is from DNS Express or a part of the admin_datagroup
If not from DNS Express, check to see if it matches the admin_datagroup created for recursive allowed networks
If it does not match both conditions, then drop

.. code-block:: tcl

when DNS_RESPONSE {
if { [DNS::origin] ne "DNSX" } {
  if { not [class match [IP::client_addr] eq "admin_datagroup" ] } {
DNS::drop
}
}
}


Testing:
~~~~~~~~

- Send DNS UDP “ANY” queries to the BIG-IP and verify that you receive a truncated response and a subsequent TCP request is initiated.
-	Attempt to do a recursive lookup from any machine other than 10.1.10.20.


Result:
~~~~~~~

-	The “ANY” query should not be allowed, and a TCP connection should be created.
-	The recursive lookup should be denied and no data returned.

