:orphan:

#####################################################
Lab 2 - Log and Change Headers
#####################################################


Example Code to help you on your journey (Not fully functional):
------------------------------------------------------------------------------------

.. code::

  when HTTP_REQUEST {
    log local0. "Request Headers: [HTTP::header names]"
  }

  when HTTP_RESPONSE {
    log local0. "Response Headers: [HTTP::header names]""
    HTTP:: command to remove header
  }
