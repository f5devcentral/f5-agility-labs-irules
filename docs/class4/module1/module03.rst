Extract JWT Payload into NGINX Variable [jwt]
=====================================================

#. Start an NGINX docker instance with the inject_header app by running the following commands:  This places the inject_header.conf file and inject_header.js files into the running NGINX instance.

   .. code-block:: shell

  EXAMPLE=jwt
  docker run --rm --name njs_example  -v $(pwd)/conf/$EXAMPLE.conf:/etc/nginx/nginx.conf:ro  -v $(pwd)/njs/$EXAMPLE.js:/etc/nginx/example.js:ro -p 80:80 -p 8090:8090 -d nginx

   The nginx.conf will be as follows, notice it includes the njs script and calls the inject_header function for every request:nginx.conf:

   .. code-block:: nginx

     ...

     http {
         js_include example.js;

         js_set $jwt_payload_sub jwt_payload_sub;

         server {
     ...
               location /jwt {
                   return 200 $jwt_payload_sub;
               }
         }
     }

   The njs code adds an HTTP header called Foo with a value of my_foo:

   .. code-block:: js

    function jwt(data) {
        var parts = data.split('.').slice(0,2)
            .map(v=>String.bytesFrom(v, 'base64url'))
            .map(JSON.parse);
        return { headers:parts[0], payload: parts[1] };
    }

    function jwt_payload_sub(r) {
        return jwt(r.headersIn.Authorization.slice(7)).payload.sub;
    }

#. To validate that it is working run the following commands:

   .. code-block:: shell

     curl 'http://localhost/jwt' -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImV4cCI6MTU4NDcyMzA4NX0.eyJpc3MiOiJuZ2lueCIsInN1YiI6ImFsaWNlIiwiZm9vIjoxMjMsImJhciI6InFxIiwienl4IjpmYWxzZX0.Kftl23Rvv9dIso1RuZ8uHaJ83BkKmMtTwch09rJtwgk"
     alice

     docker stop njs_example
