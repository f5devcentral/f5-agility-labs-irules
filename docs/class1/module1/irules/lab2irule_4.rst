:orphan:

#####################################################
Lab 2 - Log and Change Headers
#####################################################


Here is the HTTP irule event you need to code for the response:
------------------------------------------------------------------------------------

.. code::

  log local0. "Response Headers: [HTTP::header names]"
