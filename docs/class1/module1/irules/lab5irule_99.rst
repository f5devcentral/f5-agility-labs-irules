#####################################################
Lab 5 - Complete iRule
#####################################################

Completed iRule
--------------------------------------------------------------------------------------
.. code::

	# HTTP_Payload_iRule

	when HTTP_REQUEST {
		HTTP::version 1.0
		HTTP::header remove Accept-Encoding
	}

	when HTTP_RESPONSE {
		HTTP::collect [expr 1024*1024]
	}

	when HTTP_RESPONSE_DATA {
	  set find "Damn"
	  set replace "***"

	  if {[regsub -all $find [HTTP::payload] $replace new_response] > 0} {
	    HTTP::payload replace 0 [HTTP::payload length] $new_response
	  }
	}
