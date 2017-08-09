:orphan:

#########################################################
Lab 1 - Create an iRule that Parses URI to Route Traffic
#########################################################


Continue to evaluate each of the host names and send the traffic to the correct pool:
--------------------------------------------------------------------------------------
.. code::

	} elseif {[HTTP::host] equals "peruggia.f5lab.com"} {
			pool peruggia_http_pool
	} elseif {[HTTP::host] equals "wackopicko.f5lab.com"} {
			pool wackopicko_http_pool
