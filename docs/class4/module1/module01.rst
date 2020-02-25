==================================
Lab 1 - Hello World
==================================

Running inside Docker:

.. code-block:: shell

  EXAMPLE=hello
  docker run --rm --name njs_example  -v $(pwd)/conf/$EXAMPLE.conf:/etc/nginx/nginx.conf:ro  -v $(pwd)/njs/$EXAMPLE.js:/etc/nginx/example.js:ro -p 80:80 -p 8090:8090 -d nginx

nginx.conf:

.. code-block:: nginx

  load_module modules/ngx_http_js_module.so;

  events {}

  http {
    js_include example.js;

    server {
      listen 80;

      location /version {
         js_content version;
      }

      location /hello {
        js_content hello;
      }
   }
 }

example.js:

.. code-block:: js

  function version(r) {
    r.return(200, njs.version);
  }

  function hello(r) {
    r.return(200, "Hello world!\n");
  }

Checking:

.. code-block:: shell

  curl http://localhost/hello
  Hello world!

  curl http://localhost/version
  0.2.4

  # Stopping.
  docker stop njs_example

