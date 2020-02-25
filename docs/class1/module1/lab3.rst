
#####################################################
Lab 3 - HTTP to HTTPS Redirect
#####################################################


#. Create an iRule to redirect all traffic that arrives at an HTTP virtual server to be redirected to the same IP address but using an HTTPS port. 
#. The full original HTTP request should be maintained when re-directing.  Example http://my.domain.com/app1/index1.html should redirect to https://my.domain.com/app1/inex.html
#. Traffic goes to the HTTPS virtual server should still perform the pool selection and should still perform the header stripping from previous labs.

.. IMPORTANT::
  •	Estimated completion time: 20 minutes


#. Open Chrome Browser
#. Enter https://bigip1 into the address bar and hit Enter

   .. image:: /_static/class1/bigip_login.png
      :width: 800

#. Login with **username**: **admin** 
              **password**: **admin.F5demo.com**
#. Click Local Traffic -> iRules  -> iRules List
#. Click **Create** button

   .. image:: /_static/class1/irule_create.png
      :width: 800

#. Enter Name of **HTTP_to_HTTPS_iRule**
#. Enter Your Code
#. Click **Finished**
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on **http_irules_vip**

   .. image:: /_static/class1/select_vs.png
      :width: 800

#. Click on the **Resources** tab
#. Click **Manage** button for the iRules section

   .. image:: /_static/class1/resources.png
      :width: 800

#. Click on HTTP_to_HTTPS_iRule from the Available box and click the << button, thus moving it to the Enabled box, your first and now second iRule should be in the Enabled box.

   .. image:: /_static/class1/manage_irule.png
      :width: 800

#. Click the **Finished** button
#. Open the Firefox browser
#. Click the 3 horizontal line button on the far right of the address bar
#. Use developer tools in Mozilla, or use Chrome to view headers

   .. image:: /_static/class1/firefox_developer.png
      :width: 600

#. Look at the headers for each of your requests. Did you log them all? What is the value of the Server header? None of this should have changed since the last lab.

.. HINT::

  Basic Hint
  `if you need a hint here is some example code: <../../class1/module1/irules/lab3irule_0.html>`__

  Link to DevCentral: https://clouddocs.f5.com/api/irules/HTTP__redirect.html

  If you are really stuck, here is what we are looking for:

  #. `When HTTP_Request comes in <../../class1/module1/irules/lab3irule_1.html>`__
  #. `Redirect from HTTP to HTTPS <../../class1/module1/irules/lab3irule_2.html>`__
  #. `Now you should have enough to understand and the majority of code to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab3irule_99.html>`__
