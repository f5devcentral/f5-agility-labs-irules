#########################################################
Lab 1 - Complete iRule
#########################################################

Completed iRule
--------------------------------------------------------------------------------------
.. code::

	# if / elseif version

	when HTTP_REQUEST {
		if {[HTTP::host] equals "dvwa.f5lab.com"} {
			pool dvwa_pool_http
		} elseif {[HTTP::host] equals "peruggia.f5lab.com"} {
			pool peruggia_http_pool
		} elseif {[HTTP::host] equals "wackopicko.f5lab.com"} {
			pool wackopicko_http_pool
		}
	}

	# switch version

	when HTTP_REQUEST {
		switch [HTTP::host] {
			dvwa.f5lab.com { pool dvwa_pool_http }
			peruggia.f5lab.com { pool peruggia_http_pool }
			wackopicko.f5lab.com { pool wackopicko_http_pool }
		}
	}


	# Advanced, data group lookup version!

	when HTTP_REQUEST {
		if { [class match [HTTP::host] equals "hostnames_dg"] } {
			pool [class lookup [HTTP::host] "hostnames_dg"]
		}
	}
