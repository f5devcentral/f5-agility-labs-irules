:orphan:

#####################################################
Lab 5 - HTTP Payload Manipulation
#####################################################


Perform a regsub on the payload and replace with new text.
------------------------------------------------------------------------------------
.. code::

  if {[regsub -all $find [HTTP::payload] $replace new_response] > 0} {
    HTTP::payload replace 0 [HTTP::payload length] $new_response
