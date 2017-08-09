:orphan:

#####################################################
Lab 5 - HTTP Payload Manipulation
#####################################################


Code to change the version of HTTP and disable compression
------------------------------------------------------------------------------------
.. code::

	HTTP::version 1.0
	HTTP::header remove Accept-Encoding
