#####################################################
Lab 4 - Complete iRule
#####################################################

Completed iRule
--------------------------------------------------------------------------------------
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
