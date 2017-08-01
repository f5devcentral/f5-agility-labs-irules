#####################################################
Lab 3 - HTTP to HTTPS Redirect
#####################################################


Complete iRules
------------------------------------------------------------------------------------
.. code::

	HTTP_to_HTTPS_iRule

	when HTTP_REQUEST {
		HTTP::redirect "https://[HTTP::host][HTTP::uri]"
	}

	Factory F5 https redirect iRule
	
	when HTTP_REQUEST {
		HTTP::redirect https://[getfield [HTTP::host] ":" 1][HTTP::uri]
	}
