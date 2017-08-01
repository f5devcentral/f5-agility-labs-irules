#####################################################
Lab 4 - Stream Profile
#####################################################


Complete iRules
------------------------------------------------------------------------------------
.. code::

	# Stream_iRule
	
	when HTTP_REQUEST {
		HTTP::header remove Accept-Encoding
		STREAM::disable
	}

	when HTTP_RESPONSE {
		STREAM::expression @Damn@Darn@
		STREAM::enable
	}
