==================================
Hello World [hello]
==================================

#. Start an NGINX docker instance with the hello world app by running the following commands:  This places the hello.conf file and hello.js files into the running NGINX instance.

   .. code-block:: shell

     EXAMPLE=hello
     docker run --rm --name njs_example  -v $(pwd)/conf/$EXAMPLE.conf:/etc/nginx/nginx.conf:ro  -v $(pwd)/njs/$EXAMPLE.js:/etc/nginx/example.js:ro -p 80:80 -p 8090:8090 -d nginx

   The nginx.conf will look as follows:

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

   The njs example.js file is as follows.  Notice it has the 2 functions reference in the nginx.conf file:

   .. code-block:: js

     function version(r) {
        r.return(200, njs.version);
     }

     function hello(r) {
       r.return(200, "Hello world!\n");
     }

#. To see what happens run the following commands from the linux shell:

   .. code-block:: shell

     curl http://localhost/hello
     Hello world!

     curl http://localhost/version
     0.3.9

     # Stopping.
     docker stop njs_example

