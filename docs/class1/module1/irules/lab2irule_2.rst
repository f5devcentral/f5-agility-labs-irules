:orphan:

#####################################################
Lab 2 - Log and Change Headers
#####################################################


Here is the code to log to the LTM log file the HTTP::header names:
------------------------------------------------------------------------------------

.. code::

  log local0. "Request Headers: [HTTP::header names]"
