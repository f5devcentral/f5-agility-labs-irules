Choosing upstream in stream based on the underlying protocol [stream/detect_http]
=================================================================================

Running inside Docker:

.. code-block:: shell

  EXAMPLE=stream/detect_http
  docker run --rm --name njs_example  -v $(pwd)/conf/$EXAMPLE.conf:/etc/nginx/nginx.conf:ro  -v $(pwd)/njs/$EXAMPLE.js:/etc/nginx/example.js:ro -p 80:80 -p 8090:8090 -d nginx

nginx.conf:

.. code-block:: nginx

  ...

  stream {
        js_include example.js;

        js_set $upstream upstream;

        upstream httpback {
            server 127.0.0.1:8080;
        }

        upstream tcpback {
            server 127.0.0.1:3001;
        }

        server {
              listen 80;

              js_preread  preread;

              proxy_pass $upstream;
        }
  }


example.js:

.. code-block:: js

    var is_http = 0;

    function preread(s) {
        s.on('upload', function (data, flags) {
            var n = data.indexOf('\r\n');
            if (n != -1 && data.substr(0, n - 1).endsWith(" HTTP/1.")) {
                is_http = 1;
            }

            if (data.length || flags.last) {
                s.done();
            }
        });
    }

    function upstream(s) {
        return is_http ? "httpback" : "tcpback";
    }

Checking:

.. code-block:: shell

  curl http://localhost/
  HTTPBACK

  echo 'ABC' | nc 127.0.0.1 80 -q1
  TCPBACK

