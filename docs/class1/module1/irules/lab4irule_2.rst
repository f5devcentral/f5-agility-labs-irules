:orphan:

#####################################################
Lab 4 - Stream Profile
#####################################################


We need to disable encoding and stream on the HTTP_REQUEST
------------------------------------------------------------------------------------
.. code::

	HTTP::header remove Accept-Encoding
	STREAM::disable
