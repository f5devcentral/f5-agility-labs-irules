#####################################################
Lab 1 - Create iRule that Parses URI to Route Traffic
#####################################################


Example Code to help you on your journey (Not fully functional):
------------------------------------------------------------------------------------

.. code::

  when HTTP_REQUEST {
	  if {[HTTP::host] equals “dvwa.f5lab.com” }{
		  pool <pool_name>
	  elseif ….
  }

.. code::

  when HTTP_REQUEST {
	  switch [HTTP::host] {
		  “dvwa.f5lab.com” { pool <pool_name>}
  		  ….
  }
