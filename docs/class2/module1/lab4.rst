Lab 4 - Client Certificate Inspection
-------------------------------------

Scenario:
~~~~~~~~~

Your company uses smart cards for two-factor authentication.  Users access different resources from a single url and
need to be given access to those resources based on the properties of a client certificate. Users have physical
smart cards and software-based client certificates and authentication decisions will need to be made based on certificate attributes.

Requirements:
~~~~~~~~~~~~~

-  BIG-IP LTM, web server, client browser, SSL server and client certificates

To meet the business’s objectives while still maintaining a strong security policy, an iRule solution must meet the following requirements:

- inspect certificate attribute to give access to correct resource

Certificates:
~~~~~~~~~~~~~

Certificates and keys are provided for you in the lab, but here are test
certificates and private keys.

CA certificate (f5test.local)

.. code-block:: console

   -----BEGIN CERTIFICATE-----
   MIIDzTCCArWgAwIBAgIJAPn9mslcqHAUMA0GCSqGSIb3DQEBCwUAMH0xCzAJBgNV
   BAYTAlVTMQswCQYDVQQIDAJXQTEQMA4GA1UEBwwHU2VhdHRsZTELMAkGA1UECgwC
   RjUxDTALBgNVBAsMBFRlc3QxFTATBgNVBAMMDEY1dGVzdC5sb2NhbDEcMBoGCSqG
   SIb3DQEJARYNZjVAdGVzdC5sb2NhbDAeFw0yMDA2MDQyMDAzNDBaFw0zMDA2MDIy
   MDAzNDBaMH0xCzAJBgNVBAYTAlVTMQswCQYDVQQIDAJXQTEQMA4GA1UEBwwHU2Vh
   dHRsZTELMAkGA1UECgwCRjUxDTALBgNVBAsMBFRlc3QxFTATBgNVBAMMDEY1dGVz
   dC5sb2NhbDEcMBoGCSqGSIb3DQEJARYNZjVAdGVzdC5sb2NhbDCCASIwDQYJKoZI
   hvcNAQEBBQADggEPADCCAQoCggEBAKRNLa6GlVIbOjDEpGtAGfMYD7NCSbUJPyu1
   7833MCBK07whSVxR5hDTU5S7b1P6Q/LGDYAodS5nq5mf+qfCkWikl8OztMqJUUsK
   0bX9+fosBx/4H9ONV1ub9WF60vOkp6cf7y+yzfzfZ1CidrlouV0trNhYSSDVjUNt
   vaJrQIhiBsvZ5j1ebtAh4blmf/kd2W3Aui858eDtY7wL+tusTtgW5kd1ASw10s+3
   IO8pScAeaUwaNn+dfA2YXO1qgYNsWxKu9sBL25wv1xC+UBSlb5KHEAJY92ujnuBM
   utDW6OtLrc1VJwFAXx4qLeZfbdsIqOdqSv2yAMFT/76hyWZ7/ZsCAwEAAaNQME4w
   HQYDVR0OBBYEFNAfa1XirB7icQ+dCrl3vWC2odnhMB8GA1UdIwQYMBaAFNAfa1Xi
   rB7icQ+dCrl3vWC2odnhMAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEB
   AI8+bxjT+zDy8671KseREpHp1/OLfZfZVW5NiDyT40PPXxKbBzZIQRN2tZdVOO3C
   KOrW3vcpo4oKvk3SUKSLGDZan/zFiZCagp8y1U+YTTYlt9QBXv7pjZ9RgZ9Czi3w
   iiD6VG4fsPZWNUJwkUXI947aB0mmm6stVPCcgf20hfeJXhciOO12UWKoK2+eA2FL
   l/dGSOw/pTrYyFxHR0CRZub5vmNh56Mn8vZVZzT/wJdE6e2JY36riftCPgDOf4AY
   pQDxkBqkW2DpKz2f1+PYRJu+c8Q6qtA2ETz8cHNqhQ1s2NuUxqw37iPRs8MyivmV
   3EgQ6yeUw2FP3wfzYRbG574=
   -----END CERTIFICATE-----



Server certificate (www.f5test.local)

.. code-block:: console

   -----BEGIN CERTIFICATE-----
   MIIDXTCCAkWgAwIBAgIEE5upJjANBgkqhkiG9w0BAQsFADBgMQswCQYDVQQGEwJV
   UzEVMBMGA1UEChMMZjV0ZXN0LmxvY2FsMR8wHQYDVQQLExZXZWIgU2VydmVyIENl
   cnRpZmljYXRlMRkwFwYDVQQDExB3d3cuZjV0ZXN0LmxvY2FsMB4XDTIwMDYwNDIw
   MDgwNloXDTMwMDYwMjIwMDgwNlowYDELMAkGA1UEBhMCVVMxFTATBgNVBAoTDGY1
   dGVzdC5sb2NhbDEfMB0GA1UECxMWV2ViIFNlcnZlciBDZXJ0aWZpY2F0ZTEZMBcG
   A1UEAxMQd3d3LmY1dGVzdC5sb2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
   AQoCggEBAMsycgjCKdb0xAgnyHArtPBcfMWhMvoqQFf/kpM5TvbucWOI1gwHmjSj
   /+neod8diKaVYMPZSAojLHaUCD4NOERfSvrANvgxdb60Zkqg6b7FsyXoYZecH6ml
   AScYB7GSL5x9iaGtLpCWgEEdJOYnqkoY0QoWf7Xy/dPYQuxldAsQU4xzuhY7mhRz
   jI9s8oeCHyz0yg500Dq/EB9/unQybLNUNJ3GUZgxJHPbvjP8F8Rove45K2HN2CEN
   fl75Z/0Ku3F2sLiuKCgx48S6+JXcONJyjiFvjdNbwVLq2SAGicHH/ncD7MlygqfC
   SoyxaS+pW9cvHe7h7rnPqzayAm/6Lv8CAwEAAaMfMB0wGwYDVR0RBBQwEoIQd3d3
   LmY1dGVzdC5sb2NhbDANBgkqhkiG9w0BAQsFAAOCAQEAqDSi2UEWAkMhuU9Mh0PR
   aiNkSWsG8+XKqbjRARPvM+l0amtdKJjxkAMThDBHwUFlxj2XoVHs1CHlAKGKVYr/
   SCGFlI+OUz68Ul6BykoNb5dKHDwHvrOqRtZHoeluuykjiJK35JsDE+LDnfXYrZo2
   QpQzO77hEMPCsRaM8owebyGk3JIuDKNFL/jweawRg+JfLiZ3C3Yq+er6gxWGkXLH
   xwG41R8dnY5wGJqEzBh/VkvZuZsfWkWFBJef6u5gFJZ+eEvHdpb1SytfCf2kQQkz
   nOQaiGf9nRDqwY1raOyhvkOrBLth8blwTYZwWy4XDHy1ZZQD9FAfqJAo97OIwRC6
   3A==
   -----END CERTIFICATE-----


Server private key

.. code-block:: console

   -----BEGIN RSA PRIVATE KEY-----
   MIIEpAIBAAKCAQEAyzJyCMIp1vTECCfIcCu08Fx8xaEy+ipAV/+SkzlO9u5xY4jW
   DAeaNKP/6d6h3x2IppVgw9lICiMsdpQIPg04RF9K+sA2+DF1vrRmSqDpvsWzJehh
   l5wfqaUBJxgHsZIvnH2Joa0ukJaAQR0k5ieqShjRChZ/tfL909hC7GV0CxBTjHO6
   FjuaFHOMj2zyh4IfLPTKDnTQOr8QH3+6dDJss1Q0ncZRmDEkc9u+M/wXxGi97jkr
   Yc3YIQ1+Xvln/Qq7cXawuK4oKDHjxLr4ldw40nKOIW+N01vBUurZIAaJwcf+dwPs
   yXKCp8JKjLFpL6lb1y8d7uHuuc+rNrICb/ou/wIDAQABAoIBADeEduextSDIC292
   /yq2pl8txeFxY646MQ5aA8A53jtVdqGNV3497YIIdPl/HJcLSLTLB387NJWgepuD
   YqUhk4gKyT+tmNdDHDqYq4IkaPj4pzPqRA/aVkRRkvkNdbyshlmpaxtDZ/+VP0GL
   JvPDTqGkGik5cHdUBsoEwnQ4W/ZRaP+hrvFDguYlwZAe+iN35AXWdviuU7Iz1dZN
   mcsmpEyqQoHlWvmS15i9IqSkUabbvt/fWCZQTmAQHDc4J+gyYekcLf+ubVgEzB4C
   Yh/cibO+MMLHOw6aG2lzdnAwPephhhsRYvKdC4GqmxHaNMNdnXuI02HpY8ySL2Ue
   cPmlnSECgYEA5gixIlmQTNOTbq0VP0YFs09/GD1lk57rQmXQ4FTTd0t++tSyV/oX
   ugDXeHA10/K3iufaJNfKtj7bUAlux740nqgOqaq/NENiLvF3RMWFVn0UJOO8loHx
   4ZcpuWfSt/6TRgrHg+V+H0OMCEwUcebG6123Wd43b3JipHttLWFxQpkCgYEA4iI+
   4bIN61ptzZDmWc7hvIDdvFnyqotOjlwL5RAucV6W0T6SYCuOJb6UXYeDfoisHQqv
   i5c+oEqVvZHly53+Bx6zRT9zpEhJfDoF929BC3KB44XQDF2MnXzr34gRw0GvJuaR
   P0lZJqXrN93GXGX80bvqU/eMtOST1BoWkPH2FVcCgYB+TMFs+b334KbvOosS7ZBN
   rlU66uLtlXDYSOzRbuGYe1QhxkyRb1g9oR6tGvcDAx3xX3FvjyfWvlZN8I/pja54
   eg9q6rwGpwSuf5ebo9Oc9BnuUzgFbx1uXj/jc3TH3zffWiXHbma8JasqFxOWoj4P
   lqoH5rGLOEOeycHdC8ZS6QKBgQCXr7MQf/h4TANlpfHugijH4oVah9eQcLu0IKhV
   8gHFSFbQazGS0wSZ6vnotzMMWK9jF7zjXQPET+Ob8tb7O7KfogdMxyBSLa8lZmKE
   NJukCx53uVXyRXpCVf5+xe5sVI4iAP2jPxdPJnLe2aPqbPsm0O+BfYdj/APxfcJv
   Xe7dJwKBgQDgeLXskt1ymndPfDy9XphX/DksZThxy3gFZPicns4mTJ7l6VRpoAd3
   tJUawHyG97Gdo6XSfVn4Ge7FhMgskqZxHHgr6dtmxdbdheY4uyZp+Kep5gmVmynq
   2Kz+pBg3E5IaF/A1mxCGEe7EDTZUpgCuTeIRKslBBPGm6ir2vLFNTA==
   -----END RSA PRIVATE KEY-----

Client certificate (user@f5test.local)

.. code-block:: console

   -----BEGIN CERTIFICATE-----
   MIIDPjCCAiYCCQCvJznjluyZfTANBgkqhkiG9w0BAQsFADB9MQswCQYDVQQGEwJV
   UzELMAkGA1UECAwCV0ExEDAOBgNVBAcMB1NlYXR0bGUxCzAJBgNVBAoMAkY1MQ0w
   CwYDVQQLDARUZXN0MRUwEwYDVQQDDAxGNXRlc3QubG9jYWwxHDAaBgkqhkiG9w0B
   CQEWDWY1QHRlc3QubG9jYWwwHhcNMjAwNjA0MjA0OTQ4WhcNMzAwNjAyMjA0OTQ4
   WjBFMQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UECgwY
   SW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
   MIIBCgKCAQEAwNAx+2gDYK5D96Gzy6IQCR0QwgxMg+vDGYmupf9BVBCt1k0jti46
   13kR33X6satcCo6BQSGvs2L3yoKJcHSe+1YwgT9ShuNAUacNNhMJwPdDdxD/0Eiv
   Xix2hLppq8+a6IeXZCKpF/jwvd4E6bkmAiXmui74Gh5uca1fhBS2InjR+dl4XgpN
   V31j/0yVre/2R1+seWSy33RPI4UpXK5alXvVW5GwJwj2mWl+QUlg3ARzrsE3d2tO
   8W6P6RNgMtteFBbeECCgzyVwn07lY8Mx6xpgniYPtPKYeBV02o2YWrhJpqt2j8sC
   bXiIfygoWptnMSZ8y5dA5m7ICWFoh3yamQIDAQABMA0GCSqGSIb3DQEBCwUAA4IB
   AQBoofUvPgL/toZDDZO5uy2XN3kLXHghCm44l5kKlJ3C2qVcDC5HSd1LQLcIRdjz
   PKslhK6HG555BcaBU9TIM3kDPINSeeoUpEgsYxB/iLNrD2oQZtSm7md0D0HWPjiK
   RilFajuwvaOfy98EGed+5PxNhUn/sfa4281r5LDf1tQK1nhxDBxoJrFFs0LF7cW7
   TuAmaZXBr2ih9uMJbKsvluXNHL2VoIvPgB3ZrbTb8YS6OVaSa70OQRr64J7QkXBn
   tyb25/Rgg7nwN8bmp4QGYrO5YQyq3nczhM7MgZupzhpF92RrrkeyG7wDK7MqqCVx
   pEEk+76wUAl7cX8piiu0NWjq
   -----END CERTIFICATE-----

Client private key

.. code-block:: console

   -----BEGIN RSA PRIVATE KEY-----
   MIIEpAIBAAKCAQEAwNAx+2gDYK5D96Gzy6IQCR0QwgxMg+vDGYmupf9BVBCt1k0j
   ti4613kR33X6satcCo6BQSGvs2L3yoKJcHSe+1YwgT9ShuNAUacNNhMJwPdDdxD/
   0EivXix2hLppq8+a6IeXZCKpF/jwvd4E6bkmAiXmui74Gh5uca1fhBS2InjR+dl4
   XgpNV31j/0yVre/2R1+seWSy33RPI4UpXK5alXvVW5GwJwj2mWl+QUlg3ARzrsE3
   d2tO8W6P6RNgMtteFBbeECCgzyVwn07lY8Mx6xpgniYPtPKYeBV02o2YWrhJpqt2
   j8sCbXiIfygoWptnMSZ8y5dA5m7ICWFoh3yamQIDAQABAoIBAQCR1CUpb2a2lbbk
   MNHKXt1f9zK4gRLR59uckgycke04BpFj9t3eqSJp27DP4OxluiQX++X4e+DmfSDK
   cmY+voWLtIllB56EVJZN61nLnySOZLUK9bl1L7QrNtfA1Tic8JzJ59txqeFYNzjl
   cWkn2JfNohrakDGnl4KSybznKb8DXCryB/WvULtTT6/ikqvFWIemVUXaP0w3Q0R5
   49BZMwaiB7Xly8rU5vpLRFQd0l9IWMIFi2aUPnRWXwUSqFE4S1kvJzzpxGv/Z6Cl
   BiaKY1Usy1Kf8DrvKQigDTgymHwUL92px9gXE6PYViTXzKzebYFzq3JY4t7GF1yP
   deMoLagdAoGBAO+rssM9Ehd6Aq0gzmjRWYHofezNLHblCmRCh7pUCFTfXP6HvrUw
   v7i6ERh/CPYp/FTwmcqJGmr5nLxN+FfZlO+yqlrhCVv3G1LDiyQ/kMU70fgt5ecK
   2LaXOBDxsBhzlHJHt77GXZcQu6eXru43fF13mX1YZiQqhcG4rQr+5TSjAoGBAM3z
   N2LKkmZnLRJtlCjifp6P3qIopxPHOUBqj7FALC1MerhTvZMPRib0jbn6rczLs0Bb
   wjThRl3waq9ItfBa/KPnuvwloJxMkrvVcF312sK/wXjfDN0KXyxlBUgFZEO3AUpA
   ck+qlcbm6OgqhPPmy5jn1mtlbT3i+GrXqBNGWCuTAoGAMCYAPbTRI6JBU2KZ1Pjp
   0G1SjvYRDrmowseS2N305ogQ+JlwuJnYilXnBVLQDBQXO0EyxDuS8RbAZBwN3ig6
   AYWVL7ix1qXn+VKLa3bRsK352q/t1eKZ8uSiQNUtGVxu4B6ETXEwcB7OdDbGz9iZ
   xXU3grT1oCJiyK4/JUxb450CgYByufY0llwPp5I4HcrXK7UVZ1fCRZstLWH7PGFn
   gDQb1+rVG/ETJwMRWFJLNBX1a9QjGfqJsqScV/1WP876YfUy6TgEloFuEEn9UN0T
   uo1ux5tjVf24dLqn5G6YvEgqYJvbXSNQtdpRvvgnvOfrZrosJ5oOoaXFP9bazd/X
   POyI+QKBgQDDSKRlYsK9aFY7M1CdHobA+WhVNFuTEhERAeIHgIdawuv2DHB3SuYW
   I/AH9TPqLS/BzuICB0InmhINPRW5hiP1Cm6V/qOjoRBcqypQxPQdI0C+TiwrP9kA
   oUb6v6h4cPy2OFIAT342jck/pgr5/KKf8aNZBziXh3rYJX/hWLKT7w==
   -----END RSA PRIVATE KEY-----

Baseline Testing:
~~~~~~~~~~~~~~~~~
Prior to defining a solution, validate that users do not have the correct access.

- From the client work station, ensure you have access:
   curl -kv https://www.f5test.local.
- You should have full access to the url.


The iRule
~~~~~~~~~

F5 iRules have complete access to the x509 properties of a client certificate during that
authentication and can look at the attribute of the certificate to make decisions.

.. code-block:: tcl

   when RULE_INIT {
       set static::debug 1
   }
   when CLIENTSSL_CLIENTCERT {
       # Example subject:
       # C=US, O=f5test.local, OU=User Certificate, CN=user/emailAddress=user@f5test.local
       set subject_dn [X509::subject [SSL::cert 0]]
       if { $subject_dn != "" } {
           if { $static::debug } { log "Client Certificate received: $subject_dn" }
       }
   }
   when HTTP_REQUEST {
       if { [HTTP::uri] starts_with "/" } {
           if { $subject_dn contains "CN=user.f5test.local" } {
               HTTP::uri /headers.php
           } else {
               reject
           }
       }
   }

Analysis
~~~~~~~~

-  The above iRule inspects the x509 subject value in the client’s
   certificate and makes an access decision based on that value. In this
   very simple example, a specific set of users may access different
   corporate resources hosted behind the same VIP.

Testing
~~~~~~~

-  On the BIG-IP, go to Local Traffic->Profiles->SSL->Client.  In the Client Authentication section of the client SSL
   profile ``Lab4_Clientssl``, set Client Certificate to ``Require``, assign ``f5test.local`` to the Trusted Certificate Authorities option, and click ``Update``.

-  Test accessing the URL https://www.f5test.local from the client. First do not include the client certificate:
    curl -vk https://www.f5test.local

-  You should receive a failed handshake error.  Try again, but include the certificate:
    curl -vk --cert /etc/ssl/certs/f5test.pem https://www.f5test.local

-  You should now be able to pass through to the application.
    In the Resources section of the ``f5test_local`` virtual, add the ``Lab4`` irule.

-  Watch the log file on the BIG-IP:
    tail -f /var/log/ltm
    
-  Access the URL again from the client:
    curl -vk --cert /etc/ssl/certs/f5test.pem https://www.f5test.local
    
-  You should now get a different response page.  Notice the Client Certificate log message on the BIG-IP.
    
