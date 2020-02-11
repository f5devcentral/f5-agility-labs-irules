#####################################################
Lab 5 - HTTP Payload Manipulation
#####################################################

Collect an HTTP payload, change it, and release it to the client.
As in the previous lab replace Damn with Darn, or get creative.  
We arent going to use a stream profile this time we are using an
HTTP::payload command instead.

.. IMPORTANT::
  •	Estimated completion time: 20 minutes

#. Open Chrome Browser
#. Enter https://bigip1 into the address bar and hit Enter

   .. image:: /_static/class1/bigip_login.png
      :width: 800

#. #. Login with **username**: **admin** 
              **password**: **admin.F5demo.com**
#. Click Local Traffic -> iRules  -> iRules List
#. Click **Create** button

   .. image:: /_static/class1/irule_create.png
      :width: 800

#. Click **Create** button
#. Enter Name of **HTTP_Payload_iRule**
#. Enter Your Code
#. Click **Finished**
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on **http_irules_vip**

   .. image:: /_static/class1/select_vs.png
      :width: 800

#. Click on the **Resources** tab.
#. Click **Manage** button for the iRules section.

   .. image:: /_static/class1/resources.png
      :width: 800

#. What should you do here? (Hint: Remove Stream_iRule and replace with HTTP_Payload_iRule)
#. Click the Finished button
#. Open the Firefox browser
#. Enter https://dvwa.f5lab.com  and ensure you get there and it is HTTPS

.. HINT::

  Basic Hint

  `if you need a hint here is some example code: <../../class1/module1/irules/lab5irule_0.html>`_

  Link to DevCentral: https://clouddocs.f5.com/api/irules/HTTP__collect.html

  Link to DevCentral: https://clouddocs.f5.com/api/irules/HTTP__release.html


  If you are really stuck, here is what we are looking for:

  #. `When HTTP_Request comes in <../../class1/module1/irules/lab5irule_1.html>`__
  #. `Second change the version of HTTP and disable compression for the request <../../class1/module1/irules/lab5irule_2.html>`__
  #. `When HTTP_RESPONSE comes back <../../class1/module1/irules/lab5irule_3.html>`__
  #. `Next we need to collect some HTTP::collect some data. <../../class1/module1/irules/lab5irule_4.html>`__
  #. `Now when we get HTTP_RESPONSE_DATA <../../class1/module1/irules/lab5irule_5.html>`__
  #. `Now we will set some find and replace strings. <../../class1/module1/irules/lab5irule_6.html>`__
  #. `Finally we will perform a regsub on the payload and replace with new text. <../../class1/module1/irules/lab5irule_7.html>`__
  #. `Now you should have enough to understand and the majority of code to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab5irule_99.html>`__
