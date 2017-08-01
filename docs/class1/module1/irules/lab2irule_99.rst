#####################################################
Lab 2 - Complete iRule
#####################################################

Completed iRule
--------------------------------------------------------------------------------------
.. code::

	# Header_Strip_Log_iRule

	when HTTP_REQUEST {
		log local0. "Request Headers: [HTTP::header names]"
	}

	when HTTP_RESPONSE {
		log local0. "Response Headers: [HTTP::header names]"
		HTTP::header remove Server
	}

	# Advanced - Bonus and prettier

	when HTTP_REQUEST {
		foreach header [HTTP::header names] {
			log local0. "Request Header $header: [HTTP::header $header]"
		}
	}

	when HTTP_RESPONSE {
		foreach header [HTTP::header names] {
			log local0. "Response Header $header: [HTTP::header $header]"
			if {$header equals "Server"} {
				HTTP::header remove $header
			}
		}
		HTTP::header insert Server "Microsoft-IIS/8.0"
	}
