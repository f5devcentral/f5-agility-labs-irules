:orphan:

#####################################################
Lab 5 - HTTP Payload Manipulation
#####################################################


Example Code to help you on your journey (Not fully functional):
------------------------------------------------------------------------------------

.. code::

  when HTTP_RESPONSE {
    HTTP::collect <data amount>
  }
  when HTTP_RESPONSE_DATA {
    regsub <replace what with what?>
    HTTP::release
  }
