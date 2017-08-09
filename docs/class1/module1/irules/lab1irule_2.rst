:orphan:

#########################################################
Lab 1 - Create an iRule that Parses URI to Route Traffic
#########################################################

Here is the irule statement you need to use to evaluate the HTTP::host against:
------------------------------------------------------------------------------------
.. code::

  if {[HTTP::host] equals "dvwa.f5lab.com"} {
