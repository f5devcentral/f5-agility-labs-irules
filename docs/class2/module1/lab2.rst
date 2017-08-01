Lab 2 - Client Certificate Inspection
-------------------------------------

The use of smart card technology to perform client certificate
authentication is, arguably, one of the most secure and reliable
two-factor authentication systems in use today. Even without a physical
smart card, software-based client certificates still offer an advantage
over many other authentication mechanisms. F5 iRules have complete
access to the x509 properties of a client certificate during that
authentication, so there is almost no limit to what can be done here.
The following iRule is a very simple, but very power example of some of
that capability.

Objectives:

-  Deploy and test the example client certificate inspection iRule code

Lab Requirements:

-  BIG-IP LTM, web server, client browser, SSL server and client
   certificates

The iRule
~~~~~~~~~

.. code-block:: tcl
   :linenos:

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
           if { $subject_dn contains "CN=Whitfield Diffe" } {
               HTTP::uri /whitfielddiffe/index.html
           } elseif { $subject_dn contains "CN=Martin Hellman" } {
                   HTTP::uri /martinhellman/index.html
           } {
                   reject
           }
       }
   }

Certificates and keys are provided for you in the lab, but here are test
certificates and private keys.

CA certificate (f5test.local)

.. code-block:: console

   -----BEGIN CERTIFICATE-----
   MIIDqTCCApGgAwIBAgIJAJoOn2YSIE76MA0GCSqGSIb3DQEBCwUAMFsxCzAJBgNV
   BAYTAlVTMRUwEwYDVQQKEwxmNXRlc3QubG9jYWwxHjAcBgNVBAsTFUNlcnRpZmlj
   YXRlIEF1dGhvcml0eTEVMBMGA1UEAxMMZjV0ZXN0LmxvY2FsMB4XDTE3MDcxMjA3
   MzkyOVoXDTIwMDUwMTA3MzkyOVowWzELMAkGA1UEBhMCVVMxFTATBgNVBAoTDGY1
   dGVzdC5sb2NhbDEeMBwGA1UECxMVQ2VydGlmaWNhdGUgQXV0aG9yaXR5MRUwEwYD
   VQQDEwxmNXRlc3QubG9jYWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIB
   AQDVrAXCQS0w9sjGRkwiPYpHmc+aMff6HwHYH6aNwIg93P4a5wEUmIFh+ym4aJjf
   Tmfpsnk2fmbHpygfZU2LEhcxXQuiKB/1euXYizXqlESfFROIScQnCn59Ph+FukVB
   6eNV0+yc0mVpvu5u9LViZCICYGPaqoIOquPF9r9LoP1j1ZutGtHkt0VjiR0/uTiE
   BWo3RVuVi/Q6hack2+uhMrepGN045ilCJWfTnfBJoFFE/d6JVanccD9LAyuUxFbl
   ReSxs6aQNb/nZ/eO1gGts9W16E5XaFv+72wbVgSesTxjXiWnCYnGef/qHkCMxnXg
   /bLRjWzYeBeiHHLUWUp2wfo/AgMBAAGjcDBuMC8GCWCGSAGG+EIBDQQiFiBPcGVu
   U1NMIGdlbmVyYXRlZCBDQSBjZXJ0aWZpY2F0ZTAdBgNVHQ4EFgQUp1RR2qzOWUoZ
   T921koMLiOwHOFIwDwYDVR0TAQH/BAUwAwEB/zALBgNVHQ8EBAMCAYYwDQYJKoZI
   hvcNAQELBQADggEBAG29bNrTH9DSpe92p6/GBMAvyPku+sB7ksXn5Ww6PN+isUZo
   NHY9xfe6GqRVnxafIoad4GkOghwNLFnb2DSzTY7mVGLtlh6oz3sYhbmnh8d8CdKT
   OSQr+rsnZjaQHPX4SK4+PXkWDi6OlAGOAHAx7llOmk5yyIvQ8EsgnOS8MzAFoxJI
   /SSkxFtZ6WC5sYfaEX/hz+7OaDO/ck62u/0Xvy9QYGn9Noib1+25Jz1Ti77znB9k
   FpLnQcaLIOsNChX/MbOQ2m9A0AFKYWKl4uyK2LxItxF1nld3gQYLEKJkRwy45TxR
   4tNLuR5MCYspKEKkdZFs97xZhQyHkQhBekwthNg=
   -----END CERTIFICATE-----

Server certificate (www.f5test.local)

.. code-block:: console

   -----BEGIN CERTIFICATE-----
   MIIEmzCCA4OgAwIBAgIBDDANBgkqhkiG9w0BAQsFADBbMQswCQYDVQQGEwJVUzEV
   MBMGA1UEChMMZjV0ZXN0LmxvY2FsMR4wHAYDVQQLExVDZXJ0aWZpY2F0ZSBBdXRo
   b3JpdHkxFTATBgNVBAMTDGY1dGVzdC5sb2NhbDAeFw0xNzA3MTIxMjA3MjNaFw0y
   MDA1MDExMjA3MjNaMGAxCzAJBgNVBAYTAlVTMRUwEwYDVQQKEwxmNXRlc3QubG9j
   YWwxHzAdBgNVBAsTFldlYiBTZXJ2ZXIgQ2VydGlmaWNhdGUxGTAXBgNVBAMTEHd3
   dy5mNXRlc3QubG9jYWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDL
   MnIIwinW9MQIJ8hwK7TwXHzFoTL6KkBX/5KTOU727nFjiNYMB5o0o//p3qHfHYim
   lWDD2UgKIyx2lAg+DThEX0r6wDb4MXW+tGZKoOm+xbMl6GGXnB+ppQEnGAexki+c
   fYmhrS6QloBBHSTmJ6pKGNEKFn+18v3T2ELsZXQLEFOMc7oWO5oUc4yPbPKHgh8s
   9MoOdNA6vxAff7p0MmyzVDSdxlGYMSRz274z/BfEaL3uOSthzdghDX5e+Wf9Crtx
   drC4rigoMePEuviV3DjSco4hb43TW8FS6tkgBonBx/53A+zJcoKnwkqMsWkvqVvX
   Lx3u4e65z6s2sgJv+i7/AgMBAAGjggFjMIIBXzA3BglghkgBhvhCAQ0EKhYoT3Bl
   blNTTCBnZW5lcmF0ZWQgd2ViIHNlcnZlciBjZXJ0aWZpY2F0ZTAdBgNVHQ4EFgQU
   boBGjDryjx/CGFvQT1eZTJ3VNq8wgY0GA1UdIwSBhTCBgoAUp1RR2qzOWUoZT921
   koMLiOwHOFKhX6RdMFsxCzAJBgNVBAYTAlVTMRUwEwYDVQQKEwxmNXRlc3QubG9j
   YWwxHjAcBgNVBAsTFUNlcnRpZmljYXRlIEF1dGhvcml0eTEVMBMGA1UEAxMMZjV0
   ZXN0LmxvY2FsggkAmg6fZhIgTvowGwYDVR0RBBQwEoIQd3d3LmY1dGVzdC5sb2Nh
   bDAjBgNVHSAEHDAaMAsGCWCGSAFlAgELBTALBglghkgBZQIBCxIwIwYDVR0lBBww
   GgYIKwYBBQUHAwEGCCsGAQUFCAICBgRVHSUAMA4GA1UdDwEB/wQEAwIFoDANBgkq
   hkiG9w0BAQsFAAOCAQEAieguCjEV7tQ+ocoMfWsebTMJUiK+oOOZ95H5FPW5Yz7N
   abRTTGEimncrAyJYqFQgjBhaPV+5o/zn53OQpTe7sJsIzJwWWwktFGJu4zYtpet/
   llGU4/PDdHKmy9ipYEtBlutQP9OLf/PGWuKLnEQ2cT2J2mpDsnELwyJm2YiVoVZp
   wRf4/gcMZK07YrRigZl6Rr33yw8hBRprLyhL9O+tH72sEofX6+m0Z/qEqre6uveR
   3LO+WaxRxCk08ZSobiVJh/lbKEnMCOVL4DsIBDCprcMwxEzdHFtrMwCZg/iQEvZR
   Aasj6BRzEM+e92jAVluUbNja26kd6ImGZaLqul/Elw==
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
   MIIElDCCA3ygAwIBAgIBBDANBgkqhkiG9w0BAQsFADBbMQswCQYDVQQGEwJVUzEV
   MBMGA1UEChMMZjV0ZXN0LmxvY2FsMR4wHAYDVQQLExVDZXJ0aWZpY2F0ZSBBdXRo
   b3JpdHkxFTATBgNVBAMTDGY1dGVzdC5sb2NhbDAeFw0xNzA3MTIwODA2MjdaFw0y
   MDA1MDEwODA2MjdaMH0xCzAJBgNVBAYTAlVTMRUwEwYDVQQKEwxmNXRlc3QubG9j
   YWwxGTAXBgNVBAsTEFVzZXIgQ2VydGlmaWNhdGUxGjAYBgNVBAMTEXVzZXIuZjV0
   ZXN0LmxvY2FsMSAwHgYJKoZIhvcNAQkBFhF1c2VyQGY1dGVzdC5sb2NhbDCCASIw
   DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJmy1XU/hJCbvIT5Dsb4s59yep4j
   zR0OScuFi0keAaZhqdKxW69LN61/M4a7ohRQHj1YEHTRMLQuzSo1keoVqm52KKEy
   Ws9lkpq3S00nB+jCN1ZcvYbW7FDVBPne4Z+Rkd5VsSwhX2wE7B+is5L0XhKUPb4B
   WXdOnHmS/TUH5M5nxiFQnygxr69qMK+pfLqHCk8H8g84zpujE9QSks5iV1xeRdEq
   bOME/VYrllzvYrBRhCzcftJp+PtbY57i/CSawg0P/GeRvPmJoe9HO/vcoG9HmtDX
   s8mtdg6mUKCYBVhED2362bj1KiDZ6t7IoCafBXM94oPlDAG8tAucGbH5gJcCAwEA
   AaOCAT8wggE7MDsGCWCGSAGG+EIBDQQuFixPcGVuU1NMIGdlbmVyYXRlZCBzbWFy
   dGNhcmQgdXNlciBjZXJ0aWZpY2F0ZTAdBgNVHQ4EFgQUwaMwNzNNL4dhB/AzQBaj
   AkindiUwHwYDVR0jBBgwFoAUp1RR2qzOWUoZT921koMLiOwHOFIwDgYDVR0PAQH/
   BAQDAgbAMCkGA1UdJQQiMCAGCCsGAQUFBwMCBgorBgEEAYI3FAICBggrBgEFBQcD
   BDA/BgNVHREEODA2gRF1c2VyQGY1dGVzdC5sb2NhbKAhBgorBgEEAYI3FAIDoBMM
   EXVzZXJAZjV0ZXN0LmxvY2FsMCMGA1UdIAQcMBowCwYJYIZIAWUCAQsJMAsGCWCG
   SAFlAgELEzAbBgNVHQkEFDASMBAGCCsGAQUFBwkEMQQTAlVTMA0GCSqGSIb3DQEB
   CwUAA4IBAQAFKi84V5UX1BiY/XG4gkCwP63JmWwBl9DgFjdG9eXPlFfZIGw/mlEj
   uULGdHLVqOJ1nseuNdbbHic3anxN7TFlZTm+92xX6/mQhumabvXGqq5s9FjvzmQl
   6LSEH8U1oGBr1ByV44U3ifJXuSJyrUtfcZN0BifskcAa05C2pJTkDMxHnG1n/s2C
   lu+Cf2AqAoOgZCz2PsgJtbV5VXckzX+AsWAp2R4ltNWqIbaKEFGsOb9lJa53qmQc
   25iGpuAGm/ierJoVDfDfLnEWK6vWKiQ7MnbwVG6Rot08uYnyBvgK2JzoGMVhjys0
   peMa0CNvHv2B/PtbPaNtCKqHJhz6zOI3
   -----END CERTIFICATE-----

Client private key

.. code-block:: console

   -----BEGIN RSA PRIVATE KEY-----
   MIIEogIBAAKCAQEAmbLVdT+EkJu8hPkOxvizn3J6niPNHQ5Jy4WLSR4BpmGp0rFb
   r0s3rX8zhruiFFAePVgQdNEwtC7NKjWR6hWqbnYooTJaz2WSmrdLTScH6MI3Vly9
   htbsUNUE+d7hn5GR3lWxLCFfbATsH6KzkvReEpQ9vgFZd06ceZL9NQfkzmfGIVCf
   KDGvr2owr6l8uocKTwfyDzjOm6MT1BKSzmJXXF5F0Sps4wT9ViuWXO9isFGELNx+
   0mn4+1tjnuL8JJrCDQ/8Z5G8+Ymh70c7+9ygb0ea0Nezya12DqZQoJgFWEQPbfrZ
   uPUqINnq3sigJp8Fcz3ig+UMAby0C5wZsfmAlwIDAQABAoIBAGlmF7d1vWSlR5ww
   Zw/PUO5QxQFZL7lzKOvmQmP7rcn5Q0n20hbdj+rsRdtpJHalknciwvY41htZ1NvT
   LKLIBL4HTUltjJSY5PYwJ/VahLP7K5OPuXCURi4QRn9LdpHEc7FyNjM7F4KtxXbU
   TizCYxh+i/CWYFHOmMNOJ1GMfj2EIFsUh7i3D9W3A/HKaEn7RWfFWBpF8OwfF7Bl
   k/qyhjIjv8ux3f7K9izvUiVWH/T9FMPXhb89ieT6Up5Qgrq1ejq6JnHkUhZvrA3N
   AFWUI2SxMGMy+jS7HCwj5fM3it/FkkG2uf2v3CXx5CP//lmBWid3nCCr9FtB0UgK
   BwrQ7nECgYEAyxViZTBuPdH0q/GVHcknlIXvl0B4Ah5pNdgfl345fkOLjtXe5HoR
   MMuLHGACD0/mVn4rl/obU/359ANOOrDGT/66AAD24VhNRtvoeMzDRXJ+Y9QNdBwo
   tNHntZzp4msolFkSiHUObHG5jXcxryDig2Y54ZLeRJClCFqBXr1HfTsCgYEAwb8+
   LJYC/SIsbSq6O7cUhiOgcyTkKmKueFUH7ic8JzYXNOTu/mAJuVWb9X1rzCRLc6wj
   MXj9lKZoyVHaoY7aAtd0y75MuoH0FEZG7btE6iba48ZTiAKc3hZXFOszYdPwWUjI
   fRQK3g0aRPfrgXhkTFG/aXc6rWFbxZCd9x1YBFUCgYATMmNJs2lIWLdrJXv2A9TE
   +mAqiQKPGLbTSym5VUo0AEiJ6PeX214Sobr1pLGtJt1cIbMXO6Inr2NYSJO1go5M
   c4S7iVvM817iqtjvylNPFkKSRzI6XosOhKUFit6k84Ize7P/yCjj4WAr2i+NIWuo
   BhrEkvCFxLKE9qEyBmxijwKBgFzlVGtOVgqHGyQQq5C8PKQAawsqchf8jsj1hELl
   Hwtx/PiImCrxY1gwuwGe7FPKRz8kFw++gl+G1pFIpPp3owJfyglyqhl2+8/IznNo
   KifXD3bM/folvo8hyQknqNBMLV6x7idCt982CxVshcfjMLwDKjLoTwMYvkbhC0yU
   DkKtAoGABYODvNIuhUQGk8sKcjByZIpMBeeaFBqPSn0dClUvZnTDTA5sKpblnzQ7
   xj1IK+ZEQQewJ4TifT4CtskkUYDoGz21vsqlBJGXzq/mQPjbyYmeE43jxik7hZ1E
   M33AhM3mAkOT6tnFoD78DNZn8HlHKuaqtlljYCCCiH7tkA59Cuw=
   -----END RSA PRIVATE KEY-----

Analysis
~~~~~~~~

-  The above iRule inspects the x509 subject value in the client’s
   certificate and makes an access decision based on that value. In this
   very simple example, a specific set of users may access different
   corporate resources hosted behind the same VIP.

Testing
~~~~~~~
   
#. In the Client Authentication section of the client SSL
   profile ``f5test``, set Client Certificate to ``Require``, and
   assign ``ca_f5test`` to the Trusted Certificate Authorities option.

#. Test accessing the HTTPS URL https://www.f5test.local from the
   client. The client browser should prompt you to select a certificate.
   Upon selecting this certificate, you should be able to pass through
   to the application.
