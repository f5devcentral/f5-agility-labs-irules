############################################################
Lab 1 - Create an iRule that Parses the URI to Route Traffic
############################################################


**Creating your first HTTP iRule that routes traffic based upon the value of the Host name**


The goal of this lab is to route incoming HTTP requests to a specific pool based on the incoming http host name.

Please create an iRule that will route traffic based on the following table:

.. list-table::
    :widths: 40 40
    :header-rows: 1

    * - **Host Name**
      - **Pool Name**
    * - **dvwa.f5lab.com**
      - **dvwa_pool_http**
    * - **peruggia.f5lab.com**
      - **peruggia_http_pool**
    * - **wackopicko.f5lab.com**
      - **wackopicko_http_pool**

.. IMPORTANT::
   - Estimated completion time: 10 minutes

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

#. Enter Name of **URI_Routing_iRule**
#. Enter your code
#. Click **Finished**
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on **http_irules_vip**

   .. image:: /_static/class1/select_vs.png
      :width: 800

#. Click on the **Resources tab**
#. Click **Manage** button for the iRules section

   .. image:: /_static/class1/resources.png
      :width: 800

#. Click on **URI_Routing_iRule** from the Available box and click the << button, thus moving it to the Enabled box.

   .. image:: /_static/class1/lab1-irules-add.png
      :width: 800

#. Click the **Finished** button
#. Open a new tab in Chrome
#. Enter http://dvwa.f5lab.com/ and ensure you get there
#. Now enter http://peruggia.f5lab.com/ and ensure you get to the app
#. Finally, enter http://wackopicko.f5lab.com/  and ensure you can get to that app

   .. image:: /_static/class1/test_sites.png
      :width: 800

#. If you see this image below - it means your iRule did not work.

   .. image:: /_static/class1/it_works.png
      :width: 800


.. HINT::
   `If you need a basic hint here is some example code: <../../class1/module1/irules/lab1irule_0.html>`__

   Here is a link to DevCentral: https://clouddocs.f5.com/api/irules/HTTP__host.html

   If you are really stuck, here is what we are looking for:

   #. `When HTTP_Request comes in <../../class1/module1/irules/lab1irule_1.html>`__
   #. `Evaluate the HTTP_host name  <../../class1/module1/irules/lab1irule_2.html>`__
   #. `If it matches send it to the correct pool. <../../class1/module1/irules/lab1irule_3.html>`__
   #. `Loop through all the host names you want to match on and continue to direct to the correct pools. <../../class1/module1/irules/lab1irule_4.html>`__
   #. `Now you should have enough to understand and the majority of code needed to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab1irule_99.html>`__
