Lab 2 - Decode URI
==================

Running inside Docker:

.. code-block:: shell

  EXAMPLE=decode_uri
  docker run --rm --name njs_example  -v $(pwd)/conf/$EXAMPLE.conf:/etc/nginx/nginx.conf:ro  -v $(pwd)/njs/$EXAMPLE.js:/etc/nginx/example.js:ro -p 80:80 -p 8090:8090 -d nginx

nginx.conf:

.. code-block:: nginx

  ...

  http {
      js_include example.js;

      js_set $dec_foo dec_foo;

      server {
  ...
            location /foo {
                return 200 $arg_foo;
            }

            location /dec_foo {
                return 200 $dec_foo;
            }
      }
  }

example.js:

.. code-block:: js

  function dec_foo(r) {
    return decodeURIComponent(r.args.foo);
  }

Checking:

.. code-block:: shell

  curl -G http://localhost/foo --data-urlencode "foo=привет"
  Hello%20G%C3%BCnter

  curl -G http://localhost/dec_foo --data-urlencode "foo=Hello Günter"
  Hello Günter

  docker stop njs_example
