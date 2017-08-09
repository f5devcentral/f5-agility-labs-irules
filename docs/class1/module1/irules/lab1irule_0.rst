:orphan:

#########################################################
Lab 1 - Create an iRule that Parses URI to Route Traffic
#########################################################


Example code to help you on your journey (Not fully functional):
------------------------------------------------------------------------------------

Using an if / then / else / elseif

.. code::

  when HTTP_REQUEST {
	  if {[HTTP::host] equals "dvwa.f5lab.com" }{
		  pool <pool_name>
	  elseif ...
  }

Or using a switch statement

.. code::

  when HTTP_REQUEST {
	  switch [HTTP::host] {
		  "dvwa.f5lab.com" { pool <pool_name>}
      ...
  }
